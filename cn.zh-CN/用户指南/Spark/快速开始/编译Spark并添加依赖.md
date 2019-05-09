# 编译Spark并添加依赖 {#concept_xqm_wlf_4gb .concept}

环境配置完成后，您需要配置Spark-defaults.conf文件，编译Spark，并添加Maven依赖。

## 配置Spark-defaults.conf {#section_wsd_j4f_4gb .section}

在**$SPARK\_HOME/conf**路径下存在spark-defaults.conf.template文件，这个可以作为spark-defaults.conf的模版，需要在该文件中设置MaxCompute相关的账号信息后，才可以提交Spark任务到MaxCompute。默认配置内容如下，将空白部分根据实际的账号信息填上即可，其余的配置可以保持不变。

```language-java
# OdpsAccount Info Setting
spark.hadoop.odps.project.name =
spark.hadoop.odps.access.id =
spark.hadoop.odps.access.key =
# 配置Spark客户端连接访问MaxCompute项目项目的endpoint，请根据网络环境以及region填写对应MaxCompute Endpoint
spark.hadoop.odps.end.point = http://service.cn.maxcompute.aliyun.com/api
# Spark运行环境endpoint，请配置为所在region的MaxCompute VPC内网Endpoint
spark.hadoop.odps.runtime.end.point = http://service.cn.maxcompute.aliyun-inc.com/api

# 以下配置保持不变
spark.sql.catalogImplementation=odps
spark.hadoop.odps.task.major.version = cupid_v2
spark.hadoop.odps.cupid.container.image.enable = true
spark.hadoop.odps.cupid.container.vm.engine.type = hyper
```

各配置项含义说明：

|配置项|说明|
|---|--|
|`spark.hadoop.odps.project.name`|填写MaxCompute项目名称|
|`spark.hadoop.odps.access.id`|填写MaxCompute用户的accessKeyId|
|`spark.hadoop.odps.access.key`|填写MaxCompute用户的accessKeySecret|
|`spark.hadoop.odps.end.point`|填写项目所在region的MaxCompute Endpoint，如：http://service.cn.maxcompute.aliyun.com/api|
|`spark.hadoop.odps.runtime.end.point`|Spark运行环境endpoint，请配置为所在region的MaxCompute VPC内网Endpoint，如：http://service.cn.maxcompute.aliyun-inc.com/api|
|`spark.sql.catalogImplementation`|固定填 odps|
|`spark.hadoop.odps.task.major.version`|固定填 cupid_v2|
|`spark.hadoop.odps.cupid.container.image.enable`|固定填 true|
|`spark.hadoop.odps.cupid.container.vm.engine.type`|固定填 hyper|

## 主要模块以及Maven依赖 {#section_hzt_lgg_4gb .section}

项目中Spark相关的依赖模块主要有三个：

-   cupid-sdk**\[开源应用接入MaxCompute SDK\]**
-   odps-spark-datasource\_2.10**\[Spark-1.x MaxCompute数据访问API\]**
-   odps-spark-datasource\_2.11**\[Spark-2.x MaxCompute数据访问API\]**

```language-xml
# scope请设置为provided
# 另外如依赖spark-core spark-sql等模块，也请统一设置为provided
<dependency>
	<groupId>com.aliyun.odps</groupId>
	<artifactId>cupid-sdk</artifactId>
	<version>3.3.3-public</version>
	<scope>provided</scope>
</dependency>
# Spark-1.x请依赖此模块
<dependency>
	<groupId>com.aliyun.odps</groupId>
	<artifactId>odps-spark-datasource_2.10</artifactId>
	<version>3.3.3-public</version>
</dependency>
# Spark-2.x请依赖此模块
<dependency>
  	<groupId>com.aliyun.odps</groupId>
  	<artifactId>odps-spark-datasource_2.11</artifactId>
  	<version>3.3.3-public</version>
</dependency>
```

