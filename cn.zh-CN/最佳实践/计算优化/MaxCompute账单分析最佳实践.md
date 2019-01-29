# MaxCompute账单分析最佳实践 {#concept_zyb_qvp_pgb .concept}

## 背景信息 {#section_jtx_vwq_pgb .section}

阿里云大数据计算服务MaxCompute是一款商业化的大数据分析平台，其计算资源的计费方式分为预付费和后付费两种，产品每天会以Project为维度进行计量计费（账单会在第二天6点前产出）。

关于MaxCompute计量计费说明，详情请参见[计量计费说明](https://help.aliyun.com/document_detail/27989.html)文档。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238289_zh-CN.png)

通常情况下，我们会在数据开发阶段或者在产品上线前夕发布账单波动（通常情况下为增大）信息。用户可以通过自助的方式来分析账单波动情况，再对自己的作业进行优化。阿里云费用中心就是一个很好的通道，阿里云所有商业化收费的产品都可以在其中下载费用明细。

## 获取账单信息 {#section_rjq_jcr_pgb .section}

通常您需要使用主账号查看账单详情。如果您需要使用子账号查看账单信息，请首先参考[费用中心RAM配置策略](https://help.aliyun.com/document_detail/88883.html)进行子账号授权。

-   步骤1：使用主账号或者被授权的RAM子账号来登录[阿里云管控台](https://home.console.aliyun.com/new?spm=5176.8142029.388261.3.5fc16d3eb27Z8R#/)。
-   步骤2：右上角进入费用中心，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238292_zh-CN.png)

-   步骤3：在费用中心-消费记录-消费明细中，选择产品和账单日期，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238293_zh-CN.png)

    **说明：** 包年包月中的**后付费**是指项目开通包年包月计算计费模式后，产生的存储、下载对应的费用（存储、下载费用只有后付费）。

-   步骤4：为了方便批量分析数据，我们选择下载使用记录csv文件在本地分析，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238295_zh-CN.png)

    下载csv文件如下，可以在本地打开进行分析，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238298_zh-CN.png)

    ```
    --csv的表头
    项目编号，计量信息编号，数据分类，存储（Byte），SQL读取量（Byte），SQL复杂度（Byte），公网上行流量（Byte），公网下行流量（Byte），MR作业计算，开始时间，结束时间，SQL读取量_访问OTS（Byte，,SQL读取量_访问OSS（Byte）
    ```


## 上传账单明细至MaxCompute {#section_ysm_kcr_pgb .section}

使用记录明细字段解释：

-   项目编号：当前账号或子账号对应的主账号的MaxCompute Project列表。
-   计量信息编号：其中会包含存储、计算、上传和下载的计费信息编号，SQL为instanceid，上传和下载为Tunnel sessionid。
-   数据分类：Storage（存储）、ComputationSql（计算）、UploadIn（内网上传）、UploadEx（外网上传）、DownloadIn（内网下载）、DownloadEx（外网下载）。按照计费规则其中只有红色为实际计费项目。
-   开始时间/结束时间：按照实际作业执行时间进行计量，只有Storage是按照每个小时取一次数据。
-   存储（Byte）：每小时读取的存储量单位为Byte。
-   SQL读取量（Byte）：SQL计算项，每一次SQL执行时SQL的input数据量，单位为Byte。
-   SQL复杂度（Byte）：每次执行SQL的复杂度，为SQL计费因子之一。
-   公网上行流量（Byte），公网下行流量（Byte）：分别为公网上传和下载的数据量，单位Byte。
-   MapRrduce作业计算（CoreSecond）：MR作业的计算时单位为coresecond，需要转换为计算时hour。
-   SQL读取量\_访问OTS（Byte），SQL读取量\_访问OSS（Byte）：外部表实施收费后的读取数据量，单位Byte。

详细的实施步骤为：

1.  确认CSV文件数据，尤其是列分隔符等（推荐使用UE）。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238301_zh-CN.png)

    **说明：** 数据以逗号分隔，且单元格值都带有双引号。

2.  数据预处理：替换掉文档所有双引号，以方便使用Tunnel等工具。

    **说明：** 替换为不用填写。直接点击**全部替换**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947238304_zh-CN.png)

3.  创建MaxCompute表，存储下载的消费明细。

    ```
    DROP TABLE IF EXISTS maxcomputefee ;
    
    CREATE TABLE IF NOT EXISTS maxcomputefee 
    (
        projectid STRING COMMENT '项目编号'
        ,feeid STRING COMMENT '计费信息编号'
        ,type STRING COMMENT '数据分类，包括Storage、ComputationSQL、DownloadEx等'
        ,starttime DATETIME COMMENT '开始时间'
        ,storage BIGINT COMMENT '存储量'
        ,endtime DATETIME COMMENT '结束时间'
        ,computationsqlinput BIGINT COMMENT '输入数据量'
        ,computationsqlcomplexity DOUBLE COMMENT 'sql复杂度'
        ,uploadex BIGINT COMMENT '公网上行流量Byte'
        ,download BIGINT COMMENT '公网下行流量Byte'
        ,cu_usage DOUBLE COMMENT 'MR计算时*second'
        ,input_ots BIGINT COMMENT '访问OTS的数据输入量'
        ,input_oss BIGINT COMMENT'访问OSS的数据输入量'
    )
    ;
    ```

4.  Tunnel上传数据，具体Tunnel的配置详见[Tunnel命令操作](../../../../../cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md#)。

    ```
    odps@ sz_mc>tunnel upload /Users/yangyi/Desktop/ODPS_2019-01-12_2019-01-14.csv maxcomputefee -c "UTF-8" -h "true" -dfp "yyyy-MM-dd HH:mm:ss";
    ```

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338310_zh-CN.png)

    **说明：** 用户也可以通过DataWorks数据导入的功能来进行，具体请参见[操作步骤](../../../../../cn.zh-CN/快速开始/步骤一：建表并上传数据.md#)。

5.  验证数据。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338312_zh-CN.png)


## 通过SQL分析账单数据 {#section_m5f_t5r_pgb .section}

1.  分析SQL费用

    **说明：** 云上客户使用MaxCompute，95%的用户通过SQL即可满足需求，SQL也在消费成长中占据了绝大部分。

    ```
    SQL费用=一次SQL计算费用 = 计算输入数据量 SQL复杂度0.3元/GB
    ```

    ```
    --分析SQL消费，按照SQL进行排行
    SELECT  to_char(endtime,'yyyymmdd') as ds,feeid as instanceid
            ,projectid
            ,computationsqlcomplexity  --复杂度
            ,SUM((computationsqlinput / 1024 / 1024 / 1024)) as computationsqlinput  --数据输入量GB
            ,SUM((computationsqlinput / 1024 / 1024 / 1024)) * computationsqlcomplexity * 0.3 AS sqlmoney
    FROM    maxcomputefee
    WHERE   TYPE = 'ComputationSql'
    AND to_char(endtime,'yyyymmdd') >= '20190112'
    GROUP BY to_char(endtime,'yyyymmdd'),feeid
             ,projectid
             ,computationsqlcomplexity
    ORDER BY sqlmoney DESC 
    LIMIT   10000
    ;
    ```

    -   查询结果

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338316_zh-CN.png)

        根据此段SQL执行结果可以得到如下结论：

        1.  **大作业**可以优化的点：\*\*是否可以减小数据读取量、降低复杂度来优化费用成本。
        2.  也可以按照ds字段（按照天）进行汇总，分析某个时间段内的SQL消费金额走势。比如利用本地excel或云上QuickBI等工具绘制折线图等方式，更直观的反应作业的趋势。
        3.  拿到具体的instanceid，在console或者DataWorks脚本执行`wait instanceid;`命令查看具体作业和SQL。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338317_zh-CN.png)

            在浏览器中打开logview的url地址（关于logview的介绍详见[Logview](../../../../../cn.zh-CN/用户指南/Job运行信息查看/Logview.md#)），如下图。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338318_zh-CN.png)

            从logview中获取DataWorks节点名称：

            在logview中打开SourceXML可以查看到具体执行信息，如SKYNET\_NODENAME表示DataWorks的节点名称（当然只有被调度系统执行的作业才有值，临时查询为空，如下图所示）。拿到节点名称可以快速的在DataWorks找到该节点进行优化或查看责任人，如下图。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338319_zh-CN.png)

2.  分析作业增长趋势

    **说明：** 一般情况下费用的增长背后其实是作业量的暴涨，可能是重复执行，也可能是调度属性配置的不合理。

    ```
    --分析作业增长趋势
    SELECT  TO_CHAR(endtime,'yyyymmdd') AS ds
            ,projectid
            ,COUNT(*) AS tasknum
    FROM    maxcomputefee
    WHERE   TYPE = 'ComputationSql'
    AND     TO_CHAR(endtime,'yyyymmdd') >= '20190112'
    GROUP BY TO_CHAR(endtime,'yyyymmdd')
             ,projectid
    ORDER BY tasknum DESC
    LIMIT   10000
    ;
    ```

    -   执行结果

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338322_zh-CN.png)

        从执行结果可以看出来12-14日提交到MaxCompute且执行成功的作业数的波动趋势。

3.  分析存储费用

    **说明：** 存储费用的计费规则相对来说比较复杂，因为下载到的明细是每个小时取一次数据。按照MaxCompute存储计费规则，会整体24小时求和然后平均之后的值再阶梯收费。具体详见[计量计费说明](../../../../../cn.zh-CN/产品定价/计量计费说明.md#)。

    ```
    --分析存储费用
    SELECT  t.ds
            ,t.projectid
            ,t.storage
            ,CASE    WHEN t.storage < 0.5 THEN 0.01
                     WHEN t.storage >= 0.5 AND t.storage <= 100 THEN t.storage*0.0192
                     WHEN t.storage > 100 AND t.storage <= 1024 THEN (100*0.0192+(t.storage-100)*0.0096)
                     WHEN t.storage > 1024 AND t.storage <= 10240 THEN (100*0.0192+(1024-100)*0.0096+(t.storage-1024)*0.0084)
                     WHEN t.storage > 10240 AND t.storage <= 102400 THEN (100*0.0192+(1024-100)*0.0096+(10240-1024)*0.0084+(t.storage-10240)*0.0072)
                     WHEN t.storage > 102400 AND t.storage <= 1048576 THEN (100*0.0192+(1024-100)*0.0096+(10240-1024)*0.0084+(102400-10240)*0.0072+(t.storage-102400)*0.006) 
             END storage_fee
    FROM    (
                SELECT  to_char(starttime,'yyyymmdd') as ds
                        ,projectid
                        ,SUM(storage/1024/1024/1024)/24 AS storage
                FROM    maxcomputefee
                WHERE   TYPE = 'Storage'
                and to_char(starttime,'yyyymmdd') >= '20190112'
                GROUP BY to_char(starttime,'yyyymmdd')
                         ,projectid
            ) t
    ORDER BY storage_fee DESC
    ;
    ```

    -   执行结果

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338323_zh-CN.png)

        根据计算结果可以分析得出结论：

        1.  存储在13日为最高有一个增长的过程，但是在14日是有降低。
        2.  存储优化，建议表设置生命周期，删除长期不使用的临时表等。
4.  分析下载费用

    对于公网或者跨Region的数据下载，MaxCompute将按照下载的数据大小进行计费。计费公式为：

    ```
    一次下载费用=下载数据量*0.8元/GB
    ```

    ```
    --分析下载消费明细
    SELECT  TO_CHAR(starttime,'yyyymmdd') AS ds
            ,projectid
            ,SUM((download/1024/1024/1024)*0.8) AS download_fee
    FROM    maxcomputefee
    WHERE   type = 'DownloadEx'
    AND     TO_CHAR(starttime,'yyyymmdd') >= '20190112'
    GROUP BY TO_CHAR(starttime,'yyyymmdd')
             ,projectid
    ORDER BY download_fee DESC
    ;
    ```

    按照执行结果也可以分析出某个时间段内的下载费用走势。另外可以通过tunnel show history查看具体历史信息，具体命令详见[Tunnel命令操作](../../../../../cn.zh-CN/用户指南/数据上传下载/Tunnel命令操作.md#)。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/154876947338324_zh-CN.png)

    以下几种计算作业与SQL类似，按照[计量计费说明](../../../../../cn.zh-CN/产品定价/计量计费说明.md#)编写SQL即可。

5.  分析MapReduce作业消费

    **说明：** 

    ```
    MR任务当日计算费用=当日总计算时*0.46元
    ```

    ```
    --分析MR作业消费
    SELECT  TO_CHAR(starttime,'yyyymmdd') AS ds
            ,projectid
            ,(cu_usage/3600)*0.46 AS mr_fee
    FROM    maxcomputefee
    WHERE   type = 'MapReduce'
    AND     TO_CHAR(starttime,'yyyymmdd') >= '20190112'
    GROUP BY TO_CHAR(starttime,'yyyymmdd')
             ,projectid
             ,cu_usage
    ORDER BY mr_fee DESC
    ;
    ```

6.  分析外部表作业（OTS和OSS）

    **说明：** SQL外部表功能计费规则：

    ```
    一次SQL计算费用=计算输入数据量SQL复杂度0.03元/GB
    ```

    ```
    --分析OTS外部表SQL作业消费
    SELECT  TO_CHAR(starttime,'yyyymmdd') AS ds
            ,projectid
            ,(input_ots/1024/1024/1024)*1*0.03 AS ots_fee
    FROM    maxcomputefee
    WHERE   type = 'ComputationSql'
    AND     TO_CHAR(starttime,'yyyymmdd') >= '20190112'
    GROUP BY TO_CHAR(starttime,'yyyymmdd')
             ,projectid
             ,input_ots
    ORDER BY ots_fee DESC
    ;
    
    --分析OSS外部表SQL作业消费
    SELECT  TO_CHAR(starttime,'yyyymmdd') AS ds
            ,projectid
            ,(input_oss/1024/1024/1024)*1*0.03 AS ots_fee
    FROM    maxcomputefee
    WHERE   type = 'ComputationSql'
    AND     TO_CHAR(starttime,'yyyymmdd') >= '20190112'
    GROUP BY TO_CHAR(starttime,'yyyymmdd')
             ,projectid
             ,input_oss
    ORDER BY ots_fee DESC
    ;
    ```


MaxCompute产品消费的增长（暴涨）大多由于作业量的大幅度提升导致，要优化自己的费用成本，首先要检查自己SQL等作业中存在的问题，要优化具体哪一个SQL，希望本文能够给予您一些帮助。

## 更多信息 {#section_fhf_phs_pgb .section}

如果您想了解更多关于费用成本优化的文章，请参见云栖社区[帮助企业做好MaxCompute大数据平台成本优化的最佳实践](https://yq.aliyun.com/articles/257512?spm=a2c4e.11155435.0.0.ca4d7622X3jWhn)。

