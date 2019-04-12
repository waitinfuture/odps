# Spark概述 {#concept_jzp_wlb_kgb .concept}

MaxCompute Spark是MaxCompute提供的兼容开源的Spark计算服务，它在统一的计算资源和数据集权限体系之上，提供Spark计算框架，支持用户以熟悉的开发使用方式提交运行Spark作业，以满足更丰富的数据处理分析场景。

## 关键特性 {#section_wkg_jpb_kgb .section}

-   支持原生多版本Spark作业

    社区原生Spark运行在MaxCompute里，完全兼容Spark的API，支持多个Spark版本同时运行。MaxCompute Spark提供原生的Spark WebUI供用户查看。

-   统一的计算资源

    MaxCompute Spark像MaxCompute SQL/MR等任务类型一样，运行在MaxCompute项目开通的统一计算资源中。

-   统一的数据和权限管理

    完全遵循MaxCompute项目的权限体系，在访问用户权限范围内安全地查询数据。

-   与开源系统相同的使用体验

    MaxCompute Spark会和开源保持相同的体验和用户使用习惯，例如开源应用的UI、在线交互等。开源UI对于开源应用的调试至关重要，MaxCompute Spark提供原生的开源实时UI，和查询历史日志的功能。某些开源应用提供交互式体验，即把后台引擎拉起后，进行实时交互。


## 系统结构 {#section_mbt_b5c_kgb .section}

MaxCompute Spark是阿里云通过Spark on MaxCompute的解决方案，让原生Spark能够运行在MaxCompute当中。该服务的架构图如下。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/92656/155504861836635_zh-CN.png)

左侧是原生Spark的架构图，右边Spark on MaxCompute运行在阿里云自研的Cupid的平台之上，该平台可以原生支持开源社区Yarn所支持的计算框架，如Spark等。

