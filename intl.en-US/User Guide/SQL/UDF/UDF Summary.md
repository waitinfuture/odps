# UDF Summary {#concept_dkq_tn2_vdb .concept}

MaxCompute provides many built-in functions to meet the computing requests of the users.

A User Defined Function \(UDF\) is similar to any other [Built-in Function](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md#). For the corresponding relationship between Java and MaxCompute data types, see [Parameters and Return Value Types](reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

If you use [Maven](http://search.maven.org/) to search “odps-sdk-udf” from [Maven](http://search.maven.org/) to get different versions of Java  SDK,the configuration is as follows:

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
   <version>0.29.10-public</version>
</dependency>
```

In MaxCompute, you can expand two types of UDF:

|UDF Class|Description|
|:--------|:----------|
| UDF\(User Defined Scalar Function\)|User Defined Scalar function. The relationship between input and output is a one-to-one relationship. Read a row data and write an output value.|
|UDTF \(UserDefined Table Valued Function\)|User-defined table valued functions are used in scenarios where the calling of one function leads to multiple rows of data being output. It is a unique user-defined function which can return multiple fields, while UDFcan only output a return value.|
|UDAF（User Defined Aggregation Function）|User Defined Aggregation Function \(UDAF\), the relationship between its input and output is one-to-many relationships. That is to aggregate multiple input records to an output value. It can be used with a Group By clause..  For more information, see [Aggregate functions](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#)[Aggregation Functions](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md).|

**Note:** 

-   UDF stands for the set of user-defined functions, including User Defined Scalar Function, User Defined Aggregation Function and User Defined Table Valued Function. In a narrower sense, it represents user User Defined Scalar Function. The document uses this term frequently and the readers can judge the specific meaning according to the context .
-   If the system prompts that memory is insufficient with an UDF involved in the SQL statement,  configure `set odps.sql.udf.joiner.jvm.memory=xxxx;` to resolve this issue. This is because the data is huge and data skew also exists., This leads the memory size to occupythe task, which exceeds the default memory size.

MaxCompute UDF supports cross-project sharing. A UDF in project\_b can be used in project\_a.  For more information, , see Authorization in Security Guide documentation. other\_project:udf\_in\_other\_project\(arg0, arg1\) as res from table\_t;。

## UDF Examples {#section_qt4_kdf_vdb .section}

Please see [UDF Example](reseller.en-US/User Guide/SQL/UDF/Java UDF.md#) in Quick Start Volume.

