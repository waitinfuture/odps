# Table {#concept_nbg_my1_5db .concept}

A table is the data storage unit in MaxCompute. A table is a two-dimensional data structure composed of rows and columns. Each row represents a record, and each column represents a field with the same data type. One record can contain one or more columns. The column name and data type comprise the schema of a table.

The operating objects \(input, output\) of various computing tasks in MaxCompute are tables. You can create a table, delete a table, and import data into a table.

**Note:** The data management module from DataWorks allows you to create, organize, and modify data lifecycles for MaxCompute tables and grant management permissions. For more information, see [data management overview](https://www.alibabacloud.com/help/doc-detail/30284.html).

MaxCompute v2.0 supports two types of tables: internal tables and external tables.The MaxCompute2.0 version begins to support the external table.

-   For internal tables, all data is stored in MaxCompute tables, and the columns in the table can be any of the data types supported by MaxCompute[Data types](reseller.en-US/User Guide/Definition/Data types.md#).
-   For external tables, data is not stored in MaxCompute. Instead, table data can be stored in [OSS](https://www.alibabacloud.com/product/oss) or [OTS](https://www.alibabacloud.com/product/ots).  MaxCompute only records meta information of the table. You can use MaxCompute’s external table to process unstructured data on OSS or Table Store, such as video, audio, genetics, meteorological, and geographic information.

Use of Dual tables：

-   Unlike databases such as Oracle, MaxCompute does not automatically create DUAL tables.
-   If you are used to using Dual tables as the most original test tables, you can manually use commands `CREATE TABLE IF NOT EXISTS DUAL (DUMMY VARCHAR(1));` to create an empty table named DUAL with only one field for testing.

    **Note:** At present, the Set commands supported by MaxCompute SQL and new version Mapreduce are divided into two ways:

    -   Session Level: To use the new data types \(Tinyint, Smallint, Int, Float, Varchar, TIMESTAMP BINARY\), add a set statement before the table statement

        ```
        set odps.sql.type.system.odps2=true;
        ```

        And submit the execution together with the table statement.

    -   Project level: that is to support new types of project level open. The Owner of project can set up project as required.

        ```
        setproject odps.sql.type.system.odps2=true;
        ```

-   DUAL is used in the same way as Oracle, such as `select getdate() from dual;`

