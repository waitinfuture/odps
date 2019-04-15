# Spark overview {#concept_jzp_wlb_kgb .concept}

MaxCompute Spark is an open-source service and computing framework that can fulfill your data processing and analysis needs. Equipped with unified computing resource and data set permissions, MaxCompute Spark allows you to submit and run jobs by using your preferred development methods.

## Spark features {#section_wkg_jpb_kgb .section}

-   Supports different versions of native Spark jobs.

    MaxCompute is fully compatible with the APIs of the native Spark that runs in MaxCompute. Different versions of Spark can run in MaxCompute at the same time. MaxCompute Spark provides users with native Spark Web UIs.

-   Runs in unified computing resources.

    Similar to MaxCompute SQL and MaxCompute MapReduce, MaxCompute Spark runs in the unified computing resources activated for MaxCompute projects.

-   Supports unified data and permission management.

    Spark fully complies with the permission system of MaxCompute projects, allowing users to query data within their permissions.

-   Provides the same experience as open-source systems.

    MaxCompute Spark provides the same user experience as open-source systems, both in terms of an open-source application UI and online interaction. Specifically, MaxCompute Spark supports native, open-source, and real-time UIs that are essential for debugging open-source applications. Moreover, MaxCompute Spark provides the historical log query function.


## System architecture {#section_mbt_b5c_kgb .section}

-   MaxCompute Spark allows native Spark to run in MaxCompute using the Spark on MaxCompute solution. The following figure shows the architecture.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/92656/155532186136635_en-US.png)

    The left part shows the architecture of native Spark, and the right part shows the architecture of Spark on MaxCompute, which runs on the cupid platform developed by Alibaba Cloud. The cupid platform also supports computing frameworks \(such as Spark\) that are supported by Yarn, an open-source community.


