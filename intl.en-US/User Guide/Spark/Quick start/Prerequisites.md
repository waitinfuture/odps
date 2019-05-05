# Prerequisites {#concept_f1r_5tc_kgb .concept}

This topic describes how to use MaxCompute Spark to create a Spark on MaxCompute application and submit and run a Spark job in a MaxCompute project.

## Download a MaxCompute Spark release package {#section_zsg_2rb_kgb .section}

You can use a MaxCompute Spark release package and the spark-submit method to submit a job. The MaxCompute cluster supports both Spark-1.x and Spark-2.x. We recommend that you download an appropriate version based on your needs.

-   To download Spark-1.x, go to [Spark-1.6.3](http://repo.aliyun.com/download/spark-1.6.3-public.tar.gz).
-   To download Spark-2.x, go to [Spark-2.3.0](http://repo.aliyun.com/download/spark-2.3.0-public.tar.gz).

**Note:** The version that the server starts is based on the version of MaxCompute Spark with which you submit tasks.

The directory structure of the release package matches that of the community Spark. Specifically, the conf directory uses the spark-defaults.conf file as the default configuration parameter. Note that jobs in the bin directory can only be submitted by using the spark-submit method. The spark-shell and spark-sql methods are not supported.

```language-java
|-- bin
|-- conf
|-- cupid
|-- lib
|-- python
|-- sbin
|-- RELEASE
```

## Prepare the environment {#section_cc4_bxr_pgb .section}

-   Set JAVA\_HOME.

    ```
    # JDK 1.7+ is recommended.
    # JDK 1.8+ is the most recommended.
    export JAVA_HOME=/path/to/jdk
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export PATH=$JAVA_HOME/bin:$PATH
    ```

-   Set SPARK\_HOME.

    ```
    export SPARK_HOME=/path/to/spark_extracted_package
    export PATH=$SPARK_HOME/bin:$PATH
    ```


Maven version 3.3.3 or later must be used for local development.

