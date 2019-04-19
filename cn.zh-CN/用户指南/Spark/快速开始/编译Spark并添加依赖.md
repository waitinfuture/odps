# 编译Spark并添加依赖 {#concept_xqm_wlf_4gb .concept}

环境配置完成后，您需要配置Spark-defaults.conf文件，编译Spark，并添加Maven依赖。

## 配置Spark-defaults.conf {#section_wsd_j4f_4gb .section}

在**$SPARK\_HOME/conf**路径下存在spark-defaults.conf.template文件，这个可以作为spark-defaults.conf的模版，需要在该文件中设置MaxCompute相关的账号信息后，才可以提交Spark任务到MaxCompute。默认配置内容如下，将空白部分根据实际的账号信息填上即可，其余的配置可以保持不变。

```language-java
# OdpsAccount Info Setting
spark.hadoop.odps.project.name =
spark.hadoop.odps.access.id =
spark.hadoop.odps.access.key =
spark.hadoop.odps.end.point = http://service.cn.maxcompute.aliyun.com/api
```

各配置项含义说明：

|配置项|说明|
|---|--|
|`spark.hadoop.odps.project.name`|填写MaxCompute项目名称|
|`spark.hadoop.odps.access.id`|填写MaxCompute用户的accessKeyId|
|`spark.hadoop.odps.access.key`|填写MaxCompute用户的accessKeySecret|
|`spark.hadoop.odps.end.point`|填写项目所在region的MaxCompute Endpoint，如：http://service.cn.maxcompute.aliyun.com/api|

## Windows环境下编译Spark {#section_j0i_dqd_rxo .section}

编译环境要求：

1.  Java SDK 1.8及以上版本
2.  Maven 3.3.0以上版本

操作步骤（以Spark-2.x为例）：

1.  下载aliyun-cupid-sdk代码。对应命令如下。

    ``` {#codeblock_psn_y22_is0}
    git clone https://github.com/aliyun/aliyun-cupid-sdk.git -b 3.3.2-public
    ```

2.  [下载](https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0)Windows下的protoc， 即win32.zip的2.5.0版本。
3.  解压对应的poroc-2.5.0-win32.zip文件，将解压后的potoc.exe文件移动到$\{cupid install path\}\\core\\cupid-sdk\\bin\\protoc目录下。
4.  开始编译aliyun-cupid-sdk。
    -   使用cmd命令行切换到$\{cupid install path\}\\core\\cupid-sdk之后，执行如下命令。

        ``` {#codeblock_auf_iq7_o4f}
         mvn clean install -Pwindows -Ppublic -DskipTests
        ```

    -   切换到$\{cupid install path\}\\spark\\spark-2.x\\datasource之后，执行如下命令。

        ``` {#codeblock_ama_6x0_jyg}
        mvn clean install -Pwindows -Ppublic -DskipTests
        ```

    -   切换到$\{cupid install path\}\\spark\\spark-2.x\\spark-examples之后，执行如下命令。

        ``` {#codeblock_gup_7ax_l4h}
        mvn clean install -Pwindows -Ppublic -DskipTests
        ```

    -   在$\{cupid install path\}\\spark\\spark-2.x\\spark-examples\\target下查看生成的spark-examples\_2.11-3.3.2-public-shaded.jar包。
    -   测试执行Spark Demo, 请参考[Spark-2.x](cn.zh-CN/用户指南/Spark/常见使用场景示例/Spark-2.x.md#)。

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
	<version>3.3.2-public</version>
	<scope>provided</scope>
</dependency>
# Spark-1.x请依赖此模块
<dependency>
	<groupId>com.aliyun.odps</groupId>
	<artifactId>odps-spark-datasource_2.10</artifactId>
	<version>3.3.2-public</version>
</dependency>
# Spark-2.x请依赖此模块
<dependency>
  	<groupId>com.aliyun.odps</groupId>
  	<artifactId>odps-spark-datasource_2.11</artifactId>
  	<version>3.3.2-public</version>
</dependency>
```

