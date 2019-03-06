# Table operations {#concept_jvv_vk2_vdb .concept}

This article will show you how to create, delete and view tables using common commands through the MaxCompute client.

If you want to operate a table, use common commands in the client. Moreover it is more convenient to collect tables, apply permissions, and view partitions using the visible data table manager in DataWorks. For more information, see [Table Details](../../../../../reseller.en-US/User Guide/Data management/Table detail page.md#).

## Create tables {#section_wpz_zk2_vdb .section}

**Command format:**

```
CREATE TABLE [IF NOT EXISTS] table_name
 [(col_name data_type [COMMENT col_comment], ...)]
 [COMMENT table_comment]
 [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
 [LIFECYCLE days]
 [As select_statement]
CREATE TABLE [IF NOT EXISTS] table_name
 LIKE existing_table_name
```

**Action:**

Create a table.

**Note:** 

-   Both the table name and column name are case insensitive and follow the same naming conventions. The name can be up to 128 bytes in length and can contain letters, numbers, and underscores \(\_\).
-   The comment content is an effective string, and it can be up to 1,024 bytes in length.
-   \[LIFECYCLE days\]: The parameter ‘days’ refers to the time required to complete a ‘Table Operation’ lifecycle. It must be a positive integer. The unit is ‘day’.
-   Suppose that the ‘table\_name’ is no-partition table. If calculated from the last updated date, the data is still not modified after N \(days\), then MaxCompute automatically recycles the table without user intervention \(similar to ‘drop table’ operation\).
-   Suppose that the ‘table\_name’ is a partition table. MaxCompute determines whether to recycle the table according to LastDataModifiedTime of each partition. Unlike non-partitioned tables, a partitioned table is not deleted after all its partitions are reclaimed. The lifecycle can only be created for tables and not for the specified partitions.

**Example:**

```
CREATE TABLE IF NOT EXISTS sale_detail(
 shop_name STRING,
 customer_id STRING,
 total_price DOUBLE)
PARTITIONED BY (sale_date STRING,region STRING); --Create a partition table sale_detail.
```

## Drop Table {#section_gqz_zk2_vdb .section}

**Command format:**

```
DROP TABLE [IF EXISTS] table_name; -- Table name to be deleted.
```

**Action:**

-   Delete a table.
-   If the option \[IF EXISTS\] is specified, regardless of whether the table exists or not, the return is successful. If the option \[IF EXISTS\] is not specified, and the table does not exist, an exception is returned.

**Example:**

```
DROP TABLE sale_detail; -- If the table exists, success returns.
DROP TABLE IF EXISTS sale_detail; -- No matter whether the table sale_detail exists or not, success returns.
```

## Describe Table {#section_lln_ql2_vdb .section}

**Command format:**

```
DESC <table_name>; -- Table name or view name.
DESC extended <table_name>; -- View the extended table information.
```

**Action:**

Return information of a specified table, includes:

-   Owner: The owner of the table.
-   Project: The project to which a table belongs.
-   CreateTime: The creation time of the table.
-   LastDDLTime: The last DDL operation.
-   LastModifiedTime: The last time of table modification.
-   InternalTable: Indicates the object to be described is table. The value is ‘YES’ by default.
-   Size: Storage size occupied by table data, usually the compression ratio is 5. The unit is Byte.
-   Native Columns: Non-partition column information, including column name, type, comment.
-   Partition Columns: Partition column information, including partition name, type, and comment.
-   Extended Info: The information of extended table, such as StorageHandler and Location.

**Example:**

```
odps@ project_name>DESC sale_detail; -- Describe a partition table.
+------------------------------------------------------------------------------------+
| Owner: ALIYUN$odpsuser@aliyun.com | Project: test_project |
| TableComment: |
+------------------------------------------------------------------------------------+
| CreateTime: 2014-01-01 17:32:13 |
| LastDDLTime: 2014-01-01 17:57:38 |
| LastModifiedTime: 1970-01-01 08:00:00 |
+------------------------------------------------------------------------------------+
| Internaltable: Yes | size: 0 |
+------------------------------------------------------------------------------------+
| Native Columns: |
+------------------------------------------------------------------------------------+
| Field | Type | Comment |
+------------------------------------------------------------------------------------+
| shop_name | string | |
| customer_id | string | 
| total_price | double | |
+------------------------------------------------------------------------------------+
| Partition Columns: |
+------------------------------------------------------------------------------------+
| sale_date | string | |
| region | string | |
+------------------------------------------------------------------------------------+
```

**Note:** 

-   The preceding example is executed using the MaxCompute client.
-   If the table has no partition, the information of Partition Columns is not displayed.
-   To describe a 'View', the option ‘InternalTable’ cannot be displayed but the option ‘VirtualView’ can be displayed and its value is ‘YES’ by default. Similarly, the 'Size' option is replaced by the 'View Text' option, which represents the definition of the view, for example: `select * from src`. For more information, see[View operations](reseller.en-US/User Guide/SQL/DDL SQL/View operations.md#).

## View partition table {#section_gbd_lt2_vdb .section}

**Command format:**

```
desc table_name partition(pt_spec
```

**Action:**

View the specific partition information of a partition table.

**Example:**

```
odps@ project_name>desc meta.m_security_users partition (ds='20151010');
+------------------------------------------------------------------------------------+
| PartitionSize: 2109112 |
+------------------------------------------------------------------------------------+
| CreateTime: 2015-10-10 08:48:48 |
| LastDDLTime: 2015-10-10 08:48:48 |
| LastModifiedTime: 2015-10-11 01:33:35 |
+------------------------------------------------------------------------------------+
OK
```

## Show Tables/Show Tables like {#section_nw1_m52_vdb .section}

**Command format:**

```
SHOW TABLES;
SHOW TABLES like 'chart';
```

**Action:**

-   SHOW TABLES: List all tables of current project.
-   SHOW TABLES like 'chart': Lists the tables on which the following table names of the current project match the 'chart'. Regular expressions are supported.

**Example:**

```
odps@ project_name>show tables;
odps@ project_name>show tables like 'ods_brand*';
ALIYUN$odps_user@aliyun.com:table_name
......
```

**Note:** 

-   The preceding example is executed using the MaxCompute client.
-   ALIYUN is a system prompt, indicating the you are an Alibaba Cloud user.
-   In this example,odps\_user@aliyun.com is the creator of the table in this example.
-   In this example,table\_name is the name of the table.

## Show Partitions {#section_bwc_bv2_vdb .section}

**Command format:**

```
SHOW PARTITIONS ;  -- table_name: Specify the table to be queried. If the table does not exist or it is not a partition table, an exception is thrown.
```

**Action:**

List all partitions of a table.

**Example:**

```
odps@ project_name>SHOW PARTITIONS table_name;
partition_col1=col1_value1/partition_col2=col2_value1
partition_col1=col1_value2/partition_col2=col2_value2
…
```

**Note:** 

-   The preceding example is executed using the MaxCompute client.
-   Partition\_col1 and partition\_col2 are the partition columns of the table.
-   Col1\_value1, col2\_value1, col1\_value2, and col2\_value2 are corresponding values of the partition columns.

