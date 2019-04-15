# Table operations {#concept_l3j_w31_wdb .concept}

This topic describes how to create, view, delete, rename, and modify table information through the client.

## Create tables {#section_qqp_st1_wdb .section}

Statement format:

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[CLUSTERED BY (col_name [, col_name, ...]) [SORTED BY (col_name [ASC | DESC] [, col_name [ASC | DESC] ...])] INTO number_of_buckets BUCKETS] - Sets the Shuffle and Sort attributes when you create hash cluster tables.
[STORED BY StorageHandler] - Used only for external tables.
[WITH SERDEPROPERTIES (Options)] - Used only for external tables.
[LOCATION OSSLocation];- Used only for external tables.
[LIFECYCLE days]
[AS select_statement]
 CREATE TABLE [IF NOT EXISTS] table_name
 LIKE existing_table_name
```

Consider the following points:

-   When a table is created, an error is returned if another table has the same name, but the if not exists option is not specified. If the option is specified, no matter whether another table that has the same name exists or even if the source table structure and the target table structure are inconsistent, a success message is returned. The meta information of the existing table that has the same name remains unchanged.
-   Both the table name and column name are not case sensitive. They can contain letters, numbers, and underscores \(\_\), but must begin with a letter. Neither of them can exceed 128 bytes in length.
-   Up to 1,200 column definitions are allowed in a table.
-   Supported [data types](../../../../../reseller.en-US/User Guide/Definitions/Data types.md) include bigint, double, boolean, datetime, decimal, and string.

    **Note:** MaxCompute SQL statements and the latest version of MapReduce support session-level table operations. If data types such as tinyint, smallint, int, float, varchar and TIMESTAMP BINARY are involved when an SQL statement is executed, `set odps.sql.type.system.odps2=true;` must be added preceding the SQL statement. The SET statement and SQL statement are submitted simultaneously.

-   Use Partitioned by to specify the [partition](../../../../../reseller.en-US/User Guide/Definitions/Partition.md) field, which supports the following data types: tinyint, smallint, int, bigint, varchar, and string.

    The value of partition cannot have a double-byte character and must begin with a letter, followed by a letter or number. The name cannot exceed 128 bytes in length. Special characters can be used, including spaces, colons \(:\), underscores \(\_\), dollar signs \($\), pound signs \(\#\), periods \(.\), exclamation points \(!\), and at signs \(@\). Other characters, such as \(\\t\), \(\\n\), and \(/\) are considered undefined characters. If the partition field is used to partition a table, a full scan is not needed when you add a partition or update or read data in a partition. This makes table processing more efficient.

-   Up to 60,000 partitions for up to six levels are allowed in a table.
-   The content of a comment is a string whose length does not exceed 1024 bytes.
-   lifecycle indicates the lifecycle of a table, the unit is 'days.' The statement create table like does not copy the lifecycle attribute from source table
-   `CLUSTERED BY` is used to specify hash keys. MaxCompute executes hash computation formulas on specified columns, and places computation results into buckets based the specified hash keys.
    -   To prevent data skew and hotspots and better execute parallel statements, we recommend that you specify columns in `CLUSTERED BY` in the case that the value range is large and a small number of duplicate key values exist. Also, to better execute `JOIN` statements, we recommend that you select commonly used join or aggregation keys. Join and aggregation keys are similar to primary keys in conventional databases.
    -   `SORTED BY` specifies how to sort fields in a bucket. We recommend that you keep the value of `SORTED BY` consistent with that of `CLUSTERED BY` to make performance higher. After fields are specified in the `SORTED BY` clause, MaxCompute automatically generates indexes. It also executes the clause faster based on the indexes when you query data.
    -   INTO number\_of\_buckets BUCKETS specifies the number of hash buckets. The value of this field varies depending on the data size. By default, MaxCompute supports up to 1,111 reducers. Therefore, up to 1,111 hash buckets are allowed. You can run the set odps.sql.reducer.instances=xxx; command to increase the maximum value of this field. However, the maximum value cannot exceed 4,000. If the maximum value exceeds 4,000, performance deteriorates as a result.

        We recommend that you follow the following points when you specify the number of hash buckets:

        -   We recommend that the size of each bucket be around 500 MB. For example, if you want to add 1,000 buckets to a 500-GB partition, the size of each bucket is 500 MB on average. If a table contains a large amount of data, you can increase the size of each bucket from 500 MB to 2 or 3 GB. You can also run the `set odps.sql.reducer.instances=xxx` command to increase the maximum number of hash buckets that are allowed.
        -   If two tables are joined frequently, specify the same number of hash buckets for them to better execute JOIN statements and to skip Shuffle and Sort. If the number of hash buckets for one table is different from that for the other table, we recommend that you use the greater number for both tables. This keeps statement concurrency and execution efficiency at proper levels.
    -   Hash cluster tables help optimize the following aspects:
        -   Bucket pruning
        -   Aggregation
        -   Storage
    -   Hash cluster tables are subject to the following limits:
        -   The `insert into` statement is not supported. You can add data only by using the `insert overwrite` statemet.
        -   A tunnel cannot be uploaded to a range cluster table because data uploaded over a tunnel is unordered.
-   For more information about external tables, see [Access OSS](reseller.en-US/User Guide/External table/Access OSS unstructured data.md#).

For example:

Assume that the table sale\_detail is created to store sale records. The table uses sale\_date and region as partition columns. Table creation statements are described as follows:

```
create table if not exists sale_detail(
(
shop_name string,
customer_id string,
total_price double)
)
partitioned by (sale_date string,region string);
 -- Create a partition table sale_detail.
```

The statement `create table...as select ...` can also be used to create a table. After creating a table, the data is copied to the new table, such as:

```
create table sale_detail_ctas1 as
select * from sale_detail;
```

If the table sale\_detail has data, the example mentioned preceding copies all data of sale\_detail into the table sale\_detail\_ctas1.

**Note:** sale\_detail is a partitioned table, while the table created by the statement `create table...as select...` does not copy the partition attribute. The partition column of source table becomes a general column of object table. In other words, sale\_detail\_ctas1 is a non-partition table with 5 columns.

In the statement `create table...as select...` if using a constant as a column value in Select clause, it is suggested specify the column name, such as:

```
create table sale_detail_ctas2 as
        select shop_name,
            customer_id,
            total_price,
            '2013' as sale_date,
            'China' as region
        from sale_detail;
```

If the column name is not specified, the statement is as shown as follows:

```
create table sale_detail_ctas3 as
        select shop_name,
            customer_id,
            total_price,
            '2013',
            'China'
        from sale_detail;
```

Then the forth column and fifth column of the created table sale\_detail\_ctas3 become system generated names, like `_c3`, `_c4`.

To allow the destination table to have the same structure as the source table, try to use `create table … like’` statement, such as:

```
create table sale_detail_like like sale_detail;
```

Now the table structure of `sale_detail_like` is exactly the same as `sale_detail`. Except the life cycle, attributes including the column name, column comment, and table comment, of the two tables are the same. But the data in `sale_detail` cannot be copied into the table `sale_detail_like`.

An example of creating a hash cluster table is as follows:

```
CREATE TABLE T1 (a string, b string, c bigint) CLUSTERED BY (c) SORTED by (c) INTO 1024 BUCKETS; - Creates a hash cluster non-partitioned table with a few clicks.
CREATE TABLE T1 (a string, b string, c bigint) PARTITIONED BY (dt string) CLUSTERED BY (c) SORTED by (c) INTO 1024 BUCKETS; - Creates a hash cluster partitioned table.
```

## View table information {#section_cm5_mv1_wdb .section}

Statement format:

```
desc <table_name>;
desc extended <table_name>; --View external table information.
```

For example:

-   To view the info of the preceding table sale\_detail, run the following statement:

    ```
    desc sale_detail;
    ```

    Return info:

    ```
    odps@ $odps_project>desc sale_detail;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$lili.ll@alibaba-inc.com | Project: $odps_project
                      |
    | TableComment:
       |
    +--------------------------------------------------------------------+
    | CreateTime: 2017-06-28 15:05:17
       |
    | LastDDLTime: 2017-06-28 15:05:17
       |
    | LastModifiedTime: 2017-06-28 15:05:17
       |
    +--------------------------------------------------------------------+
    | InternalTable: YES | Size: 0
       |
    +--------------------------------------------------------------------+
    | Native Columns:
       |
    +--------------------------------------------------------------------+
    | Field | Type | Label | Comment
       |
    +--------------------------------------------------------------------+
    | shop_name | string | |
       |
    | customer_id | string | |
       |
    | total_price | double | |
       |
    +--------------------------------------------------------------------+
    | Partition Columns:
       |
    +--------------------------------------------------------------------+
    | sale_date | string |
       |
    | region | string |
       |
    +--------------------------------------------------------------------+
    OK
    ```

-   To view the information of the preceding table sale\_detail\_like, run the following statement:

    ```
    desc sale_detail_like
    ```

    Return info:

    ```
    odps@ $odps_project>desc sale_detail_like;
    +--------------------------------------------------------------------+
    | Owner: ALIYUN$lili.ll@alibaba-inc.com | Project: $odps_project
                      |
    | TableComment:
       |
    +--------------------------------------------------------------------+
    | CreateTime: 2017-06-28 15:42:17
       |
    | LastDDLTime: 2017-06-28 15:42:17
       |
    | LastModifiedTime: 2017-06-28 15:42:17
       |
    +--------------------------------------------------------------------+
    | InternalTable: YES | Size: 0
       |
    +--------------------------------------------------------------------+
    | Native Columns:
       |
    +--------------------------------------------------------------------+
    | Field | Type | Label | Comment
       |
    +--------------------------------------------------------------------+
    | shop_name | string | |
       |
    | customer_id | string | |
       |
    | total_price | double | |
       |
    +--------------------------------------------------------------------+
    | Partition Columns:
       |
    +--------------------------------------------------------------------+
    | sale_date | string |
       |
    | region | string |
       |
    +--------------------------------------------------------------------+
    OK
    ```


In preceding example, we can see that the attributes of sale\_detail\_like coincide with that of sale\_detail, except for the lifecycle. For more information, see [Describe Table](reseller.en-US/User Guide/Common commands/Table operations.md).

**Note:** The data size you obtain by using the Describe Table command includes the size of data in the recycle bin. If you want to clear the recycle bin, run the purge table table\_name; command. Then, run the Describe Table command, and the returned data size no longer includes the size of data in the recycle bin. To obtain details about data in the recycle bin, run the show recyclebin; command.

Check the information of sale\_detail\_ctas1, you can find that sale\_date and region are only normal columns and not partitions of the table.

-   As more data types are being added to MaxCompute, the types of data returned by the Describe Table command increase. For details, see [Data types](reseller.en-US/User Guide/Definitions/Data types.md#). If new data types are used in MaxCompute, you need to enable flag for the new data types when you execute SQL statements. However, you do not need to do so when you run the Describe Table command.

    **Note:** If the output of SQL statements depends on the input of the Describe Table command, we recommend that you promptly update the settings of flag attributes for new data types in MaxCompute.

    For example:

    ```
    set odps.sql.type.system.odps2=true;
    CREATE TABLE test_newtype (
        c1 tinyint
        ,c2 smallint
        ,c3 int
        ,c4 BIGINT
        ,c5 float
        ,c6 DOUBLE
        ,c7 decimal
        ,c8 binary
        ,c9 timestamp
        ,c10 ARRAY<map<BIGINT,BIGINT>>
        ,c11 map<STRING,ARRAY<BIGINT>>
        ,c12 STRUCT<s1:STRING,s2:BIGINT>
        ,c13 varchar(20))
    LIFECYCLE 1
    ;
    ```

    Information returned by the `desc test_newtype;` command \(some of the information is not presented\):

    ```
    | Native Columns:                                                                    |
    +------------------------------------------------------------------------------------+
    | Field           | Type       | Label | Comment                                     |
    +------------------------------------------------------------------------------------+
    | c1              | tinyint    |       |                                             |
    | c2              | smallint   |       |                                             |
    | c3              | int        |       |                                             |
    | c4              | bigint     |       |                                             |
    | c5              | float      |       |                                             |
    | c6              | double     |       |                                             |
    | c7              | decimal    |       |                                             |
    | c8              | binary     |       |                                             |
    | c9              | timestamp  |       |                                             |
    | c10             | array<map<bigint,bigint>> |       |                              |
    | c11             | map<string,array<bigint>> |       |                              |
    | c12             | struct<s1:string,s2:bigint> |       |                            |
    | c13             | varchar(20) |       |                                            |
    +------------------------------------------------------------------------------------+
    ```


You can run the `DESC EXTENDED table_name;` command to view the Clustering attribute of a hash cluster table. In the following example, the Clustering attribute is displayed in Extended Info.

```

+------------------------------------------------------------------------------------+
| Owner: ALIYUN$xxxxxxx@aliyun.com | Project: xxxxx |
| TableComment: |
+------------------------------------------------------------------------------------+
| CreateTime: 2017-12-25 11:18:26 |
| LastDDLTime: 2017-12-25 11:18:26 |
| LastModifiedTime: 2017-12-25 11:18:26 |
| Lifecycle: 2 |
+------------------------------------------------------------------------------------+
| InternalTable: YES | Size: 0 |
+------------------------------------------------------------------------------------+
| Native Columns: |
+------------------------------------------------------------------------------------+
| Field | Type | Label | Comment |
+------------------------------------------------------------------------------------+
| a | string | | |
| b | string | | |
| c | bigint | | |
+------------------------------------------------------------------------------------+
| Partition Columns: |
+------------------------------------------------------------------------------------+
| dt | string | |
+------------------------------------------------------------------------------------+
| Extended Info: |
+------------------------------------------------------------------------------------+
| TableID: 91a3395d3ef64b4d9ee1d28527552864 |
| IsArchived: false |
| PhysicalSize: 0 |
| FileNum: 0 |
| ClusterType: hash |
| BucketNum: 1024 |
| ClusterColumns: [c] |
| SortColumns: [c ASC] |
+------------------------------------------------------------------------------------+
```

If the table is a partitioned table that has the Clustering attribute, run the following command to view the partition attributes in addition to running the preceding command to view the table attributes:

```
DESC EXTENDED table_name partition(pt_spec);
```

Use the SELECT statement to view data in the table. For details, see [Introduction to the SELECT Syntax](reseller.en-US/User Guide/SQL/Select Operation/Introduction to the SELECT Syntax.md#).

## Drop a table {#section_xvw_1w1_wdb .section}

Statement format:

```
DROP TABLE [IF EXISTS] table_name;
```

**Note:** 

-   If the option \[if exists\] is not specified and the table does not exist, exception returns. If this option is specified, no matter whether the table exists or not, all return success.
-   Data in OSS is not deleted when the external tables are deleted.

For example:

```
create table sale_detail_drop like sale_detail;
    drop table sale_detail_drop;
    --If the table exists, return success; otherwise, return exception.
    drop table if exists sale_detail_drop2;
    --No matter whether the table sale_detail_drop2 exists or not, all return success.
```

## Rename a table {#section_k4t_2w1_wdb .section}

Statement format:

```
ALTER TABLE table_name RENAME TO new_table_name;
```

**Note:** 

-   Rename operation is used to update the table name only and not the data in the table.
-   If the new\_table\_name is duplicated an error may occur.
-   If the table table\_name does not exist, error may occur.

For example:

```
create table sale_detail_rename1 like sale_detail;
alter table sale_detail_rename1 rename to sale_detail_rename2;
```

## Change the owner of a table {#section_sl4_d4y_3hb .section}

MaxCompute allows you to run the changeowner command to change the owner of a table.

Example:

```
alter table table_name changeowner to 'ALIYUN$xxx@aliyun.com';
```

## Alter the comments of a table {#section_qnj_ny1_wdb .section}

Command format:

```
ALTER TABLE table_name SET COMMENT 'tbl comment';
```

**Note:** 

-   The table table\_name must exist.
-   The comment length must not exceed 1024 bytes.

For example:

```
alter table sale_detail set comment 'new comments for table sale_detail';
```

Use the command desc to view the comment modification in the table. For more information, see [Describe Table](reseller.en-US/User Guide/Common commands/Table operations.md) in [Common commands \> Table operations](reseller.en-US/User Guide/Common commands/Table operations.md).

## Alter Table LastDataModifiedTime {#section_gxl_5y1_wdb .section}

MaxCompute SQL supports touch operation to modify LastDataModifiedTime of a table. The result is to modify LastDataModifiedTime of a table to be current time.

Statement format:

```
ALTER TABLE table_name TOUCH;
```

**Note:** 

-   If the table table\_name does not exist, an error is returned.
-   This operation changes the value of LastDataModifiedTime of a table and this is when MaxCompute identifies change in the table data and then begins the corresponding lifecycle calculation.

## Modify the Hash Clustering attribute of a table {#section_kpd_m4y_3hb .section}

You can execute the ALTER TABLE statement to add or remove the Hash Clustering attribute of a partitioned table.

Add the Hash Clustering attribute:

```
ALTERTABLE table_name 
    [CLUSTEREDBY(col_name [, col_name,...])[SORTED BY(col_name [ASC|DESC][, col_name [ASC|DESC]...])]INTO number_of_buckets BUCKETS]
```

Remove the Hash Clustering attribute:

```
ALTER TABLE table_name NOT CLUSTERED;
```

**Note:** 

-   The `ALTER TABLE` statement cannot modify the Clustering attribute of a non-partitioned table because this attribute cannot be modified once it is specified for a non-partitioned table.
-   The `ALTER TABLE` statement takes effect only on the new partitions of a table. Therefore, you do not need to execute the `PARTITIONALTER TABLE` statement for the existing partitions. After the Clustering attribute is added, data is stored to the new partitions in compliance with hash clustering.

## Empty data from a non-partitioned table {#section_zpx_1z1_wdb .section}

Empty the data in specified non-partition table. This command does not support partition table. For the partition table, use `ALTER TABLE table_name DROP PARTITION` to clear the data in partition.

Command format:

```
TRUNCATE TABLE table_name;
```

