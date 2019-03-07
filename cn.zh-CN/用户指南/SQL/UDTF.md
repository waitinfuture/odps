# UDTF {#concept_bvw_yk1_1hb .concept}

本文主要介绍如何通过Java实现UDTF函数。

## UDTF示例 {#section_a4t_34f_vdb .section}

Java UDTF需要继承com.aliyun.odps.udf.UDTF类。这个类需要实现4个接口，如下表所示：

|接口定义|描述|
|:---|:-|
|public void setup\(ExecutionContext ctx\) throws UDFException|初始化方法，在UDTF处理输入数据前，调用用户自定义的初始化行为。在每个Worker内setup会被先调用一次。|
|public void process\(Object\[\] args\) throws UDFException|这个方法由框架调用，SQL中每一条记录都会对应调用一次process，process的参数为SQL语句中指定的UDTF输入参数。输入参数以Object\[\]的形式传入，输出结果通过forward函数输出。用户需要在process函数内自行调用forward，以决定输出数据。|
|public void close\(\) throws UDFException|UDTF的结束方法，此方法由框架调用，并且只会被调用一次，即在处理完最后一条记录之后。|
|public void forward\(Object …o\) throws UDFException|用户调用forward方法输出数据，每次forward代表输出一条记录。对应SQL语句UDTF的as子句指定的列。|

UDTF的程序示例，如下所示：

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

**说明：** 以上只是程序示例，关于如何在MaxCompute中运行 UDTF的方法与UDF类似，详情请参见：[运行UDF](../../../../../cn.zh-CN/快速入门/JAVA UDF开发（可选）.md)。

在SQL中可以这样使用这个UDTF，假设在MaxCompute上创建UDTF时注册函数名为user\_udtf：

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

在UDTF中，您可以读取MaxCompute的[资源](../../../../../cn.zh-CN/用户指南/基本概念/资源.md)。利用UDTF读取MaxCompute资源的示例，如下所示。

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


