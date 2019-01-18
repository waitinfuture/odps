# Java UDF {#concept_mxb_xn2_vdb .concept}

MaxCompute UDF includes three types: UDF, UDAF, and UDTF. This article focuses on how to implement these three functions through Java.

## Parameter and return value type {#section_uhs_43f_vdb .section}

The data types of UDF supported by MaxCompute SQL include thebasic types: bigint, double, boolean, datetime, decimal, string, tinyint, smallint, int, float, varchar, binary, and timestamp. Complex types: array, map, and struct.

-   The use of some basic types including tinyint, smallint, int, float, varchar, binary, and timestamp through Java UDF is as follows:
    -   UDTF get ‘signature’ by @Resolve annotation,  for example, `@Resolve("smallint->varchar(10)")`.
    -   UDF gets ‘signature’ by the reflection analysis ‘evaluate’. In this case, the MaxCompute built-in type and the Java type comply with one-to-one mapping.
    -   UDAF gets the signature with the @Resolve  annotation, and maxcompute2.0 supports the use of new types in annotations, for example, `@Resolve("smallint-> varchar (10 )")`.
-   JAVA UDF uses three complex data types :‘array’, ‘map’, and ‘struct’:
    -   UDAFs and UDTFs specify signature by @Resolve annotation, for example, `@Resolve("array<string>,struct<a1:bigint,b1:string>,string->map<string,bigint>,struct<b1:bigint>")`.
    -   The UDF maps the input and output types of the UDF through the signature of the evaluate method, reference is made to the mapping of the maxcompute type to the Java type. In this relationship, Array maps java.util.List, Map maps java.util.Map, and Struct maps com.aliyun.odps.data.Struct.
    -   UDAF gets the signature with the @Resolve annotation, and MaxCompute2.0 supports the use of new types in annotations, for example, `@Resolve("smallint-> varchar (10 )")`.

        **Note:** 

        -   com.aliyun.odps.data.Struct does not see field name and field type from reflection, so it must be complemented by @Resolve annotation. In other words, to use Struct in a UDF, add the @Resolve annotation to the UDF class. This annotation only affects overloads of parameters or return values that contain com.aliyun.odps.data.Struct.
        -   *Currently, only one @Resolve annotation can be provided on class. Therefore, only one overload in a UDF with a struct parameter or return value can exist.*

The following table lists the relations between MaxCompute and Java data types.

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
|array|java.util.List|
|Map|java.util.Map|
|Struct|com.aliyun.odps.data.Struct|

**Note:** 

-   The corresponding data type in Java and the return value data type is the object. Make sure that the first letter is uppercase.
-   The NULL value in SQL is represented by a NULL reference in Java; therefore, ‘Java primitive type’ is not allowed because it cannot represent a NULL value in SQL.
-   Here, Java type corresponding to the ‘array’ type is ‘list’.

## UDF {#section_g4r_wjf_vdb .section}

To implement UDF, the class ‘com.aliyun.odps.udf.UDF’ must be inherited and the ‘evaluate’ method must be applied. The ‘evaluate’ method must be a non-static public method. The parameter type and return value type of Evaluate method is considered as UDF signature in SQL. It means that the user can implement multiple evaluate methods in UDF. To call UDF, the framework must match the correct evaluate method according to the parameter type called by UDF.

**Note**：Classes with the same class name but different functional logic mustappear in different jar packages.  For example, UDF \(UDAF/UDTF\): udf1,  udf2 correspond to the resources udf1.jar and udf2.jar respectively, if both jars contain com.aliyun.UserFunction.class, when two udfs are used in the same SQL statement, the system randomly loads one of the classes. This causes inconsistency in the udf execution behavior or compilation failure.

UDF samples are as follows:

```
package org.alidata.odps.udf.examples; 
  import com.aliyun.odps.udf.UDF; 

public final class Lower extends UDF { 
  Public String evaluate (string s ){ 
    If (Stream = NULL ){ 
        return null; 
    } 
        return s.toLowerCase(); 
  } 
}
```

UDF is initialized and terminated through `void setup(ExecutionContext ctx)` and `void close()`.

The use method of UDF is similar to built-in functions in MaxCompute SQL. For more information, see [Built-in Functions](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).

## Other UDF examples {#section_kb5_v44_k2b .section}

In the following code, UDF with three overloads is defined. The first, second, and third overloads use ARRAY, MAP, and STRUCT respectively as a parameter. Since the third overloads use a struct as a parameter or return value, therefore, a `@Resolve` annotation must be placed on the UDF class to specify the specific type of struct.

```
@Resolve ("struct, string-> string ") 
public class UdfArray extends UDF { 
  public String evaluate(List vals, Long len) { 
    return vals.get(len.intValue()); 
  } 
  Public String evaluate (MAP map, string key ){ 
    return map.get(key); 
  } 
  public String evaluate(Struct struct, String key) { 
  return struct.getFieldValue("a") + key;
 } 
}
```

The user can pass the complex type directly into the UDF:

```
create function my_index as 'UdfArray' using 'myjar.jar'; 
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from colors;
```

## UDAF {#section_vdy_4kf_vdb .section}

To implement Java UDAF, inherit the class ‘com.aliyun.odps.udf.Aggregator’ and the following interfaces must be applied:

```
public abstract class Aggregator implements ContextFunction {
  @Override
  public void setup(ExecutionContext ctx) throws UDFException {                                                                   
  }
  @Override
  public void close() throws UDFException {    
  }
  /**
   * Create an aggregate buffer
   * @return Writable - Aggregate buffer
   */
  abstract public Writable newBuffer();
  /**
   * @param buffer: aggregation buffer
   * @param args: specified parameter to call UDAF in SQL
   * @throws UDFException
   */
  abstract public void iterate(Writable buffer, Writable[] args) throws UDFException;
  /**
   * generate final result
   * @param buffer
   * @return final result of Object UDAF
   * @throws UDFException
   */
  abstract public Writable terminate(Writable buffer) throws UDFException;
  abstract public void merge(Writable buffer, Writable partial) throws UDFException;
}
```

The three most important interfaces are ‘iterate’, ‘merge’, and ‘terminate’. The main logic of UDAF relies on these three interfaces. In addition, user must realize defined Writable buffer.

Take ‘achieve average calculation’ as an example and next figure describes the realization logical and computational procedure of this function in MaxCompute UDAF:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12003/15478044021855_en-US.jpg)

In the preceding figure , the input data is sliced according to a certain size.For more information about slicing, see [MapReduce](reseller.en-US/User Guide/MapReduce/Summary/MapReduce.md)\). The size of each slice is suitable for a worker to complete in the specified time. This slice size must be configured manually by the user.

The calculation process of UDAF is divided into two steps:

-   In the first step, each worker counts the data quantity and total sum in a slice. You can consider the data quantity and total sum in each slice as an intermediate result.

-   In the second step, a worker gathers the information of each slice generated in the first stage. In the final output, r.sum / r.count is the average of all input data.


Use the following UDAF encoding example to calculate the average:

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
    if (arg ! = null) {
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

**Note:** 

-   For Writable’s readFields function,  since the partial writable object can be reused, the same object readFields function is called multiple times. This function expects the entire object to be reset each time it is called. If the object contains a collection, it must be emptied.
-   The use method of UDAF is similar to aggregation functions in MaxCompute SQL. For more information, see [Aggregation Functions](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md).
-   How to run UDTF is similar to UDF. For more information, see [Java UDF Development](../../../../../reseller.en-US/Quick Start/Java UDF Development.md).

## UDTF {#section_a4t_34f_vdb .section}

Java UDTF class must inherit the class ‘com.aliyun.odps.udf.UDTF’. This class has four interfaces:

|Interface Definition|Description|
|:-------------------|:----------|
|public void setup\(ExecutionContext ctx\) throws UDFException|The initialization method to call user-defined initialization behavior before UDTF processes the input data. ‘Setup’ will be called first and once for each worker.|
|public void process\(Object\[\] args\) throws UDFException|The framework calls this method. Each record in SQL calls ‘process’ once accordingly. The parameters of ‘process’ are the specified UDTF input parameters in SQL. The input parameters are passed in as Object\[\], and the results are output through ‘forward’ function. The user must call ‘forward’ in the ‘process’ function by itself to determine the output data.|
|public void close\(\) throws UDFException|The termination method of UDTF. The framework calls this method, and only once; that is, after processing the last record.|
|public void forward\(Object …o\) throws UDFException|The user calls the ‘forward’ method to output data. Each ‘forward’ represents the output of a record, corresponding to the column specified by UDTF 'as’ clause in SQL.|

A UDTF program sample is as follows:

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

**Note:** The preceding example is for reference only. How to run UDTF is similar to using UDF. For more information, see [Java UDF Development](../../../../../reseller.en-US/Quick Start/Java UDF Development.md).

In SQL,use this UDTF as the following example. Suppose that the register function name in MaxCompute is  ‘user\_udtf’.

```
select user_udtf(col0, col1) as (c0, c1) from my_table;
```

Suppose the values of  col0 and col1 in my\_table are:

```
+------+------+
| col0 | col1 |
+------+------+
| A B | 1 |
| C D | 2 |
+------+------+
```

Then the ‘SELECT’  result is:

```
+----+----+
| c0 | c1 |
+----+----+
| A | 1 |
| B | 1 |
| C | 2 |
| D | 2 |
+----+----+
```

## Instructions {#section_yjs_hpf_vdb .section}

UDTFs are often used as following in SQL:

```
select user_udtf(col0, col1) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

But using UDTF has the following limits:

-   Other expressions are not allowed in the same SELECT clause:

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   UDTF  cannot be nested.

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   It cannot be used with ‘group by / distribute by / sort by’  in the same SELECT clause.

    ```
    select user_udtf(key) as mycol ... group by mycol
    ```


## Other UDTF Examples {#section_h4k_ppf_vdb .section}

In UDTF, learn more aboutMaxCompute [Resources](../../../../../reseller.en-US/User Guide/Definition/Resource.md).  The following describes how to use UDTFs to read MaxCompute resources:

1.  Compile a UDTF program. Once the compilation is successful, export the Jar package \(udtfexample1.jar\).

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
       while ((line = br.readLine()) ! = null) {
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

2.  Add resources in MaxCompute:

    ```
    Add file file_resource.txt;
    Add jar udtfexample1.jar;
    Add table table_resource1 as table_resource1;
    Add table table_resource2 as table_resource2;
    ```

3.  Create UDTF \(my\_udtf\) in MaxCompute:

    ```
    create function mp_udtf as com.aliyun.odps.examples.udf.UDTFResource using 
    'udtfexample1.jar, file_resource.txt, table_resource1, table_resource2';
    ```

4.  Create the resource tables: table\_resource1, table\_resource2 and the physical table tmp1 in MaxCompute. Insert corresponding data into the tables.
5.  Run this UDTF.

    ```
    select mp_udtf("10","20") as (a, b, fileResourceLineCount) from tmp1;  
    Return result:
    +-------+------------+-------+
    | a | b | fileResourceLineCount |
    +-------+------------+-------+
    | 10 | 2 | fileResourceLineCount=3|tableResource1RecordCount=0|tableResource2RecordCount=0 |
    | 10 | 2 | fileresourcelinecount = 3 | tableResource1RecordCount = 0 | tableResource2RecordCount = 0 |
    +-------+------------+-------+
    ```


## UDTF Examples—Complex Data Types {#section_uz1_hqf_vdb .section}

The code in the following example defines UDF with three overloads. The first overload uses ‘array’ as the parameter; the second uses ‘map’ as the parameter; and the third uses ‘struct’ as the parameter. Since the third overload uses ‘struct’ as the parameter or returned value, the UDF class must havethe @Resolve annotation to specify the  specific type of ‘struct’.

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

Users can pass in the complex data type in the UDF:

```
create function my_index as 'UdfArray' using 'myjar.jar';
select id, my_index(array('red', 'yellow', 'green'), colorOrdinal) as color_name from colors;
```

## Hive UDF Compatibility Example {#section_why_mqf_vdb .section}

MaxCompute 2.0 supports Hive-style UDFs. Some Hive UDFs and UDTFs can be used directly in MaxCompute.

**Note:** Currently, the compatible Hive version is 2.1.0, and the corresponding Hadoop version is 2.7.2. UDFs that are developed in other versions of Hive/Hadoop may need to be recompiled using this Hive/Hadoop version.

Example:

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
      if (objectInspectors[i] ! = objectInspectors[0]) {
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

**Note:** For the use of  Hive UDF, see:

-   [https://cwiki.apache.org/confluence/display/Hive/HivePlugins](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)
-   [https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)
-   [https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy](https://cwiki.apache.org/confluence/display/Hive/GenericUDAFCaseStudy)

The UDF can pack any type and amount of parameters into array to output. Suppose that the output jar package is named test.jar:

```
--Add resource
Add jar test.jar;
--Create function
CREATE FUNCTION hive_collect as 'com.aliyun.odps.compiler.hive.Collect' using 'test.jar';
--Use function
set odps.sql.hive.compatible=true;
select hive_collect(4y,5y,6y) from dual;
+------+
| _c0 |
+------+
| [4, 5, 6] |
+------+
```

**Note:** The UDF supports all data types, including array, map, struct, and other complex types.

Note: 

-   MaxCompute’s add  jar command permanently creates a resource in the project, specify the jar when creating an UDF, but you cannot automatically add all jars to the classpath.
-   To use compatible Hive UDF, add `set  odps.sql.hive.compatible=true;` opposite the SQL statement, and submit it with SQL statement.
-   When using compatible Hive UDFs, you must pay attention to [JAVA sandbox](reseller.en-US/User Guide/Java Sandbox.md) limits of MaxCompute.

