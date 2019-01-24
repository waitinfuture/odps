# PySpark {#concept_lbf_1mg_4gb .concept}

本文以一个简单的PySpark读写MaxCompute表作为例子讲解整个开发过程。

-   1.下载[odps\_sdk.py](https://github.com/aliyun/aliyun-cupid-sdk/blob/3.3.2-public/spark/spark-2.x/datasource/src/main/python/odps_sdk.py)到本地。
-   2.下载[odps-spark-datasource-1.0.3.jar](http://repo.aliyun.com/download/odps-spark-datasource-1.0.3.jar)至本地，截止至文档发布的时刻，1.0.3为最新版本，如版本更新请自行更新。
-   3.准备Python版Spark app，如下：
-   4.example.py代码示例：

    ```language-python
    1.from odps.odps_sdk import OdpsOps
    2.from pyspark import SparkContext, SparkConf
    3.from pyspark.sql import SQLContext, DataFrame
    4.
    5.if __name__ == '__main__':
    6.    conf = SparkConf().setAppName("pyspark_odps_test")
    7.    sc = SparkContext(conf=conf)
    8.    sql_context = SQLContext(sc)
    9.
    10.    # Normal ODPS Table READ
    11.    # Params: partitions [default value]
    12.    # Params: num_partition [default value]
    13.    normal_df = OdpsOps.read_odps_table(sql_context, "cupid_testa1", "cupid_wordcount")
    14.    for i in normal_df.sample(False, 0.01).collect():
    15.        print i
    16.    print "Read normal odps table finished"
    17.
    18.    # Partition ODPS Table READ
    19.    # Params: partitions ["pt1='part1',pt2='part1'", "pt1='part1',pt2='part2'"]
    20.    # Params: num_partition [default value]
    21.    partition_df = OdpsOps.read_odps_table(sql_context, "cupid_testa1", "cupid_partition_table1",
    22.                                 partitions=["pt1='part1',pt2='part1'", "pt1='part1',pt2='part2'"])
    23.    for i in partition_df.sample(False, 0.01).collect():
    24.        print i
    25.    print "Read partition odps table finished"
    26.
    27.    # Normal ODPS Table Write
    28.    # Params: partition_str [default value]
    29.    # Params: is_overwrite [default value]
    30.    OdpsOps.write_odps_table(sql_context, normal_df.sample(False, 0.001), "cupid_testa1", "cupid_wordcount_py")
    31.    print "Write normal odps table finished"
    32.
    33.    # Partition ODPS Table Write
    34.    # Params: partition_str "pt1='part3',pt2='part4'"
    35.    # Params: is_overwrite [default value]
    36.    OdpsOps.write_odps_table(sql_context, partition_df.sample(False, 0.001), "cupid_testa1", "cupid_partition_table1_py",
    37.                             "pt1='part1',pt2='part3'")
    38.    print "Write partition odps table finished"
    ```

    Cluster模式运行

    ```language-python
    spark-submit --master yarn-cluster --jars odps-spark-datasource-1.0.3.jar --py-files odps_sdk.py example.py
    ```


