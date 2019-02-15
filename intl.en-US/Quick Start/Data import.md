# Data import {#concept_amj_mjy_5db .concept}

This article gives you a detailed introduction to Tunnel command importing data, MaxCompute Studio importing data, and other ways of importing data to MaxCompute.

You can import and export  data through MaxCompute using the following methods:

-    [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md), directly on the console.
-   MaxCompute Studio in the visualization method. For more information, see [Import and Export Data](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Manage data and resources/Import and export data.md).
-   Java tools written with the SDK provided by [Tunnel](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel SDK/Summary.md).
-   Flume and Fluentd plug-ins.
-   Data is imported and exported through DataWorks. For more information, see [Data Integration overview](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#).

For data export, see commands about downloading in [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).

## Tunnel commands Follow the steps below to import data using tunnel commands. {#section_v5p_zjy_5db .section}

1.  Prepare the data

    In this example, the local file wc\_example.txt is saved into the directory D:\\odps\\odps\\bin. The content is as follows:

    ```
    I LOVE CHINA! MY NAME IS MAGGIE.
    I LIVE IN HANGZHOU! I LIKE PLAYING BASKETBALL!
    ```

2.  Create a MaxCompute table

    To import the data created in the preceding step, a MaxCompute table must be created.

    ```
    CREATE TABLE wc_in (word string);
    ```

3.  Run tunnel command

    Import the data to the MaxCompute console by running the tunnel command as follows:

    ```
    tunnel upload D:\odps\odps\bin\wc_example.txt wc_in;
    ```

4.  After the command has run successfully, check the records in the table wc\_in.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15502233481484_en-US.png)

    **Note:** 

    -   For more information of Tunnel commands, for example, how to import data into a partitioned table, see [Tunnel Operation](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).
    -   If multiple columns are in the table, you can specify column separators by using-fd parameter.

## MaxCompute studio imports data {#section_unz_vky_5db .section}

Follow the steps below to import data using MaxCompute Studio. Before using MaxCompute Studio, make sure that you have [installed MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md) and [configured Project Space Connection](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md).

1.  Data Preparation

    In this example, the local file wc\_example.txt is saved into the directory D:\\odps\\odps\\bin. The content is as follows:

    ```
    I LOVE CHINA! MY NAME IS MAGGIE.
    I LIVE IN HANGZHOU! I LIKE PLAYING BASKETBALL!
    ```

2.  Create a MaxCompute table

    To import the data created in the preceding step, a MaxCompute table must be created first. Right-click **tables&views** in the project and operate as follows:

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15502233491490_en-US.png)

    If the statement is executed successfully, then the table has been created.

3.  Upload data files

    Right-click the newly created table wc\_in in the tables&views list

    **Note:** If the table name does not appear in the list, click the **refresh** button.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15502233491522_en-US.png)


## Tunnel SDK {#section_gdq_z4y_5db .section}

The following is a scenario example to show you how to upload data using the Tunnel SDK.

**Scenario**

Upload data into MaxCompute, where the project is odps\_public\_dev, the table name is tunnel\_sample\_test and the partitions are pt=20150801,dt=hangzhou.

**Procedure**

1.  Create a table and add corresponding partitions:

    ```
    CREATE TABLE IF NOT EXISTS tunnel_sample_test( 
    id STRING, 
    name STRING)
    PARTITIONED BY (pt STRING, dt STRING); --Create a table.
    ALTER TABLE tunnel_sample_test 
    ADD IF NOT EXISTS PARTITION (pt='20150801',dt='hangzhou'); --Add the partitions.
    ```

2.  Create the program directory structure of UploadSample as follows:

    ```
    |---pom.xml
    |---src 
        |---main 
            |---java 
                |---com 
                    |---aliyun 
                        |---odps 
                            |---tunnel 
                                |---example 
                                    |---UploadSample.java
    ```

    Directory description:

    -   pom.xml: maven program file.
    -   UploadSample: tunnel source file.  
3.  Write UploadSample program as follows:

    ```
    
    package com.aliyun.odps.tunnel.example;
    import java.io.IOException;
    import java.util.Date;
    
    import com.aliyun.odps.Column;
    mport com.aliyun.odps.Odps;
    import com.aliyun.odps.PartitionSpec;
    import com.aliyun.odps.TableSchema;
    import com.aliyun.odps.account.Account;
    import com.aliyun.odps.account.AliyunAccount;
    Import com. aliyun. ODPS. Data. record;
    import com.aliyun.odps.data.RecordWriter;
    import com.aliyun.odps.tunnel.TableTunnel;
    import com.aliyun.odps.tunnel.TunnelException;
    import com.aliyun.odps.tunnel.TableTunnel.UploadSession;
    
    public class UploadSample {
    private static String accessId = "####";
    private static String accessKey = "####";
    private static String tunnelUrl = "http://dt.odps.aliyun.com";
    
    private static String odpsUrl = "http://service.odps.aliyun.com/api";
    
    private static String project = "odps_public_dev";
    private static String table = "tunnel_sample_test";
    private static String partition = "pt=20150801,dt=hangzhou";
    
    public static void main(String args[]) {
    Account account = new AliyunAccount(accessId, accessKey);
    Odps odps = new Odps(account);
    odps.setEndpoint(odpsUrl);
    odps.setDefaultProject(project);
    try {
    Tabletunnel tunnel = new tabletunnel (ODPS );
    tunnel.setEndpoint(tunnelUrl);
    PartitionSpec partitionSpec = new PartitionSpec(partition);
    UploadSession uploadSession = tunnel.createUploadSession(project,
    table, partitionSpec);
    
    
    System.out.println("Session Status is : "
    + uploadSession.getStatus().toString());
    
    
    TableSchema schema = uploadSession.getSchema();
    RecordWriter recordWriter = uploadSession.openRecordWriter(0);
    Record record = uploadSession.newRecord();
    for (int i = 0; i < schema.getColumns().size(); i++) {
    Column column = schema.getColumn(i);
    switch (column.getType()) {
    case BIGINT:
    record.setBigint(i, 1L);
    break;
    case BOOLEAN:
    record.setBoolean(i, true);
    Break;
    case DATETIME:
    record.setDatetime(i, new Date());
    break;
    case DOUBLE:
    record.setDouble(i, 0.0);
    break;
    case STRING:
    record.setString(i, "sample");
    break;
    default:
    throw new RuntimeException("Unknown column type: "
    + column.getType());
    }
    }
    for (int i = 0; i < 10; i++) {
    recordWriter.write(record);
    }
    recordWriter.close();
    UPR session. Commit (New long [] {0l });
    System.out.println("upload success!") ;
    
    } catch (TunnelException e) {
    e.printStackTrace();
    } catch (IOException e) {
    e.printStackTrace();
    }
    }
    }
    ```

    **Note:** The configuration of the accesskeyid and the accesskeysecret is omitted here. Please change your information when you are actually running.

4.  The configuration of pom.xmlis as follows:

    ```
    <? xml version="1.0" encoding="UTF-8"? >
    <project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.aliyun.odps.tunnel.example</groupId>
    <artifactId>UploadSample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
      <dependency>
        <groupId>com.aliyun.odps</groupId>
        <artifactId>odps-sdk-core</artifactId>
        <version>0.20.7-public</version>
      </dependency>
    </dependencies>
    <repositories>
      <repository>
      <id>alibaba</id>
      <name>alibaba Repository</name>
      <url>http://mvnrepo.alibaba-inc.com/nexus/content/groups/public/</url>
      </repository>
    </repositories>
    </project>
    
    
    ```

5.  Compile and run

    Compile the program UploadSample:

    ```
    mvn package
    ```

    Run the program UploadSample. Here, Eclipse is used to import the Maven project.

    1.  Right-click on the **Java program** and click**Import \>** \> **Maven \> ** \> **Existing Maven Projects**, The settings are as follows:

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15502233491545_en-US.PNG)

    2.  Right-click on **UploadSample.java** and click**Run As** \> **Run Configurations**, as follows:

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15502233491547_en-US.PNG)

    3.  Click **Run**After running successfully, the console shows as follows:

        ```
        Session Status is : NORMAL
        upload success!
        ```

6.  Check running result.

    Input the following statement on the console:

    ```
    select * from tunnel_sample_test;
    ```

    The result is shown as follows:

    ```
    +----+------+----+----+
    | id | name | pt | dt |
    +----+------+----+----+
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    | sample | sample | 20150801 | hangzhou |
    +----+------+----+----+
    ```

    **Note:** 

    -   As an independent service in MaxCompute, Tunnel has an exclusive access port provided for users. When you download data with MaxCompute Tunnel over the Alibaba Cloud intranet, MaxCompute does not bill you for the traffic produced by this operation. The intranet address is only valid for cloud products in the Shanghai region.
    -   For more information about MaxCompute Alibaba intranet and public network access addresses, see [Access domains and data centers](../../../../../reseller.en-US/Prepare/Configure Endpoint.md).

## Other import methods {#section_vcl_r5y_5db .section}

In addition to MaxCompute Console and Tunnel Java SDK, data can also be imported through Alibaba Cloud DTplus products, Sqoop, Fluentd, Flume, LogStash , and other tools.And more [Tools](../../../../../reseller.en-US/User Guide/Data upload and download/Data upload and download tools.md).

