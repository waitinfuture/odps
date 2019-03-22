# Table {#concept_nbg_my1_5db .concept}

A table is the data storage unit in MaxCompute. A table is a two-dimensional data structure composed of rows and columns. Each row represents a record, and each column represents a field with the same data type. One record can contain one or more columns. The column name and data type comprise the schema of a table.

The operating objects \(input, output\) of various computing tasks in MaxCompute are tables. You can create a table, delete a table, and import data into a table. For more information, see [Table operations](reseller.en-US/User Guide/Common commands/Table operations.md#).

**Note:** The data management module of DataWorks allows you to create, organize, and modify data lifecycles for MaxCompute tables and grant management permissions. For more information, see [Introduction](../../../../../reseller.en-US/User Guide/Data management/Introduction.md#).

MaxCompute 2.0 supports internal tables and external tables.

-   Data of internal tables is stored in MaxCompute. The columns in external tables can be of any [data types](reseller.en-US/User Guide/Definitions/Data types.md#) supported by MaxCompute.
-   Data of external tables is not stored in MaxCompute. Instead, this data can be stored in [OSS](https://www.alibabacloud.com/product/oss) or [OTS](https://www.alibabacloud.com/product/ots).  MaxCompute only records metadata of the external tables. You can use MaxCompute to process unstructured data of external tables, such as video, audio, or meteorological data.

**Note:** Use of DUAL tables:

-   Unlike databases such as Oracle, MaxCompute does not automatically create DUAL tables.
-   If you are using DUAL tables for testing, you can run the `CREATE TABLE IF NOT EXISTS DUAL (DUMMY VARCHAR(1));` command to create a table named DUAL with only one field for testing.
-   DUAL tables are used in the same way as Oracle. For example, you can run the `select getdate() from dual;` to use a DUAL table.

