# \(Optional\) Use MapReduce {#concept_blb_kxd_vdb .concept}

This topic describes how to compile and run the MapReduce WordCount example program after you install the MaxCompute client.

**Note:** Maven users can search `odps-sdk-mapred` at [Maven library](https://search.maven.org/) to download the preferred Java SDK \(available in different versions\). The configuration information is as follows:

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-mapred</artifactId>
    <version>0.26.2-public</version>
</dependency>
```

## Prerequisites {#section_ht1_byd_vdb .section}

-   JDK1.6 or a later version is installed.
-   The MaxCompute client is deployed by following the instructions provided in [Install and configure a client](../../../../../reseller.en-US/Prepare/Install and configure a client.md). For more information, see [Client](../../../../../reseller.en-US/Tools and Downloads/Client.md).

## Procedure {#section_mcx_3yd_vdb .section}

1.  After installing and configuring the client, open `odpscmd.bat` and enter the corresponding project space.
2.  Run the following statements to create input and output tables:

    ```
    CREATE TABLE wc_in (key STRING, value STRING);
    CREATE TABLE wc_out (key STRING, cnt BIGINT);
    -- Create input and output tables.
    ```

    For more information about table creation statements, see [Table operations](../../../../../reseller.en-US/User Guide/Common commands/Table operations.md).

3.  Upload data by using either of the following methods:
    -   Use the Tunnel command, for example:

        ```
        tunnel upload kv.txt wc_in
        -- Upload example data.
        ```

        The data in the kv.txt file is as follows:

        ```
        238,val_238
        186,val_86
        186,val_86
        ```

    -   Insert data by running an SQL statement, for example:

        ```
        insert into table wc_in select '238',' val_238' from (select count(*) from wc_in) a;
        ```

4.  Compile the MapReduce program and upload it to the MaxCompute client.

    In this topic, Eclipse-based MapReduce program development is used as an example. MaxCompute provides the [Eclipse development plug-in](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Install.md#) and local MapReduce debugging \(which you can use alongside [MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md#)\). These methods can help you to quickly develop MapReduce programs.

    You must create a MaxCompute project in Eclipse first, and then compile the MapReduce program \(a .jar package, for example, [Word-count-1.0.jar](../../../../../reseller.en-US/User Guide/MapReduce/Program Example/WordCount samples.md)\). After the local debugging is performed successfully, upload the compiled program to MaxCompute.  For more information, see [Install](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Install.md).

5.  Add the .jar package to the project by running the following command:

    ```
    add jar word-count-1.0.jar;
    ```

6.  In the MaxCompute client, run the following command:

    ```
    jar -resources word-count-1.0.jar -classpath /home/resources/word-count-1.0.jar com.taobao.jingfan.WordCount wc_in wc_out;
    ```

7.  In the MaxCompute client, check the results by running the following command:

    ```
    select * from wc_out;
    ```

    **Note:** If any resource is used in a Java program, you must add the resource to the -resources parameter. For more information about JAR commands, see [Commands](../../../../../reseller.en-US/User Guide/MapReduce/Function Introduction/Commands.md).


