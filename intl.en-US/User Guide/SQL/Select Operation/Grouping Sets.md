# Grouping Sets {#concept_lth_qcw_hfb .concept}

This topic describes how to use the GROUPING SETS clause when you need to execute multiple UNION ALL clauses for scenarios where you need to aggregate and analyze data of multiple dimensions.

GROUPING SETS by MaxCompute is an extension to the GROUP BY clause in the SELECT statement. You can group results in various ways by using GROUPING SETS without executing multiple SELECT statements. This can allow the MaxCompute engine to produce better implementation plans with higher performance.

**Note:** A number of examples in this topic are demonstrated by using MaxCompute Studio. We recommend that you install MaxCompute Studio by following the instructions provided in [Install IntelliJ IDEA](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md#) before you proceed with subsequent operations.

## Example {#section_wrh_zcw_hfb .section}

1.  Prepare data.

    ```
    create table requests LIFECYCLE 20 as
    select * from values
        (1, 'windows', 'PC', 'Beijing'),
        (2, 'windows', 'PC', 'Shijiazhuang'),
        (3, 'linux', 'Phone', 'Beijing'),
        (4, 'windows', 'PC', 'Beijing'),
        (5, 'ios', 'Phone', 'Shijiazhuang'),
        (6, 'linux', 'PC', 'Beijing'),
        (7, 'windows', 'Phone', 'Shijiazhuang')
    as t(id, os, device, city);
    ```

2.  Use one of the following two methods to group data:
    -   Execute multiple SELECT statements.

        ```
        SELECT NULL, NULL, NULL, COUNT(*)
        FROM requests
        UNION ALL
        SELECT os, device, NULL, COUNT(*)
        FROM requests GROUP BY os, device
        UNION ALL
        SELECT null, null, city, COUNT(*)
        FROM requests GROUP BY city;
        ```

    -   Use GROUPING SETS.

        ```
        SELECT os,device, city ,COUNT(*)
        FROM requests
        GROUP BY os, device, city GROUPING SETS((os, device), (city), ());
        ```

        Data is aggregated as follows:

        ```
        +----+--------+------+------------+
        | os | device | city | cnt        |
        +----+--------+------+------------+
        | NULL | NULL   | NULL | 7          |
        | NULL | NULL   | Beijing | 4          |
        | NULL | NULL   | Shijiazhuang | 3          |
        | ios | Phone  | NULL | 1          |
        | linux | PC     | NULL | 1          |
        | linux | Phone  | NULL | 1          |
        | windows | PC     | NULL | 3          |
        | windows | Phone  | NULL | 1          |
        +----+--------+------+------------+
        ```

        **Note:** Expressions not used in GROUPING SETS use NULL as placeholders. You can execute UNION statements on grouping sets, such as the city column in lines 1 through 5.


## CUBE and ROLLUP {#section_q5g_jdw_hfb .section}

CUBE and ROLLUP are special GROUPING SETS functions.

CUBE lists all possible combinations of specified columns as grouping sets. ROLLUP aggregates data by level to produce grouping sets.

Example code:

```
GROUP BY CUBE\(a, b, c\) is equal to GROUPING SETS\(\(a,b,c\),\(a,b\),\(a,c\),\(b,c\),\(a\),\(b\),\(c\),\(\)\).

GROUP BY ROLLUP\(a, b, c\) is equal to GROUPING SETS\(\(a,b,c\),\(a,b\),\(a\)\).
```

## GROUPING and GROUPING\_ID {#section_d4j_kdw_hfb .section}

NULL is used as placeholders in grouping sets, but it can also be a value that is manually entered. In the code, however, placeholder NULLs are indistinguishable from value NULLs. The GROUPING function is provided to address this issue.

GROUPING allows you to specify the name of a column as a parameter. If the specified lines are aggregated based on a column whose name is used as a parameter in this function, 0 is returned, indicating that NULL is an entered value. Otherwise, 1 is returned, indicating that NULL is a placeholder.

GROUPING\_ID can be used to specify the names of one or more columns as parameters. The GROUPING results in these columns are formed into integers by using BitMap.

Example code:

```
SELECT a,b,c ,COUNT(*),
GROUPING(a) ga, GROUPING(b) gb, GROUPING(c) gc, GROUPING_ID(a,b,c) groupingid
FROM VALUES (1,2,3) as t(a,b,c)
GROUP BY CUBE(a,b,c);
```

Data is aggregated as follows:

```
+------------+------------+------------+------------+------------+------------+------------+------------+
| a          | b          | c          | _c3        | ga         | gb         | gc         | groupingid |
+------------+------------+------------+------------+------------+------------+------------+------------+
| NULL       | NULL       | NULL       | 1          | 1          | 1          | 1          | 7          |
| NULL       | NULL       | 3          | 1          | 1          | 1          | 0          | 6          |
| NULL       | 2          | NULL       | 1          | 1          | 0          | 1          | 5          |
| NULL       | 2          | 3          | 1          | 1          | 0          | 0          | 4          |
| 1          | NULL       | NULL       | 1          | 0          | 1          | 1          | 3          |
| 1          | NULL       | 3          | 1          | 0          | 1          | 0          | 2          |
| 1          | 2          | NULL       | 1          | 0          | 0          | 1          | 1          |
| 1          | 2          | 3          | 1          | 0          | 0          | 0          | 0          |
+------------+------------+------------+------------+------------+------------+------------+------------+
```

