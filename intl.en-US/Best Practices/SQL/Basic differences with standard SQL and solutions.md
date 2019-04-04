# Basic differences with standard SQL and solutions {#concept_tcb_h12_5db .concept}

This article describes problems that occur frequently when users who are familiar with the relational database SQL use MaxCompute SQL.  For specific MaxCompute SQL syntax,  see [SQL  Overview](../../../../../reseller.en-US/User Guide/SQL/SQL summary.md).

## Basic differences of MaxCompute SQL {#section_zt4_pb2_5db .section}

**Scenarios **

-   Transactions are not supported. \(The commit and rollback actions are not supported. We recommend using the codes that have idempotence and support re-running. We also recommend  using Insert Overwrite, instead of Insert Into, to write data.\)
-   Indexes and primary and foreign key constraints are not supported.
-   Auto increment fields and default values are not supported.  If the default value exists, assign the value when writing the data.

**Table Partitioning**

-   A table supports 60,000 partitions.
-   Otherwise an error is returned. If the partitions are list partitions and the query performs filtering based only on list partitions,  an error may be returned if the total number of partitions exceeds 10,000.

**Precision**

-   The Double type has precision. Therefore, we do not recommend that you directly join two Double  fields using an equal sign \(=\).  We recommend deducting one number from the other, and consider the two numbers as the same if their difference is smaller than a preset value, for example,`abs(a1- a2)< 0.000000001`.
-   Currently, MaxCompute SQL supports the high-precision type Decimal.  If you have higher precision requirements, you can store the data as the string type and use UDF to implement such calculation.

**Data type conversion**

-   If two different field types need to be joined, to prevent unexpected errors, we recommend first converting the types before joining them. This also helps code maintenance.
-   For implicit conversions of dates and strings,  to input a string to a function that requires entering a date,  you can convert between the string and date according to the `yyyy-mm-dd hh:mm:ss` format.  For other formats, see [Date functions \> TO\_DATE](../../../../../reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md).

## DDL difference and solutions { .section}

**Table structure**

-   The partition column name cannot be changed. Only the value corresponding to the partition column can be changed. 
-   You can add columns but cannot delete columns and modify the data type of columns. You can add columns like this : `ALTER TABLE table_name ADD COLUMNS (col_name1 type1, col_name2 type2…)`

    If you must delete columns or modify the column data types, here is the most secure way:

    1.  Create a new table. For example: `CREATE TABLE new_table_name as SELECT c1,c2,c3 FROM table_name;`
    2.  Delete your original table and rename your new table as the original one. For example: `ALTER TABLE new_table_name as RENAME TO table_name;`Then you need to insert your data manually into your new table.

## DML difference and solutions {#section_bz4_hd2_5db .section}

**INSERT**

-   The most intuitive syntax difference is that insert into/overwrite is followed by the keyword  **Table**.
-   The field mapping of the data inserted into tables is not based on the alias of Select, but on the order of the fields of Select or the fields in the table.

**UPDATE/DELETE**

-   Currently, the Update/Delete statements are not supported.
-   If you want to perform Update, you may need to import source partition/table data into the new partition/table, and the corresponding update logic must be executed during the import process.
-   If you want to perform Delete, you can drop the table to delete data. For non partition table, you can empty the table data through the `TRUNCATE TABLE table_name;` statement. For partition table, you can delete partitions by `ALTER TABLE table_name DROP IF EXISTS PARTITION` \(partition name='specific partition value'\). You can also use `insert overwrite` statement to update or delete data.

**SELECT**

-   The Select field in the Group by query is either the group field of Group By or the aggregate function to be used.  From the logic perspective,  if a non-group column and Group By Key have multiple records of data, the data cannot be displayed without the aggregate function.

 **Subquery** 

Subqueries must have aliases.  We recommend that all queries have aliases.

**IN/NOT IN**

-   The data volume in the subquery following In/Not In and Exist/Not Exist cannot exceed 1000. To solve the problem.  If the uniqueness of the results returned by the subquery is guaranteed in the business aspect, you can consider removing  Distinct to improve the query performance.

**10,000 results returned by MaxCompute SQL**

-   MaxCompute limits the number of data records returned by the separately executed Select clause. For the specific configurations, see [Other operations](../../../../../reseller.en-US/User Guide/Common commands/Other operations.md). The maximum data number is set to 10,000. If the number of data records to be queried is large.

**MAPJOIN**

-   The Join clause does not support Cartesian products. The Join clause must use the on clause to set the joining conditions.  If you want to create broadcast tables for small tables, you must use MapJoin Hint.

**ORDER BY**

-   **Order By must be followed by Limit n**. If you want to sort a large number of data records, or even sort the full table, you can  set N to a large number.  However, use this method with caution. Because the system does not have advantages of distributed systems, the performance may be affected. 

**UNION ALL**

-   The data types, quantities, and names of all columns involved in the UNION ALL calculation must be consistent. Otherwise, an exception is returned.
-   The UNION ALL query must be nested by a subquery.

