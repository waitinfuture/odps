# Column and Partition operation {#concept_ryh_5j1_wdb .concept}

This article shows you how to add, delete, and modify table partition command operations.

## Add partition {#section_dfj_q1b_wdb .section}

Statement format:

```
ALTER TABLE TABLE_NAME ADD [IF NOT EXISTS] PARTITION partition_spec
partition_spec:(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)
```

**Note:** 

-   The partition name must be lowercase.
-   Only ‘creating partitions’ are supported wherein, ‘creating partition columns’ are not supported.
-   If the same name partition has already existed and the option \[if not exists\] is not specified, an exception returns.
-   Currently, the maximum number of partitions supported in a single MaxCompute table is 60,000.
-   For tables that have multi-level partitions, to add a new partition, all partition values must be specified.

Example: 

add a new partition for the table ‘sale\_detail’.

```
alter table sale_detail add if not exists partition (sale_date='201312', region='hangzhou');
-- Add partition successfully, to store the sale detail of hangzhou region in December of 2013.
alter table sale_detail add if not exists partition (sale_date='201312', region='shanghai');
-- Add partition successfully, to store the sale detail of shanghai region in December of 2013.
alter table sale_detail add if not exists partition(sale_date='20111011');
-- Only specify a partition sale_date, error occurs and return.
alter table sale_detail add if not exists partition(region='shanghai');
-- Only specify a partition region, error occurs and return.
```

## Drop partition {#section_ahp_kbb_wdb .section}

Delete the syntax format for the partition is as follows:

```
ALTER TABLE TABLE_NAME DROP [IF EXISTS] PARTITION partition_spec;
partition_spec:(partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...)
```

**Note:** If the partition does not exist and the option \[if exists\] is not specified, then an error returns.

Example: 

delete a partition from the table sale\_detail.

```
alter table sale_detail drop if exists partition(sale_date='201312',region='hangzhou'); 
-- -Delete the sale details of Hangzhou in December of 2013 successfully.
```

## Add column {#section_k3l_4bb_wdb .section}

Statement format:

```
ALTER TABLE table_name ADD COLUMNS (col_name1 type1, col_name2 type2...)
```

```
ALTER TABLE table_name ADD COLUMNS (col_name1 type1 comment 'XXX',col_name2 type2 comment 'XXX');
```

**Note:** You cannot specify order for a new column. By default, a new column is placed in the last column.

## Modify column name {#section_k3x_qbb_wdb .section}

Statement format:

```
ALTER TABLE table_name CHANGE COLUMN old_col_name RENAME TO new_col_name;
```

**Note:** 

-   Column ‘old\_col\_name’ refers to an existing column.
-   A column named ‘new\_col\_name’ cannot exist in the table.

## Alter Column/Partition Comment {#section_ugq_pcb_wdb .section}

Modify column/partition comment is as follows:

```
ALTER TABLE table_name CHANGE COLUMN col_name COMMENT comment_string;
```

**Note:** The maximum comment content is 1024 bytes.

## Modify column names and column notes simultaneously {#section_vcd_scb_wdb .section}

Statement format:

```
ALTER TABLE table_name CHANGE COLUMN old_col_name new_col_name column_type COMMENT column_comment;
```

**Note:** 

-   Column ‘old\_col\_name’ must be an existing column.
-   A column named ‘new\_col\_name’ cannot exist in the table.
-   The content of the comment cannot exceed 1024 bytes.

## Modify LastDataModifiedTime of table/partition {#section_mnj_5cb_wdb .section}

MaxCompute MaxCompute SQL supports ‘touch’ operation to modify LastDataModifiedTime of a partition.  The result is to modify ‘LastDataModifiedTime’ of a partition to be current time.

**Statement format:**

```
ALTER TABLE table_name TOUCH PARTITION(partition_col='partition_col_value', ...)
```

**Note:** 

-   If ‘table\_name’ or ‘partition\_col’ does not exist, an error returns.
-   If the specified partition\_col\_value does not exist, an error returns.
-   This operation changes the value of ‘LastDataModifiedTime’ in the table and now MaxCompute determines whether the data of the table or partition has changed and the lifecycle calculation begins again.

## Modify partition value {#section_lrr_ycb_wdb .section}

MaxCompute SQL supports to change the partition value for corresponding partition value through ‘rename’ operation.

Statement format:

```
ALTER TABLE table_name PARTITION (partition_col1 = partition_col_value1, partition_col2 = partiton_col_value2, ...) 
RENAME TO PARTITION (partition_col1 = partition_col_newvalue1, partition_col2 = partiton_col_newvalue2, ...)
```

**Note:** 

-   The name of a partition column cannot be modified. Only the values in that column can be altered.
-   To modify values in one or more partitions among multi-level partitions, users must write values for partitions at each level.

