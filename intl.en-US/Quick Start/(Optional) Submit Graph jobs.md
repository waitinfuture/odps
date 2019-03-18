# \(Optional\) Submit Graph jobs {#concept_gzg_1c2_vdb .concept}

This topic uses the [SSSP algorithm](../../../../../reseller.en-US/User Guide/Graph/Examples/SSSP.md) as an example to describe how to submit Graph jobs.

Submitting a [Graph](../../../../../reseller.en-US/User Guide/Graph/Summary.md) job is similar to submitting a [MapReduce](../../../../../reseller.en-US/User Guide/MapReduce/Function Introduction/Commands.md) job. Maven users can search `odps-sdk-graph` at [Maven library](http://search.maven.org/) to download the preferred Java SDK \(available in different versions\). The configuration information is as follows:

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-graph</artifactId>
    <version>0.20.7</version>
</dependency>
```

## Procedure {#section_esm_lc2_vdb .section}

1.  Log on to the console and run `odpscmd`.
2.  Create input and output tables.

    ```
    create table sssp_in (v bigint, es string);
    create table sssp_out (v bigint, l bigint);
    ```

    For more information about table creation statements, see [Table operations](../../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table Operations.md#).

3.  Upload data.

    Local data is as follows:

    ```
    2, 2, 3, 4, 4
    2 1:2,3:2,4:1
    3 1:1,2:2,5:1
    4 1:4,2:1,5:1
    5 3:1,4:1
    ```

    A space is used to separate two columns.

    ```
    tunnel u -fd " " sssp.txt sssp_in;
    ```

4.  Compile an SSSP example.

    Compile and debug the [SSSP example](../../../../../reseller.en-US/User Guide/Graph/Examples/SSSP.md) on your local PC by following the instructions provided in [Detailed introduction](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Detailed introduction.md). In this example, the code is packaged as the odps-graph-example-sssp.jar file.

    **Note:** You only need to package the SSSP code. You do not need to package the SDK in the odps-graph-example-sssp.jar file.

5.  Add JAR resources.

    ```
    add jar $LOCAL_JAR_PATH/odps-graph-example-sssp.jar
    ```

    **Note:** For more information, see [Resources operations](../../../../../reseller.en-US/User Guide/Common commands/Resources operations.md).

6.  Run SSSP.

    ```
    jar -libjars odps-graph-example-sssp.jar -classpath $LOCAL_JAR_PATH/odps-graph-example-sssp.jar com.aliyun.odps.graph.example.SSSP 1 sssp_in sssp_out;
    ```

    JAR commands are used to run MaxCompute Graph jobs in the same way as the commands for running [MapReduce](../../../../../reseller.en-US/User Guide/MapReduce/Function Introduction/Commands.md) jobs.

    When a Graph job is running, the corresponding instance ID, progress, and result summary are displayed in the command output, for example:

    ```
    ID = 20130730160742915gl205u3
    2013-07-31 00:18:36 SUCCESS
    Summary:
    Graph Input/Output
    Total input bytes=211
    Total input records = 5
    Total output bytes=161
    Total output records=5
    Graph_input _ [BSP. sssp_in] _ bytes = 211
    Graph_input _ [BSP. sssp_in] _ records = 5
    graph_output_[bsp.sssp_out]_bytes=161
    Graph_output _ [BSP. sssp_out] _ records = 5
    Graph statistics
    Total edges=14
    Total halted vertices=5
    Total sent messages=28
    Total supersteps=4
    Total vertices=5
    Total workers=1
    Graph timers
    Average superstep time (milliseconds) = 7
    Load time (milliseconds)=8
    Max superstep time (milliseconds) =14
    Max time superstep=0
    Min superstep time (milliseconds) = 5
    Min time superstep=2
    Setup Time (milliseconds) = 277
    Shutdown Time (milliseconds) = 20
    Total superb time (milliseconds) = 30
    Total time (milliseconds)=344
    OK
    ```

    **Note:** To use the Graph function, you only need to open and submit a [Graph job](../../../../../reseller.en-US/User Guide/Graph/Summary.md#).


