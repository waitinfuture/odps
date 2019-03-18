# Run SQL statements and export data {#concept_tgf_v5y_5db .concept}

This topic describes how to run SQL statements to export data.

After importing data to MaxCompute, you can process the data by running SQL statements on the [MaxCompute client](../../../../../reseller.en-US/Prepare/Install and configure a client.md#) or in [DataWorks](../../../../../reseller.en-US/User Guide/Data development/Node type/ODPS SQL node.md#).

Currently, MaxCompute SQL supports the use of:

-   A variety of operators.
-   DDL statements to manage tables, partitions, and views.
-   SELECT statements to query records in tables and WHERE statements to filter records in tables.
-   INSERT statements to insert and update data.
-   JOIN operations to associate two tables and MAPJOIN operations for multiple small tables.
-   Built-in and user-defined functions for computing.
-   Regular expressions.

**Note:** 

-   MaxCompute SQL does not support transactions, indexes, update operations, or delete operations. The SQL syntax of MaxCompute is different from that of Oracle and MySQL. Therefore, you cannot migrate SQL statements from other databases to MaxCompute.
-   For more information about SQL operations, see [SQL summary](../../../../../reseller.en-US/User Guide/SQL/SQL summary.md#).
-   After you submit a MaxCompute job, some time is required to schedule the job. Therefore, MaxCompute is suitable for processing jobs in batches, that is, processing a massive volume of data. MaxCompute is not suitable for frontend business systems that must process several thousand or tens of thousands of transactions per second. For more information about how to optimize a job, see [SQL optimization](../../../../../reseller.en-US/Best Practices/Compute optimization/SQL optimization.md#).
-   For the limits of MaxCompute SQL, see [SQL limits](../../../../../reseller.en-US/User Guide/SQL/SQL limits.md#).

## Obtain and analyze data {#section_ynz_3mq_kgb .section}

The following is an example of using SQL to query the number of single people \(with different education backgrounds\) that have purchased a house. The result is saved to the result\_table table for analysis and display.

```language-sql
INSERT OVERWRITE TABLE result_table  --Insert data to the result_table table.
SELECT education,COUNT(marital) AS num
FROM bank_data
WHERE housing = 'yes'
    AND marital = 'single'
GROUP BY education;
```

You can use `select * from result_table;` to view data in the result\_table table, as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11952/155287078437072_en-US.png)

You can process multiple tables by using multiple SQL statements in DataWorks. For more information, see [Business flow](../../../../../reseller.en-US/User Guide/Data development/Business flow/Business flow.md#).

## Export data {#section_jvd_2j4_jgb .section}

After processing data by using SQL statements, you can export data to your D drive by running a tunnel command. For more information, see [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md#).

```
tunnel download result_table D:\result.txt;
```

If **download OK** \(outlined in red in the following figure\) is displayed, the data is successfully exported.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11952/155287078437074_en-US.png)

**Note:** You can use the [data integration](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#) feature to export data to other data sources \(such as MySQL\).

