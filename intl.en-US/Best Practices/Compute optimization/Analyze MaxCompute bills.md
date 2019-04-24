# Analyze MaxCompute bills {#concept_zyb_qvp_pgb .concept}

This topic describes how to analyze MaxCompute bills.

## Scenario {#section_jtx_vwq_pgb .section}

Alibaba Cloud MaxCompute is a big data analysis platform. You can select Pay-As-You-Go or Subscription when purchasing the computing resources. The computing resources are charged by project on a daily basis, and the bills are generated before 06:00 the next day.

Generally, bill fluctuations are released during data development or on the eve of a product launch. You can download the bill details from Alibaba Cloud Expenses center and optimize your tasks according to the fluctuation information provided.

## Obtain the bill details {#section_rjq_jcr_pgb .section}

-   Log on to the [Alibaba Cloud console](https://home.console.aliyun.com/new?spm=5176.8142029.388261.3.5fc16d3eb27Z8R#/).
-   Choose **Billing Management** \> **Billing Management**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438292_en-US.png)

-   Choose **Purchases record** \> **Purchases details**, and select the target product and billing date, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438293_en-US.png)

    **Note:** The **post-payment** method of the Subscription mode is used to pay storage and download fees. This is the only payment method for which storage and download fees are available.

-   Choose **Purchases record** \> **Usage record**, select the target product, service period, and measurement granularity, and enter the verification code. Then, export the CSV file, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438295_en-US.png)

    Analyze the CSV file \(as shown in the following figure\) on your local server.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438298_en-US.png)

    ```
    --Table header of the CSV file.
    ProjectId, MeteringId, Type, Storage, EndTime, ComputationSqlInput, ComputationSqlComplexity, UploadEx, DownloadEx, CUUsage, InputOTS, InputOSS, StartTime, OdpsSpecCode, DataWorks Scheduling Task ID
    ```


## Upload the bill details to MaxCompute {#section_ysm_kcr_pgb .section}

The following information describes the parameters used in the CSV table header:

-   ProjectId: The MaxCompute project list of your Alibaba Cloud account.
-   MeteringId: Includes the IDs of the storage, computing, upload, and download tasks. The ID of a SQL task is instanceid, and the ID of upload and download tasks is Tunnel sessionid.
-   Type: Includes Storage \(storage data\), ComputationSql \(computing data\), UploadIn \(data uploaded through the intranet\), UploadEx \(data uploaded through the Internet\), DownloadIn \(data downloaded through the intranet\), and DownloadEx \(data downloaded through the Internet\). According to the billing rules, only the items that are outlined in red in the preceding figure are charged.
-   StartTime/EndTime: The time from which a task begins to run, and the time at which a task stops running. Only the storage data is obtained by hour.
-   Storage: The storage data that is read on an hourly basis. Unit: byte.
-   ComputationSqlInput: The volume of input data that is computed by running a SQL statement. Unit: byte.
-   ComputationSqlComplexity: The SQL complexity. It is one of the SQL billing factors.
-   UploadEx/DownloadEx: The volume of data that is uploaded or downloaded through the Internet. Unit: byte.
-   CoreSecond: The unit of MapReduce task computing time. You need to convert this unit to hour.
-   InputOTS/InputOSS: The volume of data that is read after external tables begin to incur charges. Unit: byte.

To upload the bill details to MaxCompute, follow these steps:

1.  Verify that the data and column delimiters in the CSV file are correct. \(We recommend that you use UE to view the CSV file.\)

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438301_en-US.png)

    **Note:** The data in the file is separated by commas \(,\), and the cell values are enclosed in double quotation marks \(""\).

2.  Replace all the double quotation marks \(""\) with null.
3.  Create a MaxCompute table to store the bill details.

    ```
    DROP TABLE IF EXISTS maxcomputefee ;
    
    CREATE TABLE IF NOT EXISTS maxcomputefee 
    (
        projectid STRING COMMENT 'ProjectId'
        ,feeid STRING COMMENT 'MeteringId'
        ,type STRING COMMENT 'Type, including Storage, ComputationSQL, and DownloadEx'
        ,storage BIGINT COMMENT 'Storage'
        ,endtime DATETIME COMMENT 'EndTime'
        ,computationsqlinput BIGINT COMMENT 'ComputationSqlInput'
        ,computationsqlcomplexity DOUBLE COMMENT 'ComputationSqlComplexity'
        ,uploadex BIGINT COMMENT 'UploadEx'
        ,download BIGINT COMMENT 'DownloadEx'
        ,cu_usage DOUBLE COMMENT 'MR Compute Hour (second)'
        ,input_ots BIGINT COMMENT 'InputOTS'
        ,input_oss BIGINT COMMENT 'InputOSS'
        ,starttime DATETIME COMMENT 'StartTime'
        ,source_type String COMMENT 'OdpsSpecCode'
        ,source_id String COMMENT 'DataWorks Scheduling Task ID'
    );
    					
    ```

4.  Run the tunnel command to upload data. For more information about tunnel configurations, see [Tunnel commands](../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md#).

    ```
    odps@ sz_mc>tunnel upload /Users/yangyi/Desktop/ODPS_2019-01-12_2019-01-14.csv maxcomputefee -c "UTF-8" -h "true" -dfp "yyyy-MM-dd HH:mm:ss";
    ```

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955438310_en-US.png)

    **Note:** You can also upload data by using the data import feature of DataWorks. For more information, see [Create a table and upload data](../../../../reseller.en-US/Quick Start/Step 1: Create a table and upload data.md#).

5.  View the data.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955538312_en-US.png)


## Use SQL to analyze the bill data {#section_m5f_t5r_pgb .section}

1.  Analyze the SQL fees.

    **Note:** Approximately 95% users can meet their service requirements by using SQL. The incurred fees also occupy a large proportion of the MaxCompute bills.

    ```
    SQL fee = One-time SQL computing fee = Input computing data × SQL complexity × USD 0.0447/GB
    ```

    ```
    --Analyze the SQL fees. The SQL fees are ranked by sqlmoney.
    SELECT  to_char(endtime,'yyyymmdd') as ds,feeid as instanceid
            ,projectid
            ,computationsqlcomplexity  --Complexity
            ,SUM((computationsqlinput / 1024 / 1024 / 1024)) as computationsqlinput  --Input data (GB)
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

    -   View the result, as shown in the following figure.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955538316_en-US.png)

        The preceding figure shows that:

        1.  You can save costs by reducing the input data and complexity of **large-scale SQL tasks**.
        2.  You can summarize the data based on the ds field \(on a daily basis\) and analyze the SQL fee trend during a specified period of time. For example, you can draw a line chart in an Excel file or by using tools such as Quick BI to display the data.
        3.  After obtaining the value of the instanceid field, you can run the `wait instanceid;` command in the console or in DataWorks to view the target task and SQL statements.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955538317_en-US.png)

            Open a browser and enter the logview URL \(for more information about the logview, see [Logview](../../../../reseller.en-US/User Guide/View Job Running Information/Logview.md#)\), as shown in the following figure.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955538318_en-US.png)

            Obtain the DataWorks node name from the logview.

            Open the SourceXML in the logview to view the task running details. As shown in the following figure, SKYNET\_NODENAME \(which is underlined in red\) is the name of a DataWorks node. Only tasks that are executed by the scheduling system have a value. With the node name \(SKYNET\_NODENAME\), you can locate the target node, optimize the node, and view the node owner.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955738319_en-US.png)

2.  Analyze the task increasing trend.

    **Note:** A surge in the number of tasks due to repeated running or unreasonable configurations to scheduling attributes may result in increased fees.

    ```
    --Analyze the task increasing trend.
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

    -   Result

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955838322_en-US.png)

        The preceding figure shows the fluctuation of tasks that are submitted to MaxCompute and are successfully executed from January 12, 2019 to January 14, 2019.

3.  Analyze the storage fees.

    ```
    --Analyze the storage fees.
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

    -   Result

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955838323_en-US.png)

        The preceding figure shows that:

        1.  The storage fees increased on January 13, 2019 but decreased on January 14, 2019
        2.  You can optimize the storage tasks by setting the table lifecycle and deleting unnecessary temporary tables.
4.  Analyze the download fees.

    MaxCompute charges fees for data that is downloaded from the Internet or across regions according to the downloaded data volume.

    ```
    One-time download fee = Downloaded data volume × USD 0.192/GB
    ```

    ```
    --Analyze the download fees.
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

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/122299/155609955838324_en-US.png)

5.  Analyze MapReduce task fees.

    **Note:** 

    ```
    MapReduce task fee = Total computing time × USD 0.06854
    ```

    ```
    --Analyze MapReduce task fees.
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

6.  Analyze external table \(OTS and OSS\) task fees.

    **Note:** The fees of external SQL tables are charged as follows:

    ```
    SQL computing fee = Input data × SQL complexity × USD 0.00447/GB
    ```

    ```
    --Analyze SQL task fees of an OTS table.
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
    
    --Analyze SQL task fees of an OSS table.
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


The surge in the number of tasks may result in the increase of MaxCompute fees. To save costs, we recommend that you optimize your tasks such as SQL tasks in advance.

