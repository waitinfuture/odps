# UDF概述 {#concept_dkq_tn2_vdb .concept}

UDF全称为User Defined Function，即用户自定义函数。MaxCompute提供很多内建函数来满足您的计算需求，同时您还可以通过创建自定义函数来满足不同的计算需求。

UDF在使用上与普通的[内建函数](intl.zh-CN/用户指南/SQL/内建函数/数学函数.md#) 类似，Java和MaxCompute的数据类型的对应关系，请参见[参数与返回值类型](intl.zh-CN/用户指南/SQL/UDF/Java UDF.md#) 。

如果您使用[Maven](http://search.maven.org/)，可以从 [Maven 库](http://search.maven.org/)中搜索odps-sdk-udf，从而获取不同版本的Java SDK，相关配置信息如下所示：

```language-xml
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
   <version>0.29.10-public</version>
</dependency>
```

MaxCompute支持的UDF有三种：

|UDF 分类|描述|
|:-----|:-|
|User Defined Scalar Function（通常也称之为UDF）|用户自定义标量值函数（User Defined Scalar Function）。其输入与输出是一对一的关系，即读入一行数据，写出一条输出值 。|
|UDTF（User Defined Table Valued Function）|自定义表值函数，是用来解决一次函数调用输出多行数据场景的，也是唯一能返回多个字段的自定义函数。而UDF只能一次计算输出一条返回值。|
|UDAF（User Defined Aggregation Function）|自定义聚合函数，其输入与输出是多对一的关系， 即将多条输入记录聚合成一条输出值。可以与SQL中的Group By语句联用。具体语法请参见[聚合函数](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)。|

**说明：** 

-   UDF广义的说法代表了自定义标量函数，自定义聚合函数及自定义表函数三种类型的自定义函数的集合。狭义来说，仅代表用户自定义标量函数。文档会经常使用这一名词，请读者根据文档上下文判断具体含义 。
-   SQL语句中有使用自定义的函数，提示内存不够。请配置 `set odps.sql.udf.joiner.jvm.memory=xxxx;`。原因是数据量太大并且有倾斜，任务超出默认设置的内存。

MaxCompute的UDF支持跨项目分享，即在project\_a中可以使用project\_b的UDF。具体的授权可以参考安全指南的[授权](../../../../../intl.zh-CN/安全指南/安全功能详解/用户及授权管理/授权.md#)文档中的“跨项目空间Table、Resource、Function分享示例”章节，使用时写法如`select other_project:udf_in_other_project(arg0, arg1) as res from table_t;`。

## UDF示例 {#section_qt4_kdf_vdb .section}

UDF 的相关示例请参见[UDF示例](intl.zh-CN/用户指南/SQL/UDF/Java UDF.md#) 。

