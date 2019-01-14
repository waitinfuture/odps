# Announcement {#concept_p4b_ndc_5db .concept}

This article provides you with updated records of the use of MaxCompute.

## January 15th,2019 HongKong Region Underlying Structure Optimization {#section_kmt_djk_lgb .section}

MaxCompute will optimize the underlying Meta Data Warehouse structure during **16:00 - 20:00 January 15th 2019** to improve the stability and performance. During the release window , **HongKong Region** users could encounter task submission delay about **one minute** or task failure. Under very extremely scenario , application would be **not available for 30 minutes**. Please avoid to submit task during the release window. There is NO affect to other regions. If you have any concerns, please contact us by DingTalk or ticket .

## Dec 24 ,2018 MaxCompute Supports Timezone Configuration {#section_wjq_mrc_ggb .section}

The default timezone of MaxCompute is UTC+8. It computes base on UTC+8 time when you use datetime/timestamp/date data types or time related built-in functions. Dec 24 ,2018 , MaxCompute is starting to support timezone configuration functionality. There are two ways to configure timezone in MaxCompute:

-   Session level: `set odps.sql.timezone=<timezoneid>;`This statement needs to be submitted with the calculation statement, such:

    ```
    set odps.sql.timezone=Asia/Tokyo;
    select getdate();
    Results:
    output:
    +------------+
    | _c0        |
    +------------+
    | 2018-10-30 23:49:50 |
    +------------+
    ```

-   Project level: `setProject odps.sql.timezone=<timezoneid>;`This query has to be executed with command line by Project owner. Once timezone configuration submitted in Project level, all time related computing would use new timezone immediately and there would be affection to data of existing tasks. We suggest to configure timezone for new projects rather than existing projects.

**Note:**

-   Timezone configuration supports:
    1.  SQL built-in date functions GETDATE, TO\_DATE, TO\_CHAR
    2.  The conversion between datetime and string.
-   Timezone configuration does not support:
    1.  All other tasks except SQL, like MR.
    2.  UDF, UDJ, UDT, select transform function in SQL.
-   Timezone configuration format is like Asia/Shanghai \(daylight saving is considered \) rather then GMT+9.
-   When there is difference between SDK and project timezone , you need to configure GMT timezone for executing \`date to string\` command.
-   After timezone configuration, there would be discrepancy between real time and output time. The difference is 5 minutes 52 seconds when real time is between 1900 and 1928. The difference is 9 seconds when real time is before 1928.
-   MaxCompute service, Java SDK and related console will release new version \(Java SDK/Console version with postfix `-oversea` \) in near future to ensure the accuracy of MaxCompute datetime type data in multiple regions.The display of existing MaxCompute datatime data which is older than 1928 will be affected in new version.
-   Please also upgrade Java SDK/Console\(if the local timezone is not UTC+8\) when you do MaxCompute upgrading, to ensure the accuracy and consistence between SQL computing result and tunnel transforming data after \`1900-01-01\`. For the datetime data before \`1900-01-01\`, there would be 343 seconds difference between SQL computing output and tunnel transforming data. For the existing datetime data before \`1928-01-01\`, the output time will reduce 352 seconds in new version.
-   There would be data discrepancy risk between SQL output and tunnel transforming data if SDK/Console version without postfix of `-oversea` used continually. The time difference is 9 seconds if real time is before \`1900-01-01\` .The time difference is 352 seconds if real time is in the range of 1900-01-01 ~ 1928-01-01.

    **Note:** 

    The time zone update or configuration of Java SDK/Console version does not affect the time zone configuration of DataWorks, so there will be differences in time zone. You need to evaluate the impact of timed task scheduling in DataWorks. In Japan Region, DataWorks server is GMT+9, in Singapore Region, DataWorks server is GMT+8.

-   When using third-party clients connected by JDBC, you need to set time zones on the client side to ensure consistency with time zones on the server side.

