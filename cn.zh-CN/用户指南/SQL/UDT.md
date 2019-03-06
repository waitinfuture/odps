# UDT {#concept_drb_xk4_hfb .concept}

MaxCompute基于新一代的SQL引擎再推出新功能User Defined Type（简称UDT）。UDT允许在SQL中直接引用第三方语言的类或者对象，获取其数据内容或者调用其方法。

## 应用场景 {#section_wbd_xp4_hfb .section}

UDT的常用场景如下：

-   场景1：某些用其他语言实现非常简单的功能，比如只需要调用一次Java内置类的方法即可实现的功能，而MaxCompute的内置函数却没有简单的方法实现。若使用UDF实现，整个过程又过于繁杂。
-   场景2：SQL中需要调用第三方库来实现相关功能。希望能够在SQL中直接调用，而不需要再wrap一层UDF。
-   场景3：[Select Transform](intl.zh-CN/用户指南/SQL/SELECT操作/Select Transform语法.md#)支持把脚本写到SQL语句中，可读性和代码维护大提升。但是某些语言无法这么用，比如Java源代码必须经过编译才能执行，希望类似这些语言也可直接写到SQL中。

## UDT概述 {#section_drd_zp4_hfb .section}

MaxCompute中的UDT功能允许在SQL中直接引用第三方语言的类或者对象，获取其数据内容或者调用其方法。

在其他的SQL引擎中也有UDT的概念，但是和MaxCompute的概念有许多差异。很多SQL引擎中的概念比较像MaxCompute的struct复杂类型。而某些语言提供了调用第三方库的功能，如Oracle的CREATE TYPE。相比之下，MaxCompute的UDT更像CREATE TYPE的概念，Type中不仅仅包含数据域，还包含方法。而且MaxCompute不需要用特殊的DDL语法来定义类型的映射，而是在SQL中直接使用。

示例如下：

```
set odps.sql.type.system.odps2=true;    
-- 打开新类型，因为下面的操作会用到Integer，即int类型
SELECT java.lang.Integer.MAX_VALUE;
```

上述语句的输出结果如下：

```
+-----------+
| max_value |
+-----------+
| 2147483647 |
+-----------+
```

和Java语言一样，java.lang package可以省略，所以上述示例可以简写为：

```
set odps.sql.type.system.odps2=true;
SELECT Integer.MAX_VALUE;
```

可以看到，上述示例在select列表中直接写上了类似于Java表达式的表达式，而这个表达式按照Java的语义来执行，这个表达式表现出的能力就是MaxCompute的UDT。

UDT所提供的所有扩展能力，实际上用UDF都可以实现。如上述示例，若使用UDF实现，需要您进行下述操作。

1.  定义一个UDF的类。

    ```
    package com.aliyun.odps.test;
    public class IntegerMaxValue extends com.aliyun.odps.udf.UDF {
      public Integer evaluate() {
         return Integer.MAX_VALUE;
      } 
    }
    ```

2.  将上面的UDF编译，并打成Jar包。然后上传Jar包，并创建function。

    ```
    add jar odps-test.jar;
    create function integer_max_value as 'com.aliyun.odps.test.IntegerMaxValue' using 'odps-test.jar';
    ```

3.  在SQL中使用。

    ```
    select integer_max_value();
    ```

    UDT简化了上述一系列的过程，让您能够轻松简单地用其他语言扩展SQL的功能。


## 实现原理 {#section_hgw_cn4_hfb .section}

概述中的示例表现的是Java静态域访问的能力，而UDT的能力远不限于此。下述示例将为您介绍UDT的执行过程和具体功能。

```
-- 示例数据
@table1 := select * from values ('100000000000000000000') as t(x);
@table2 := select * from values (100L) as t(y);
-- 代码逻辑
@a := select new java.math.BigInteger(x) x from @table1;          -- new创建对象
@b := select java.math.BigInteger.valueOf(y) y from @table2;      -- 静态方法调用
select /*+mapjoin(b)*/ x.add(y).toString() from @a a join @b b;   -- 实例方法调用
```

输出结果如下：

```
100000000000000000100
```

上述示例还表现了一种用UDF比较不好实现的功能：子查询的结果允许UDT类型的列。如上面变量a的x列是java.math.BigInteger类型，而不是内置类型。UDT类型的数据可以被带到下一个operator中再调用其他方法，甚至能参与数据shuffle。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22183/155186483613239_zh-CN.png)

如上图可知，该UDT共有三个STAGE：M1、R2和J3。如果您熟悉MapReduce原理便会知道，由于join的存在需要做数据reshuffle，所以会出现多个stage。一般情况下，不同stage不仅是在不同进程，甚至是在不同物理机器上运行的。

查看M1，会发现M1仅仅执行了`new java.math.BigInteger(x)` 这个操作。

查看J3，可以看到J3在不同阶段执行了`java.math.BigInteger.valueOf(y)`和`x.add(y).toString()`的操作。这几个操作不仅分阶段执行，而且在不同进程、不同物理机器上执行。但是UDT把这个过程封装起来，看起来和在同一个JVM中执行的效果几乎一样。

详细功能说明

-   目前UDT仅支持了Java语言，后续版本会加入对其他语言的支持。
-   UDT同样允许用户上传自己的Jar包，并且直接引用。当前提供了一些flag以方便使用。
    -   set odps.sql.session.resources指定引用的资源，可以指定多个，用逗号隔开。如`set odps.sql.session.resources=foo.sh,bar.txt;`。

        **说明：** 这个flag和Select Transform中指定资源的flag相同，所以这个flag会同时影响两个功能。 如UDT概述中UDF的Jar包，用UDT来使用：

        ```
        set odps.sql.type.system.odps2=true;
        set odps.sql.session.resources=odps-test.jar; 
        --指定要引用的jar，这些jar一定要事先上传到project，并且需要是jar类型的资源
        select new com.aliyun.odps.test.IntegerMaxValue().evaluate();
        ```

    -   odps.sql.session.java.imports指定默认的Java package，可以指定多个，用逗号隔开。和Java的import语句类似，可以提供完整类路径，如java.math.BigInteger，也可以使用`*`。暂不支持`static import`。

        如UDT概述中UDF的Jar包，用UDT来使用还可有如下写法：

        ```
        set odps.sql.type.system.odps2=true;
        set odps.sql.session.resources=odps-test.jar;
        set odps.sql.session.java.imports=com.aliyun.odps.test.*;  
        -- 指定默认的package
        select new IntegerMaxValue().evaluate();
        ```

-   UDT支持的操作如下所示：

    -   实例化对象的new操作。
    -   实例化数组的new操作，包括使用初始化列表创建数组，如new Integer\[\] \{ 1, 2, 3 \}。
    -   方法调用，包括静态方法调用（因此能用工厂方法构建对象）。
    -   域访问，包括静态域。
    **说明：** 

    -   仅支持公有方法和公有域的访问。
    -   UDT中的标识符是大小写敏感的，包括package、类名、方法名和域名。
    -   UDT支持类型转换，支持SQL的风格，如cast\(1 as java.lang.Object\)。但不支持Java风格的类型转换，如\(Object\)1。
    -   暂不支持匿名类和lambda表达式（后续版本可能会支持）。
    -   暂不支持无返回值的函数调用（因为UDT均出现在expression中，没有返回值的函数调用无法嵌入到expression中，这个问题在后续的版本中会有解决方案）。
-   Java SDK的类都是默认可用的。

    **说明：** 目前runtime使用的JDK版本是JDK1.8，比该版本更新的JDK功能可能不支持。

-   所有的运算符都是MaxCompute SQL的语义，不是UDT的语义。如`String.valueOf(1) + String.valueOf(2)`的结果是3（string隐式转换为double，并且double相加） ，而不是12（java中string相加是concatenate的语义）。

    除string的相加操作比较容易混淆外，另一个比较容易混淆的是=操作。SQL中的=不是赋值而是判断相等。而对于Java对象来说，判断相等应该用equals方法，通过等号判断的相等无法保证其行为（在UDT场景下，同一对象的概念是不能保证的，详情请参见下文的说明）。

-   内置类型与特定Java类型有一一映射关系，详情请参见[Java UDF](intl.zh-CN/用户指南/SQL/UDF/Java UDF.md#)中的数据类型映射表，这个映射关系在UDT也有效。

    -   内置类型的数据能够直接调用其映射到的Java类型的方法，如`'123'.length() , 1L.hashCode()`。
    -   UDT类型能够直接参与内置函数或者UDF的运算， 如`chr(Long.valueOf('100'))`，其中`Long.valueOf`返回的是`java.lang.Long`类型的数据，而内置函数chr接受的数据类型是内置类型BIGINT。
    -   Java的primitive类型可以自动转化为其boxing类型，并应用前两条规则。
    **说明：** 部分内置的[新数据类型](../../../../../intl.zh-CN/用户指南/基本概念/数据类型.md#)需要设置`set odps.sql.type.system.odps2=true;` 方可使用，否则会报错。

-   UDT对泛型有比较完整的支持，如`java.util.Arrays.asList(new java.math.BigInteger('1'))`，编译器能够根据参数类型知道该方法的返回值是`java.util.List<java.math.BigInteger>`类型。

    **说明：** 构造函数需要指定类型参数，否则使用java.lang.Object，这一点和Java保持一致：`new java.util.ArrayList(java.util.Arrays.asList('1', '2'))`的结果是`java.util.ArrayList<Object>`类型，而`new java.util.ArrayList<String>(java.util.Arrays.asList('1', '2'))`的结果是`java.util.ArrayList<String>`类型。

-   UDT对同一对象的概念是模糊的。这是由数据的reshuffle导致的。从第一部分join的示例可以看出，对象有可能会在不同进程，不同物理机器之间传输，在传输过程中同一个对象的两个引用后面可能分别引用了不同的对象（比如对象先被shuffle到两台机器，然后下次又shuffle回一起）。

    所以在使用UDT时，应该避免使用=operator来判断相等，而是使用equals方法。

    某行某列的对象，其内部包含的各个数据对象的相关性是可以保证的。不能保证的是不同行或者不同列的对象的数据相关性。

-   目前UDT不能用作shuffle key：包括join、group by、distribute by、sort by、order by、cluster by等结构的key。

    并不是说UDT不能用在这些结构中，UDT可以在expression中间的任意阶段使用，只是不能作为最终输出。比如您虽然不能`group by new java.math.BigInteger('123')`，但是可以`group by new java.math.BigInteger('123').hashCode()`。因为hashCode的返回值是int.class类型可以当做内置类型int来使用（应用上述内置类型与特定java类型规则）。

-   UDT扩展了类型转换规则：
    -   UDT对象能够被隐式类型转换为其基类对象。
    -   UDT对象能够被强制类型转换为其基类或子类对象。
    -   没有继承关系的两个对象之间遵守原来的类型转换规则，注意这时候可能会导致内容变化，比如java.lang.Long类型的数据是可以强制转换为java.lang.Integer的，应用的是内置类型的bigint强制转换为int的过程，而这个过程会真的导致数据内容的变化，甚至可能会有精度损失。
-   目前UDT对象不能落盘，这意味着不能将UDT对象insert到表中（实际上DDL不支持UDT，创建不出这样的表），当然，隐式类型转换变成了内置类型的除外。同时，屏显的最终结果也不能是UDT类型，对于屏显的场景，由于所有的java类都有toString\(\)方法，而java.lang.String类型是合法的。所以debug的时候，可以用这种方法来观察UDT的内容。

    您也可以设置`set odps.sql.udt.display.tostring=true;`这样MaxCompute会自动帮用户把所有的以UDT为最终输出的列wrap上`java.util.Objects.toString(...)`，从而方便调试。这个flag只对屏显语句生效，对insert语句不生效，所以专门用在调试中。

    内置类型支持binary，因此支持自己实现serialize的过程，将byte\[\]的数据落盘。下次读出来的时候再还原回来。

    某些类可能自带序列化和反序列化的方法，如protobuffer。目前UDT依旧不支持落盘，还是需要用户自己调用序列化反序列化方法，变成binary数据类型来落盘。

-   UDT不仅能够实现scalar函数的功能，配合着内置函数[collect list](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)和[explode](intl.zh-CN/用户指南/SQL/内建函数/其他函数.md#)，完全能够实现 aggregator和table function的功能。

## UDT示例 {#section_ors_lq4_hfb .section}

-   使用Java数组示例

    ```
    set odps.sql.type.system.odps2=true;
    set odps.sql.udt.display.tostring=true;
    select
        new Integer[10],    -- 创建一个10个元素的数组
        new Integer[] {c1, c2, c3},  -- 通过初始化列表创建一个长度为3的数组
        new Integer[][] { new Integer[] {c1, c2}, new Integer[] {c3, c4} },  -- 创建多维数组
        new Integer[] {c1, c2, c3} [2], -- 通过下标操作访问数组元素
        java.util.Arrays.asList(c1, c2, c3);    -- 这个创建了一个 List<Integer>，这个也能当做array<int>来用，所以这是另一个创建内置array数据的方法
    from values (1,2,3,4) as t(c1, c2, c3, c4);
    ```

-   使用JSON示例

    UDT的runtime自带一个JSON的依赖（2.2.4），因此可以直接使用JSON。

    ```
    set odps.sql.type.system.odps2=true;
    set odps.sql.session.java.imports=java.util.*,java,com.google.gson.*; -- 同时import多个package时用逗号隔开
    @a := select new Gson() gson;   -- 构建gson对象
    select 
    gson.toJson(new ArrayList<Integer>(Arrays.asList(1, 2, 3))), -- 将任意对象转成 json 字符串。
    cast(gson.fromJson('["a","b","c"]', List.class) as List<String>) -- 反序列化json字符串, 注意gson的接口，直接反序列化出来是List<Object>类型，所以这里强转成了 List<String>，方便后续使用。
    from @a;
    ```

    相比于内置函数[get\_json\_object](intl.zh-CN/用户指南/SQL/内建函数/字符串函数.md#)，上述用法不仅使用方便，在需要对Json字符串多个部分做内容提取时，先将JSON字符串反序列成格式化数据，大大提升工作效率。

    除JSON外，MaxCompute runtime自带的依赖还包括：commons-logging（1.1.1）， commons-lang（2.5）， commons-io（2.4），protobuf-java（2.4.1）。

-   复杂类型操作示例

    内置类型array和map与java.util.List和java.util.Map存在映射关系。结果如下：

    -   Java中实现了java.util.List或者java.util.Map接口的类的对象，都可参与MaxComputeSQL的复杂类型操作。
    -   MaxComput中 array，map的数据，能够直接调用List或者Map的接口。
    ```
    set odps.sql.type.system.odps2=true;
    set odps.sql.session.java.imports=java.util.*;
    select
        size(new ArrayList<Integer>()),        -- 对 ArrayList数据调用内置函数size
        array(1,2,3).size(),                   -- 对内置类型array调用 List的方法
        sort_array(new ArrayList<Integer>()),  -- 对 ArrayList 的数据进行排序
        al[1],                                 -- 虽然java的List不支持下标操作，但是别忘了array是支持的
        Objects.toString(a),        -- 过去不支持将array类型cast成string，现在有绕过方法了
        array(1,2,3).subList(1, 2)             -- 求subList
    from (select new ArrayList<Integer>(array(1,2,3)) as al, array(1,2,3) as a) t;
    ```

-   聚合操作的实现示例

    UDT实现聚合的原理是，先用内置函数[collect\_set](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)或[collect\_list](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)函数将数据转变成List，之后对该List应用UDT的标量方法求得这一组数据的聚合值。

    例如下述示例实现对BigInteger求中位数（由于数据是java.math.BigInteger类型的，所以不能直接用内置的[median](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)函数）。

    ```
    set odps.sql.session.java.imports=java.math.*;
    @test_data := select * from values (1),(2),(3),(5) as t(value);
    @a := select collect_list(new BigInteger(value)) values from @test_data;  -- 先把数据聚合成list
    @b := select sort_array(values) as values, values.size() cnt from @a;  -- 求中位数的逻辑，先将数据排序
    @c := select if(cnt % 2 == 1, new BigDecimal(values[cnt div 2]), new BigDecimal(values[cnt div 2 - 1].add(values[cnt div 2])).divide(new BigDecimal(2))) med from @b;
    -- 最终结果
    select med.toString() from @c;
    ```

    由于collect\_list会先把所有数据都收集到一块，是没有办法实现partial aggregate的，所以这个做法的效率会比内置的aggregator或者UDAF低，所以在内置aggregator能实现的情况下，应尽量使用内置的aggregator。同时把一个group的所有数据都收集到一起的做法，会增加数据倾斜的风险。

    但是另一方面，如果UDAF本身的逻辑就是要将所有数据收集到一块（比如类似内置函数wm\_concat的功能），此时使用上述方法，反而可能比UDAF（注意不是内置aggregator）高。

-   表值函数的实现示例

    表值函数允许输入多行多列数据，输出多行多列数据。可以按照下述原理实现：

    1.  对于输入多行多列数据，可以参考聚合函数实现的示例。
    2.  要实现多行的输出，可以让UDT方法输出一个Collection类型的数据（List 或者 Map\)，然后调用explode函数，将Collections展开成多行。
    3.  UDT本身就可以包含多个数据域，通过调用不同的getter方法来获取各个域的内容即可展开成多列。
    下述示例实现将一个json字符串的内容展开出来的功能。

    ```
    @a := select '[{"a":"1","b":"2"},{"a":"1","b":"2"}]' str; -- 示例数据
    @b := select new com.google.gson.Gson().fromJson(str, java.util.List.class) l from @a; -- 反序列化json
    @c := select cast(e as java.util.Map<Object,Object>) m from @b lateral view explode(l) t as e;  -- 用explode打成多行
    @d := select m.get('a') as a, m.get('b') as b from @c; -- 展开成多列
    select a.toString() a, b.toString() b from @d; -- 最终结果输出(注意变量d的输出中a, b两列是Object类型）
    ```


## 功能/性能/安全性 {#section_mxl_mq4_hfb .section}

UDT在功能方面的优势如下：

-   使用极其简单，不需要定义任何function。
-   JDK的所有功能都可以拿来用，大大扩展了SQL的能力。
-   代码直接和SQL放在一块，便于管理。
-   其它类库拿来即用，代码重用率高。
-   可以使用面向对象的思想设计某些功能。

后续待完善功能如下：

-   支持无返回值的函数调用（或者有返回值，但是忽略返回值，直接取操作数本身，如调用List的add方法，结束后返回执行完add操作的List）。
-   支持匿名类和lambda表达式。
-   支持用作shuffle key。
-   支持JAVA外的其他语言，如python。

性能方面，UDT执行过程和UDF非常接近，其性能与UDF几乎是一致的，而且计算引擎做了很多优化，在某些场景下UDT的性能更高。

-   对象在一个进程内实际上是不需要做列化反序列化的，只有跨进程的时候才需要。简单地说，就是在没有join或者aggregator等需要做数据reshuffle的情况下，UDT并没有序列化反序列化的开销。
-   UDT的Runtime实现是基于codegen，而不是反射，所以不会存在反射带来的性能损失连续的多个UDT的操作，实际上会合并在一起，在一个FunctionCall里一起执行，如前面例子中`values[x].add(values[y]).divide(java.math.BigInteger.valueOf(2))` 这个看似存在多次UDT方法调用的操作，实际上只有一次调用。所以虽然UDT操作的单元都比较小，但是并不会因此造成多次函数调用的接口上的额外开销。

在安全控制方面，UDT和UDF完全一样，都会受到[Java沙箱](intl.zh-CN/用户指南/Java沙箱.md#)policy的限制。所以如果要使用受限的操作，需要打开沙箱隔离，或者申请沙箱白名单。

