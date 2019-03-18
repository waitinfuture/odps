# Import data {#concept_amj_mjy_5db .concept}

This topic describes how to import data to MaxCompute by running a tunnel command.

MaxCompute provides [different methods for importing and exporting data](../../../../../reseller.en-US/User Guide/Data upload and download/Data upload and download.md#). For more information about how to import data by running a tunnel command on the MaxCompute client, see [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).

## Run a tunnel command to import data {#section_v5p_zjy_5db .section}

1.  **Prepare the data.**

    Download the [bank.txt](https://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/cn/shujia/0.2.00/assets/pic/data-develop/banking.txt) file for this example and save it to D:\\. This file contains the information such as the age, profession, and highest level of educational attainment of different people. The following is an example:

    ```
    44,blue-collar,married,basic.4y,unknown,yes,no,cellular,aug,thu,210,1,999,0,nonexistent,1.4,93.444,-36.1,4.963,5228.1,0
    53,technician,married,unknown,no,no,no,cellular,nov,fri,138,1,999,0,nonexistent,-0.1,93.2,-42,4.021,5195.8,0
    28,management,single,university.degree,no,yes,no,cellular,jun,thu,339,3,6,2,success,-1.7,94.055,-39.8,0.729,4991.6,1
    ```

2.  **Create a MaxCompute table.**

    Create a MaxCompute table to store the preceding data. If you have created a table according to [Create and view a table](reseller.en-US/Quick Start/Create and view a table.md#), skip this step. In this example, the table is named bank\_data.

3.  **Run a tunnel command.**

    Run the following tunnel command to import data to the MaxCompute client:

    ```
    tunnel upload D:\banking.txt bank_data;
    ```

    If **OK** \(outlined in red in the following figure\) is displayed, the data is uploaded.![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/155287075736997_en-US.png)

4.  **Verify the result.**

    Use `select count(*) from bank_data;` to check whether all the records in the bank\_data table are successfully uploaded. In this example, the table has 41,188 records in total.![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11951/15528707571484_en-US.png)

    **Note:** 

    -   For more information about tunnel commands, for example, how to import data into a partitioned table, see [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md).

## Other import methods {#section_vcl_r5y_5db .section}

You can also import data to MaxCompute by using [MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Manage data and resources/Import and export data.md#), [Tunnel SDK](../../../../../reseller.en-US/SDK Reference /Java SDK.md#), [data integration](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#), or open-source applications such as Sqoop, Fluentd, Flume, and Logstash. For more information, see [Data upload and download tools](../../../../../reseller.en-US/User Guide/Data upload and download/Data upload and download tools.md).

## What to do next {#section_t1h_c34_jgb .section}

After importing data to MaxCompute, you can run SQL statements to process the data in MaxCompute. For more information, see [Run SQL statements and export data](reseller.en-US/Quick Start/Run SQL statements and export data.md#).

