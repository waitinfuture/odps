# Java UDF {#concept_mxb_xn2_vdb .concept}

MaxCompute的UDF包括UDF，UDAF 和UDTF三种函数，本文将重点介绍如何通过Java实现这三种函数。

## 参数与返回值类型 {#section_uhs_43f_vdb .section}

MaxCompute 2.0版本升级后，Java UDF支持的数据类型从原来的Bigint，String，Double，Boolean扩展了更多基本的数据类型，同时还扩展支持了ARRAY，MAP，STRUCT等复杂类型。

-   Java UDF使用新基本类型的方法，如下所示：
    -   UDTF通过@Resolve注解来获取signature，如：`@Resolve("smallint->varchar(10)")`。
    -   UDF通过反射分析evaluate来获取signature，此时MaxCompute内置类型与 Java类型符合一一映射关系。
    -   UDAF通过@Resolve注解来获取signature，MaxCompute2.0支持在注解中使用新类型，如：`@Resolve("smallint->varchar(10)")`。
-   Java UDF使用复杂类型的方法，如下所示：
    -   UDTF通过@Resolve annotation来指定signature，如：`@Resolve("array<string>,struct<a1:bigint,b1:string>,string->map<string,bigint>,struct<b1:bigint>")`。
    -   UDF通过evaluate方法的signature来映射UDF的输入输出类型，此时参考 MaxCompute类型与Java类型的映射关系。其中array对应java.util.List，map对应java.util.Map，struct对应com.aliyun.odps.data.Struct。
    -   UDAF通过@Resolve注解来获取signature，MaxCompute2.0支持在注解中使用新类型，如： `@Resolve("smallint->varchar(10)")`。

        **说明：** 

        -   您可以使用`type,*`实现任意个数的传参，例如@resolve\("string,\*-\>array<string\>"\)，请注意此处array后需要加subtype。
        -   `com.aliyun.odps.data.Struct`从反射看不出field name和field type，所以需要用@Resolve annotation来辅助。即如果需要在UDF中使用 struct，要求在UDF class上也标注上@Resolve注解，这个注解只会影响参数或返回值中包含`com.aliyun.odps.data.Struct`的重载。
        -   目前class上只能提供一个@Resolve annotation，因此一个UDF中带有struct参数或返回值的重载只能有一个。

MaxCompute数据类型与Java类型的对应关系，如下所示：

|MaxCompute Type|Java Type|
|:--------------|:--------|
|Tinyint|java.lang.Byte|
|Smallint|java.lang.Short|
|Int|java.lang.Integer|
|Bigint|java.lang.Long|
|Float|java.lang.Float|
|Double|java.lang.Double|
|Decimal|java.math.BigDecimal|
|Boolean|java.lang.Boolean|
|String|java.lang.String|
|Varchar|com.aliyun.odps.data.Varchar|
|Binary|com.aliyun.odps.data.Binary|
|Datetime|java.util.Date|
|Timestamp|java.sql.Timestamp|
|Array|java.util.List|
|Map|java.util.Map|
|Struct|com.aliyun.odps.data.Struct|

**说明：** 

-   在UDF中使用输入或输出参数的类型请务必使用Java Type，否则会报错ODPS-0130071。
-   Java中对应的数据类型以及返回值数据类型是对象，首字母请务必大写。
-   SQL中的NULL值通过Java中的NULL引用表示，因此Java primitive type是不允许使用的，因为无法表示SQL中的NULL值。
-   此处Array类型对应的Java类型是List，而不是数组。

## UDF {#section_g4r_wjf_vdb .section}

实现UDF需要继承com.aliyun.odps.udf.UDF类，并实现evaluate方法。evaluate方法必须是非static的public方法 。Evaluate方法的参数和返回值类型将作为SQL中UDF的函数签名。这意味着您可以在UDF中实现多个evaluate方法，在调用UDF时，框架会依据UDF调用的参数类型匹配正确的evaluate方法 。

特别注意：不同的jar包最好不要有类名相同但实现功能逻辑不一样的类。如，UDF\(UDAF/UDTF\)： udf1、 udf2分别对应资源udf1.jar、udf2.jar，如果两个jar包里都包含一个com.aliyun.UserFunction.class类，当同一个sql中同时使用到这两个udf时，系统会随机加载其中一个类，那么就会导致UDF执行行为不一致甚至编译失败。

UDF 的示例如下：

```
package org.alidata.odps.udf.examples; 
  import com.aliyun.odps.udf.UDF; 

public final class Lower extends UDF { 
  public String evaluate(String s) { 
    if (s == null) { 
        return null; 
    } 
        return s.toLowerCase(); 
  } 
}
```

可以通过实现`void setup(ExecutionContext ctx)`和`void close()`来分别实现UDF的初始化和结束代码。

UDF的使用方式与MaxCompute SQL中普通的内建函数相同，详情请参见 [内建函数](intl.zh-CN/用户指南/SQL/内建函数/数学函数.md)。

如果您想了解使用Intellij IDEA开发工具完成完整的Java UDF开发示例，请参见[Intellij IDEA Java UDF开发最佳实践](../../../../../intl.zh-CN/最佳实践/数据开发/Intellij IDEA Java UDF开发最佳实践.md#)。使用Eclipse开发工具完成完整的Java UDF开发示例，请参见[Eclipse Java UDF开发最佳实践](../../../../../intl.zh-CN/最佳实践/数据开发/Eclipse Java UDF开发最佳实践.md#)。

## 其他UDF示例 {#section_kb5_v44_k2b .section}

如以下代码，定义了一个有三个overloads的UDF，其中第一个用了array作为参数，第二个用了map作为参数，第三个用了struct。由于第三个overloads了struct作为参数或者返回值，因此要求必须要对UDF class打上`@Resolve` annotation，来指定 struct的具体类型。

```
@Resolve("struct,string->string") 
public class UdfArray extends UDF { 
  public String evaluate(List vals, Long len) { 
    return vals.get(len.intValue()); 
  } 
  public String evaluate(Map map, String key) { 
    return map.get(key); 
  } 
  public String evaluate(Struct struct, String key) { 
  return struct.getFieldValue("a") + key;
 } 
}
```

用户可以直接将复杂类型传入UDF中：

```
create function my_index as 'UdfArray' using 'myjar.jar'; 
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from co
```

## UDAF {#section_vdy_4kf_vdb .section}

实现Java UDAF类需要继承com.aliyun.odps.udf.Aggregator，并实现如下几个接口：

```
public abstract class Aggregator implements ContextFunction {
  @Override
  public void setup(ExecutionContext ctx) throws UDFException {                                                                   
  }
  @Override
  public void close() throws UDFException {    
  }
  /**
   * 创建聚合Buffer
   * @return Writable 聚合buffer
   */
  abstract public Writable newBuffer();
  /**
   * @param buffer 聚合buffer
   * @param args SQL中调用UDAF时指定的参数，不能为null，但是args里面的元素可以为null，代表对应的输入数据是null
   * @throws UDFException
   */
  abstract public void iterate(Writable buffer, Writable[] args) throws UDFException;
  /**
   * 生成最终结果
   * @param buffer
   * @return Object UDAF的最终结果
   * @throws UDFException
   */
  abstract public Writable terminate(Writable buffer) throws UDFException;
  abstract public void merge(Writable buffer, Writable partial) throws UDFException;
}
```

其中最重要的是iterate，merge和terminate三个接口，UDAF的主要逻辑依赖于这三个接口的实现。此外，还需要您实现自定义的Writable buffer。

以实现求平均值avg为例，下图简要说明了在MaxCompute UDAF中这一函数的实现逻辑及计算流程：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12003/15475238151855_zh-CN.jpg)

在上图中，输入数据被按照一定的大小进行分片（有关分片的描述请参见 [MapReduce](intl.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)），每片的大小适合一个worker在适当的时间内完成。这个分片大小的设置需要您手动配置完成。

UDAF的计算过程分为两个阶段：

-   第一阶段：每个worker统计分片内数据的个数及汇总值，您可以将每个分片内的数据个数及汇总值视为一个中间结果。
-   第二阶段：worker汇总上一个阶段中每个分片内的信息。在最终输出时，r.sum / r.count即是所有输入数据的平均值。

计算平均值的UDAF的代码示例，如下所示：

```
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import com.aliyun.odps.io.DoubleWritable;
import com.aliyun.odps.io.Writable;
import com.aliyun.odps.udf.Aggregator;
import com.aliyun.odps.udf.UDFException;
import com.aliyun.odps.udf.annotation.Resolve;
@Resolve("double->double")
public class AggrAvg extends Aggregator {
  private static class AvgBuffer implements Writable {
    private double sum = 0;
    private long count = 0;
    @Override
    public void write(DataOutput out) throws IOException {
      out.writeDouble(sum);
      out.writeLong(count);
    }
    @Override
    public void readFields(DataInput in) throws IOException {
      sum = in.readDouble();
      count = in.readLong();
    }
  }
  private DoubleWritable ret = new DoubleWritable();
  @Override
  public Writable newBuffer() {
    return new AvgBuffer();
  }
  @Override
  public void iterate(Writable buffer, Writable[] args) throws UDFException {
    DoubleWritable arg = (DoubleWritable) args[0];
    AvgBuffer buf = (AvgBuffer) buffer;
    if (arg != null) {
      buf.count += 1;
      buf.sum += arg.get();
    }
  }
  @Override
  public Writable terminate(Writable buffer) throws UDFException {
    AvgBuffer buf = (AvgBuffer) buffer;
    if (buf.count == 0) {
      ret.set(0);
    } else {
      ret.set(buf.sum / buf.count);
    }
    return ret;
  }
  @Override
  public void merge(Writable buffer, Writable partial) throws UDFException {
    AvgBuffer buf = (AvgBuffer) buffer;
    AvgBuffer p = (AvgBuffer) partial;
    buf.sum += p.sum;
    buf.count += p.count;
  }
}
```

**说明：** 

-   Writable\[\] writables：表示一行数据。代码中是指你传入的列，比如writables\[0\]表示第一列，writables\[1\]表示第二列。
-   iterate\(Writable writable,Writable\[\] writables\)方法：writable是指一个阶段性的汇总数据，在不同map任务中，group by后得出的数据（可理解为一个集合），每行执行一次。
-   merge\(\)方法：将不同的map直接结算的结果进行汇总。
-   terminate\(\)方法：返回数据。
-   newBuffer\(\)方法：创建初始返回结果的值。
-   UDAF在SQL中的使用语法与普通的内建聚合函数相同，详情请参见 [聚合函数](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md)。
-   关于如何运行UDTF的方法与 UDF 类似，详情请参见 [运行 UDF](../../../../../intl.zh-CN/快速入门/JAVA UDF开发（可选）.md)。
-   String对应的Writable类型为Text。

## UDTF {#section_a4t_34f_vdb .section}

Java UDTF需要继承com.aliyun.odps.udf.UDTF类。这个类需要实现4个接口，如下表所示：

|接口定义|描述|
|:---|:-|
|public void setup\(ExecutionContext ctx\) throws UDFException|初始化方法，在UDTF处理输入数据前，调用用户自定义的初始化行为。在每个Worker内setup会被先调用一次。|
|public void process\(Object\[\] args\) throws UDFException|这个方法由框架调用，SQL中每一条记录都会对应调用一次process，process的参数为SQL语句中指定的UDTF输入参数。输入参数以Object\[\]的形式传入，输出结果通过forward函数输出。用户需要在process函数内自行调用forward，以决定输出数据。|
|public void close\(\) throws UDFException|UDTF的结束方法，此方法由框架调用，并且只会被调用一次，即在处理完最后一条记录之后。|
|public void forward\(Object …o\) throws UDFException|用户调用forward方法输出数据，每次forward代表输出一条记录。对应SQL语句UDTF的as子句指定的列。|

UDTF 的程序示例，如下所示：

```
package org.alidata.odps.udtf.examples;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.UDTFCollector;
import com.aliyun.odps.udf.annotation.Resolve;
import com.aliyun.odps.udf.UDFException;
// TODO define input and output types, e.g., "string,string->string,bigint".
   @Resolve("string,bigint->string,bigint")
   public class MyUDTF extends UDTF {
     @Override
     public void process(Object[] args) throws UDFException {
       String a = (String) args[0];
       Long b = (Long) args[1];
       for (String t: a.split("\\s+")) {
         forward(t, b);
       }
     }
   }
```

**说明：** 以上只是程序示例，关于如何在MaxCompute中运行 UDTF的方法与UDF类似，详情请参见：[运行UDF](../../../../../intl.zh-CN/快速入门/JAVA UDF开发（可选）.md)。

在SQL中可以这样使用这个UDTF，假设在MaxCompute上创建UDTF时注册函数名为 user\_udtf：

```
select user_udtf(col0, col1) as (c0, c1) from my_table;
```

假设my\_table的col0，col1的值如下所示：

```
+------+------+
| col0 | col1 |
+------+------+
| A B | 1 |
| C D | 2 |
+------+------+
```

则 select 出的结果，如下所示：

```
+----+----+
| c0 | c1 |
+----+----+
| A  | 1  |
| B  | 1  |
| C  | 2  |
| D  | 2  |
+----+----+
```

## 使用说明 {#section_yjs_hpf_vdb .section}

UDTF在SQL中的常用方式如下：

```
select user_udtf(col0, col1, col2) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

但使用UDTF有如下使用限制：

-   同一个SELECT子句中不允许有其他表达式。

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   UDTF不能嵌套使用。

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   不支持在同一个select子句中与group by / distribute by / sort by联用。

    ```
    select user_udtf(key) as mycol ... group by mycol
    ```


## 其他UDTF示例 {#section_h4k_ppf_vdb .section}

在UDTF中，您可以读取MaxCompute的 [资源](../../../../../intl.zh-CN/用户指南/基本概念/资源.md)。利用UDTF读取MaxCompute资源的示例，如下所示。

1.  编写UDTF程序，编译成功后导出jar包（udtfexample1.jar）。

    ```
    package com.aliyun.odps.examples.udf;
    import java.io.BufferedReader;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
    import java.util.Iterator;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.UDFException;
    import com.aliyun.odps.udf.UDTF;
    import com.aliyun.odps.udf.annotation.Resolve;
    /**
     * project: example_project 
     * table: wc_in2 
     * partitions: p2=1,p1=2 
     * columns: colc,colb
     */
    @Resolve("string,string->string,bigint,string")
    public class UDTFResource extends UDTF {
      ExecutionContext ctx;
      long fileResourceLineCount;
      long tableResource1RecordCount;
      long tableResource2RecordCount;
      @Override
      public void setup(ExecutionContext ctx) throws UDFException {
      this.ctx = ctx;
      try {
       InputStream in = ctx.readResourceFileAsStream("file_resource.txt");
       BufferedReader br = new BufferedReader(new InputStreamReader(in));
       String line;
       fileResourceLineCount = 0;
       while ((line = br.readLine()) != null) {
         fileResourceLineCount++;
       }
       br.close();
       Iterator<Object[]> iterator = ctx.readResourceTable("table_resource1").iterator();
       tableResource1RecordCount = 0;
       while (iterator.hasNext()) {
         tableResource1RecordCount++;
         iterator.next();
       }
       iterator = ctx.readResourceTable("table_resource2").iterator();
       tableResource2RecordCount = 0;
       while (iterator.hasNext()) {
         tableResource2RecordCount++;
         iterator.next();
       }
     } catch (IOException e) {
       throw new UDFException(e);
     }
    }
       @Override
       public void process(Object[] args) throws UDFException {
         String a = (String) args[0];
         long b = args[1] == null ? 0 : ((String) args[1]).length();
         forward(a, b, "fileResourceLineCount=" + fileResourceLineCount + "|tableResource1RecordCount="
         + tableResource1RecordCount + "|tableResource2RecordCount=" + tableResource2RecordCount);
        }
    }
    ```

2.  添加资源到MaxCompute。

    ```
    Add file file_resource.txt;
    Add jar udtfexample1.jar;
    Add table table_resource1 as table_resource1;
    Add table table_resource2 as table_resource2;
    ```

3.  在MaxCompute中创建UDTF函数（my\_udtf）。

    ```
    create function mp_udtf as com.aliyun.odps.examples.udf.UDTFResource using 
    'udtfexample1.jar, file_resource.txt, table_resource1, table_resource2';
    ```

4.  在MaxCompute创建资源表table\_resource1、table\_resource2和物理表tmp1，并插入相应的数据。
5.  运行该UDTF。

    ```
    select mp_udtf("10","20") as (a, b, fileResourceLineCount) from tmp1;  
    返回：
    +-------+------------+-------+
    | a | b      | fileResourceLineCount |
    +-------+------------+-------+
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    | 10    | 2          | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    +-------+------------+-------+
    ```


## 复杂数据类型示例 {#section_uz1_hqf_vdb .section}

如以下代码，定义了一个有三个overloads的UDF，其中第一个用了array作为参数，第二个用了map作为参数，第三个用了struct。由于第三个overloads用了struct作为参数或者返回值，因此要求必须要对UDF class上@Resolve annotation，来指定struct的具体类型。

```
@Resolve("struct<a:bigint>,string->string")
public class UdfArray extends UDF {
  public String evaluate(List<String> vals, Long len) {
    return vals.get(len.intValue());
  }
  public String evaluate(Map<String,String> map, String key) {
    return map.get(key);
  }
  public String evaluate(Struct struct, String key) {
    return struct.getFieldValue("a") + key;
  }
}
```

您可以直接将复杂类型传入UDF中，如下所示：

```
create function my_index as 'UdfArray' using 'myjar.jar';
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from colors;
```

## HIVE UDF兼容示例 {#section_why_mqf_vdb .section}

MaxCompute 2.0支持了Hive风格的UDF，有部分的HIVE UDF、UDTF可以直接在MaxCompute上使用。

**说明：** 目前支持兼容的Hive版本为2.1.0; 对应Hadoop版本为2.7.2。UDF如果是在其他版本的Hive/Hadoop开发的，可能需要使用此Hive/Hadoop版本重新编译。

示例如下：

```
package com.aliyun.odps.compiler.hive;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
public class Collect extends GenericUDF {
  @Override
  public ObjectInspector initialize(ObjectInspector[] objectInspectors) throws UDFArgumentException {
    if (objectInspectors.length == 0) {
      throw new UDFArgumentException("Collect: input args should >= 1");
    }
    for (int i = 1; i < objectInspectors.length; i++) {
      if (objectInspectors[i] != objectInspectors[0]) {
        throw new UDFArgumentException("Collect: input oi should be the same for all args");
      }
    }
    return ObjectInspectorFactory.getStandardListObjectInspector(objectInspectors[0]);
  }
  @Override
  public Object evaluate(DeferredObject[] deferredObjects) throws HiveException {
    List<Object> objectList = new ArrayList<>(deferredObjects.length);
    for (DeferredObject deferredObject : deferredObjects) {
      objectList.add(deferredObject.get());
    }
    return objectList;
  }
  @Override
  public String getDisplayString(String[] strings) {
    return "Collect";
  }
}
```

**说明：** 对应HIVE UDF的使用请参考：

-   [HivePlugins](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)
-   [DeveloperGuide UDTF](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)
-   [GenericUDAFCaseStudy](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy)

该udf可以将任意类型、数量的参数打包成array输出。假设输出jar包名为 test.jar：

```
--添加资源
Add jar test.jar;
--创建function
CREATE FUNCTION hive_collect as 'com.aliyun.odps.compiler.hive.Collect' using 'test.jar';
--使用function
set odps.sql.hive.compatible=true;
select hive_collect(4y,5y,6y) from dual;
+------+
| _c0  |
+------+
| [4, 5, 6] |
+------+
```

**说明：** 该UDF可以支持所有的类型，包括array，map，struct等复杂类型。

使用兼容hive的udf需要注意：

-   MaxCompute的add jar命令会永久地在project中创建一个resource，所以创建udf时需要指定jar包，无法自动将所有jar包加入classpath。
-   在使用兼容的HIVE UDF的时候，需要在sql前加set语句`set odps.sql.hive.compatible=true;`语句，set语句和sql语句一起提交执行。
-   在使用兼容的HIVE UDF时，还要注意MaxCompute的[Java沙箱](intl.zh-CN/用户指南/Java沙箱.md)限制。

