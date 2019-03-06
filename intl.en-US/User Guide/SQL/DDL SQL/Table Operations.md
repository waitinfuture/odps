# Table Operations {#concept_l3j_w31_wdb .concept}

This article shows you how to create, view, delete, rename and modify table information through the client.

## Create tables {#section_qqp_st1_wdb .section}

Statement format:

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
[STORED BY StorageHandler] -- Limited to external tables
[WITH SERDEPROPERTIES (Options)] -- Limited to external tables
[LOCATION OSSLocation];-- Limited to external tables
[LIFECYCLE days]
[As select_statement]
 CREATE TABLE [IF NOT EXISTS] table_name
 LIKE existing_table_name
```

Consider the following points:

-   When a table is created, an error is returned if the same name table exists without specifying the "if not exists" option. If the option is specified, no matter whether a same name table exists and even if the source table structure and the target table structure are inconsistent, all return successfully. The Meta information of the existing table does not change.
-   Both the table name and column name are case insensitive and cannot have special characters. It must begin with a letter and can include a-z, A-Z, digits, and underscores \(\_\). The name length cannot exceed 128 bytes.
-   1200 column definitions are allowed in a table.
-   The data types support Bigint、Double、Boolean、Datetime、Decimal and String, MaxCompute2.0 extends many [data types](../../../../../reseller.en-US/User Guide/Definitions/Data types.md).

    **Note:** Once data type such asTinyint、Smallint、 Int、 Float、Varchar or TIMESTAMP BINARY is involved when running an SQL statement, `set odps.sql.type.system.odps2=true;` must be added before the SQL statement. The set statement and SQL statement are submitted simultaneously.

-   Use Partitioned by to specify the [partition](../../../../../reseller.en-US/User Guide/Definitions/Partition.md) and now Tinyint、Smallint、 Int、 Bigint、Varchar and String are supported.

    The partition value cannot have a double byte characters \(for example, Chinese\), and must begin with an uppercase or a lowercase letter, followed by letter or a number. The name length cannot exceed 128 bytes. Special characters can be used, which include space, colon \(:\), underscore \(\_\), dollar sign \($\), pound sign \(\#\), period \(.\), exclamation point \(!\), and ’@’. Other characters such as \(\\t\), \(\\n\), \(/\), and so on are considered as undefined characters. When using partition fields in a partition table, to improve the processing efficiency, a full table scan is not needed to add, update, and read the data in a partition.

-   Currently, 60,000 partitions are allowed in a table, and the partition hierarchy cannot exceed 6 levels.
-   The comment content is the effective string and its length must not exceed 1024 bytes
-   Lifecycle indicates the lifecycle of the table, the unit is ‘days’. The statement create table like does not copy the lifecycle attribute from source table
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

The statement`create table...as select ...` can also be used to create a table. After creating a table, the data is copied to the new table, such as:

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

To let the destination table have the same structure as the source table, try to use `create table … like’` statement, such as:

```
create table sale_detail_like like sale_detail;
```

Now the table structure of `sale_detail_like` is exactly the same as `sale_detail`. Except the life cycle, attributes including the column name, column comment, and table comment, of the two tables are the same. But the data in `sale_detail` cannot be copied into the table `sale_detail_like`.

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

-   To view the infomation of the preceding table sale\_detail\_like, run the following statement:

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

Check the information of sale\_detail\_ctas1, you can find that sale\_date and region are only normal columns and not partitions of the table.

## Drop a table {#section_xvw_1w1_wdb .section}

Statement format:

```
DROP TABLE [IF EXISTS] table_name;
```

**Note:** 

-   If the option \[if exists\] is not specified and the table does not exist, exception returns.  If this option is specified, no matter whether the table exists or not, all return success.
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

## Alter Table Comments {#section_qnj_ny1_wdb .section}

Command format:

```
ALTER TABLE table_name SET COMMENT 'tbl comment';
```

**Note:** 

-   The table table\_name must exists.
-   The comment length must not exceed 1024 bytes.

For example:

```
alter table sale_detail set comment 'new comments for table sale_detail';
```

Use the command desc to view the comment modification in the table. For more information, see [Describe Table](reseller.en-US/User Guide/Common commands/Table operations.md).

## Alter Table LastDataModifiedTime {#section_gxl_5y1_wdb .section}

MaxCompute SQL supports touch operation to modify LastDataModifiedTime of a table. The result is to modify LastDataModifiedTime of a table to be current time.

Statement format:

```
ALTER TABLE table_name TOUCH;
```

**Note:** 

-   If the table table\_name does not exist, an error is returned.
-   This operation changes the value of LastDataModifiedTime of a table and this is when MaxCompute identifies change in the table data and then begins the corresponding lifecycle calculation.

## Empty data from a non-partitioned table {#section_zpx_1z1_wdb .section}

Empty the data in specified non-partition table, This command does not support partition table.  For the partition table, use `ALTER TABLE table_name DROP PARTITION` to clear the data in partition.

Command format:

```
TRUNCATE TABLE table_name;
```

