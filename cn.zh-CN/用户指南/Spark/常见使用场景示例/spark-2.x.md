# spark-2.x {#concept_jp3_l4g_4gb .concept}

本小节给出一些常用场景的说明以及代码，包括SparkPi，WordCount、MaxCompute表的读写、GraphX、MLlib、OSS访问处理、Spark-SQL等使用场景。

**说明：** 

用户构建Spark应用的时候，由于是用MaxCompute提供的Spark客户端去提交应用所以需要注意一些依赖scope的定义。

-   spark-core spark-sql等所有spark社区发布的包，用provided scope。
-   cupid-sdk用provided scope。
-   odps-spark-datasource用默认的compile scope。

```language-xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.binary.version}</artifactId>
    <version>${spark.version}</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>cupid-sdk</artifactId>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-spark-datasource_${scala.binary.version}</artifactId>
</dependency>
```

## 案例说明 {#section_d5s_hpg_4gb .section}

-   SparkPi
    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/SparkPi.scala)
    -   提交方式

        ```language-java
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.SparkPi \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   WordCount
    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/WordCount.scala)
    -   提交方式

        ```language-java
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.WordCount \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   JavaOdpsTableReadWrite

    Java版MaxCompute Table读写接口提供以下7种使用方法。

    ```language-java
    1. read from normal table via rdd api
    2. read from single partition column table via rdd api
    3. read from multi partition column table via rdd api
    4. read with multi partitionSpec definition via rdd api
    5. save rdd into normal table
    6. save rdd into partition table with single partition spec
    7. dynamic save rdd into partition table with multiple partition spec
    ```

    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/java/com/aliyun/odps/spark/examples/JavaOdpsTableReadWrite.java)
    -   提交方式

        ```language-java
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.JavaOdpsTableReadWrite \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   MaxComputeTableReadWrite

    Scala版MaxCompute Table读写接口提供以下7种使用方法。

    ```language-java
    1. read from normal table via rdd api
    2. read from single partition column table via rdd api
    3. read from multi partition column table via rdd api
    4. read with multi partitionSpec definition via rdd api
    5. save rdd into normal table
    6. save rdd into partition table with single partition spec
    7. dynamic save rdd into partition table with multiple partition spec
    ```

    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/OdpsTableReadWrite.scala)
    -   提交方式

        ```language-java
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.OdpsTableReadWrite \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   GraphX PageRank
    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/graphx/PageRank.scala)
    -   提交方式

        ```language-java
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.graphx.PageRank \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Mllib Kmeans-ON-OSS

    KmeansModelSaveToOss

    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/mllib/KmeansModelSaveToOss.scala)
    -   提交方式

        ```language-java
        # 代码中的OSS账号信息相关需要填上，再编译提交
        val spark = SparkSession
              .builder()
              .config("spark.hadoop.fs.oss.accessKeyId", "***")
              .config("spark.hadoop.fs.oss.accessKeySecret", "***")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
              .appName("KmeansModelSaveToOss")
              .getOrCreate()
        
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.mllib.KmeansModelSaveToOss \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   OSS UnstructuredData

    SparkUnstructuredDataCompute

    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/oss/SparkUnstructuredDataCompute.scala)
    -   提交方式

        ```language-java
        # 代码中的OSS账号信息相关需要填上，再编译提交
        val spark = SparkSession
              .builder()
              .config("spark.hadoop.fs.oss.accessKeyId", "***")
              .config("spark.hadoop.fs.oss.accessKeySecret", "***")
              .config("spark.hadoop.fs.oss.endpoint", "oss-cn-hangzhou-zmf.aliyuncs.com")
              .appName("SparkUnstructuredDataCompute")
              .getOrCreate()
        
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.oss.SparkUnstructuredDataCompute \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```

-   Spark-SQL on MaxCompute Table
    -   [详细代码](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/spark-examples/src/main/scala/com/aliyun/odps/spark/examples/sparksql/SparkSQL.scala)
    -   提交方式

        ```language-java
        # 运行可能会报Table Not Found的异常，因为用户的MaxCompute Project中没有代码中指定的表
        # 可以参考代码中的各种接口，实现对应Table的SparkSQL应用
        
        Step 1. build aliyun-cupid-sdk
        Step 2. properly set spark.defaults.conf
        Step 3. bin/spark-submit --master yarn-cluster --class \
              com.aliyun.odps.spark.examples.sparksql.SparkSQL \
              ${ProjectRoot}/spark/spark-2.x/spark-examples/target/spark-examples_2.11-version-shaded.jar
        ```


