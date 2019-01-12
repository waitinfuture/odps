# Develop Graph {#concept_fpk_zzn_vdb .concept}

After the [MaxCompute Java module](reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md#) is created, [Overview of Graph models](../../../../../reseller.en-US/User Guide/Graph/Summary.md#) can be developed.

## Sample Code {#section_v15_214_vdb .section}

There are some code examples of Graph in the examples directory, and you can refer to the example to get familiar with the structure of the Graph program.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12134/15472677542427_en-US.png)

## Develop a Graph Program {#section_x15_214_vdb .section}

1.  Right-click the module source code **directory src \> ** \> **main, **select **new**, and select **MaxCompute  Java**.

2.  Select the GraphLoader/Vertex type and enter the class name \(package name is supported\) in the **Name**  text box. Click **OK**, and the frame code will be automatically filled in by the template, you can continue to modify.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12134/15472677542428_en-US.png)


## Debug Graph Locally {#section_z15_214_vdb .section}

After the Graph program is developed, test your code and check whether it meets the expectations.  You can run the Graph code locally.

1.  Run the Graph program: Right-click the Driver class and select **Run**.  In the displayed Run Configuration dialog box, configure the MaxCompute project on which the Graph program runs.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12134/15472677542429_en-US.png)

2.  Click **OK**. If table data of the specified MaxCompute  project is not downloaded to warehouse, download data first. If a mock project is used or the MaxCompute project table data is downloaded, skip this step.  Then, the graph local  run framework reads specified table data in warehouse as the Graph input and runs the Graph program locally. You can view log output and result display on the console. Each time you debug locally ,  a new temporary directory is created under the Intellij directory, as shown in the following figure:

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12134/15472677542430_en-US.png)

    **Note:** For a detailed introduction to warehouse, see   Local Warehouse Directory  section in [Develop UDF](reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Develop and debug UDF.md#).


## Run the Graph Program in the Production Environment {#section_cb5_214_vdb .section}

After local debugging is complete, release the Graph program to the server and run it in the MaxCompute distributed environment.

1.  Package the Graph program to a JAR package and release it to the server.  For more imformation, please see [How to package and release Graph](reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package、Upload and Register.md#).

2.  Use the MaxCompute console integrated with MaxCompute Studio in seamless mode, that is, in the Project Explorer  Window, right-click Project and select Open in Console, and input the commands similar to the following [JAR command](../../../../../reseller.en-US/User Guide/MapReduce/Function Introduction/Commands.md#) in the console command line:

    ```
    jar -libjars xxx.jar -classpath /Users/home/xxx.jar com.aliyun.odps.graph.examples.PageRank pagerank_in pagerank_out;
    ```


For more imformation about Graph development, please see [Graph](../../../../../reseller.en-US/Quick Start/Graph.md#).

