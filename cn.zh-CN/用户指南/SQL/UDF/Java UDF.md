# Java UDF {#concept_mxb_xn2_vdb .concept}

MaxCompute的UDF包括UDF，UDAF 和UDTF三种函数，本文将重点介绍如何通过Java实现这三种函数。

## 参数与返回值类型 {#section_uhs_43f_vdb .section}

MaxCompute 2.0版本升级后，Java UDF支持的数据类型从原来的Bigint，String，Double，Boolean扩展了更多基本的数据类型，同时还扩展支持了ARRAY，MAP，STRUCT等复杂类型。

-   Java UDF使用新基本类型的方法，如下所示：
    -   UDTF通过@Resolve注解来获取signature，如：`@Resolve("smallint->varchar(10)")`。
    -   UDF通过反射分析evaluate来获取signature，此时MaxCompute内置类型与 Java类型符合一一映射关系。
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

可以通过实现`````void setup(ExecutionContext ctx)`````和`void close()`来分别实现UDF的初始化和结束代码。

UDF的使用方式与MaxCompute SQL中普通的内建函数相同，详情请参见 [内建函数](cn.zh-CN/用户指南/SQL/内建函数/数学函数.md)。

使用Writable类型实现Concat的示例如下：

```
package com.aliyun.odps.udf.example;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF;
public class MyConcat extends UDF {
  private Text ret = new Text();
  public Text evaluate(Text a, Text b) {
    if (a == null || b == null) {
      return null;
    }
    ret.clear();
    ret.append(a.getBytes(), 0, a.getLength());
    ret.append(b.getBytes(), 0, b.getLength());
    return ret;
  }
}
```

**说明：** 

所有的Writable类型所在的package为com.aliyun.odps.io。

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

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12003/15519494011855_zh-CN.jpg)

在上图中，输入数据被按照一定的大小进行分片（有关分片的描述请参见 [MapReduce](cn.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)），每片的大小适合一个worker在适当的时间内完成。这个分片大小的设置需要您手动配置完成。

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
-   UDAF在SQL中的使用语法与普通的内建聚合函数相同，详情请参见 [聚合函数](cn.zh-CN/用户指南/SQL/内建函数/聚合函数.md)。
-   关于如何运行UDTF的方法与 UDF 类似，详情请参见 [运行 UDF](../../../../../cn.zh-CN/快速入门/JAVA UDF开发（可选）.md)。

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

