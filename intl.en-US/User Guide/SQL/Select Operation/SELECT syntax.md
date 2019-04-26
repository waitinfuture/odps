# SELECT syntax {#concept_i1q_lkb_wdb .concept}

This topic describes MaxCompute `SELECT` syntax and the precautions for executing nested queries, sorting operations, and grouping queries by using `SELECT` syntax.

## SELECT statement format {#section_dmz_cb1_hfb .section}

The `SELECT` statement format is as follows:

```
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[ORDER BY order_condition]
[DISTRIBUTE BY distribute_condition [SORT BY sort_condition] ]
[LIMIT number]
```

## Column expressions {#section_g04_7mz_4cj .section}

When using the `SELECT` statement to read data from a table, specify the names of the columns to be read, or use an asterisk \(\*\) wildcard to specify all columns. The following is an example `SELECT` statement:

```
select * from sale_detail;
```

To specify only the shop\_name column in the sale\_detail table, use the following statement:

```
select shop_name from sale_detail;
```

Use a `WHERE` clause when you specify a filtering condition. For example:

```
select * from sale_detail where shop_name like 'hang%';
```

When you use a `SELECT` statement, up to 10,000 rows of results can be displayed. If the `SELECT` statement serves as a clause, all the results are returned to the upper-level query.

**Note:**

-   Full table scans cannot be specified when `SELECT` statements are used on a partitioned table.

    For new projects created after January 10, 2018, full table scans cannot be specified for a partitioned table in a project. Instead, you must specify each partition condition to be scanned. This is to help reduce excessive resource usage and lower costs. If your instances are billed by the Pay-As-You-Go billing method, the total number of partition conditions to be scanned is regarded as one billable item.

    If the table definition is `t1(c1,c2) partitioned by(ds)`, you cannot run the following statement in a new project. Otherwise, an error may occur:

    ```
    Select * from t1 where c1=1;
    Select * from t1 where (ds=‘20180202’ or c2=3);
    Select * from t1 left outer join t2 on a.id =b.id and a.ds=b.ds and b.ds=‘20180101’);  
    --When Join statement is running, if the partition clipping condition is placed in where clause, the partition clipping takes effect. If you put it in on clause, the partition clipping of sub table takes effect, and the main table performs a full table scan.
    ```

    If you perform a full table scan on a partitioned table, you can add a set statement `set odps.sql.allow.fullscan=true;` before the SQL statement that is used to scan the partitioned table. The set statement must be entered together with the SQL statement. In the case that the sales\_detail table is a partitioned table, submit the following simple query statements at the same time for a full table scan:

    ```
    set odps.sql.allow.fullscan=true;
    select * from sale_detail;
    ```

-   table\_reference supports nested subqueries, for example:

    ```
    select * from (select region from sale_detail) t where region = 'shanghai';
    ```

    **Note:** When you use a `SELECT` statement, up to 10,000 rows of results can be displayed. If the `SELECT` statement serves as a clause, all the results are returned to the upper-level query.


**Use DISTINCT to remove duplicates**

`DISTINCT`: If duplicated data rows exist, you can use the Distinct option before the field to remove duplicates. In this case, only one value is returned. If you use the ALL option, or do not specify this option, all duplicated values in the fields are returned.

If you use the Distinct option, only one row of a record is returned, which is shown as follows:

```
select distinct region from sale_detail;
select distinct region, sale_date from sale_detail;
-- Performs the Distinct option on multiple columns. The Distinct option has an effect on Select column sets rather than a single column.
```

## **WHERE clause conditions** {#section_2to_4zy_w6o .section}

The filter conditions supported by `WHERE` clauses as shown in the following table:

|Filter criteria|Description|
|:--------------|:----------|
|\> ,<, =, \>=, <=, <\>|Relational operators|
|like, rlike|The source and pattern parameters of like and rlike can only be of the String type.|
|in, not in|If a subquery is attached to the in or not in a condition, only the values of one column are returned for the subquery, and the returned values cannot exceed 1,000 entries.|

You can specify a partition scope in the `WHERE` clause of a `SELECT` statement that is used to scan specified partitions of a table instead of a whole table, shown as follows:

```
SELECT sale_detail. * FROM sale_detail WHERE sale_detail.sale_date >= '2008' AND sale_detail.sale_date <= '2014';
```

The `WHERE` clause of MaxCompute SQL supports query by using the between-and condition. The preceding SQL statement can be rewritten as follows:

```
SELECT sale_detail. * FROM sale_detail WHERE sale_detail.sale_date BETWEEN '2008' AND '2014';
```

## Queries using GROUP BY {#section_7uh_ee0_ua8 .section}

`GROUP BY`: Query by group. In most cases, `GROUP BY` is used in combination with an [aggregation function](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md#). For a `SELECT` statement that contains an aggregate function to be used in combination with `GROUP BY`, it must comply with the following rules:

-   Rule i: The key using `GROUP BY` is the name of a column in the input table.
-   Rule ii: The key using `GROUP BY` is an expression consisting of columns of the input table. However, this expression cannot be the alias of an output column of the `SELECT` statement.

**Note:** In the case that rules i and ii are in conflicting with each other, rule i takes priority over rule ii. Specifically, if the key using `GROUP BY` is a column or expression of the input table and an output column of `SELECT`, rule i takes priority over rule ii.

**For example:** 

```
select region from sale_detail group by region;
-- Runs successfully with the name of a column in the input table directly used as the group by column
select sum(total_price) from sale_detail group by region;
-- Runs successfully with the table grouped by the region value and returns the total sales of each group
Select region, sum (total_price) from sale_detail group by region;
-- Runs successfully with the table grouped by the region value and returns the region value (unique in the group) and total sales of each group
select region as r from sale_detail group by r;
 -- Runs with the alias of the Select column and returns an error
select 2 + total_price as r from sale_detail group by 2 + total_price;
-- Requires a complete expression of the column
Select region, total_price from sale_detail group by region;
-- Returns an error; all columns not using an aggregate function in the Select statement must exist in group by
select region, total_price from sale_detail group by region, total_price;
-- Runs successfully
```

The preceding restrictions are imposed because `GROUP BY` operations come before `SELECT` operations during SQL parsing. Therefore, `GROUP BY` statements can only accept the columns or expressions of the input table as keys.

**Note:** For more information, see [Aggregate Functions](reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md).

## Queries using ORDER BY, SORT BY, and DISTRIBUTE {#section_yo1_g6w_oz2 .section}

-   `ORDER BY`: Globally sorts all data based on certain columns. To sort records in descending order, use the DESC keyword. For global sorting, **order by must be used together with limit**. When `ORDER BY` is used for sorting, NULL is considered to be smaller than any other value. This action is the same as that in MySQL but different from that in Oracle.

    Unlike `GROUP BY`, `ORDER BY` must be followed by the alias of the `SELECT` column. If the `SELECT` operation is performed on a column and the column alias is not specified, the column name is used as the column alias.

    ```
    select * from sale_detail order by region;
    -- Returns an error because order by is not used together with limit
    select * from sale_detail order by region limit 100;
    select region as r from sale_detail order by region limit 100;
    -- Returns an error because ORDER BY is not followed by a column alias
    select region as r from sale_detail order by r limit 100;
    ```

    The number in `[limit number]` is a constant to limit the number of output rows. If you want to directly view the result of a `SELECT` statement without LIMIT from the screen output, you can view up to 10,000 rows. The upper limit of screen display varies by project. However, the upper limit can be controlled through the setproject console.

-   `SORT BY`: For partial ordering, `DISTRIBUTE BY` must be added in front of the statement. `SORT BY` is used to partially sort the results of `DISTRIBUTE BY`. Aliases of `SELECT` output columns must be used.

    ```
    select region from sale_detail distribute by region sort by region;
    select region as r from sale_detail sort by region;
    -- Returns an error and exits because no distribute by exists.
    ```

-   `DISTRIBUTE BY`: Performs hash-based sharding on data by values of certain columns. Aliases of `SELECT` output columns must be used.

    ```
    select region from sale_detail distribute by region;
    -- Runs successfully because the column name is an alias
    select region as r from sale_detail distribute by region;
    -- Returns an error because DISTRIBUTE BY is not followed by a column alias
    select region as r from sale_detail distribute by r;
    ```


`ORDER BY` and `GROUP BY` cannot be used together with `DISTRIBUTE BY` and `SORT BY`. Aliases of `SELECT` output columns must be used.

**Note:** 

-   The keys of `ORDER BY`, `SORT BY`, and `DISTRIBUTE BY` must be output columns \(namely, column aliases\) of `SELECT` statements.
-   In MaxCompute SQL parsing, `ORDER BY`, `SORT BY`, and `DISTRIBUTE BY` come after `SELECT` operations. Therefore, they can only accept the output columns of `SELECT` statements as keys.

