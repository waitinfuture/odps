# UNION, INTERSECT, and EXCEPT {#concept_m5k_pkb_wdb .concept}

This topic describes the syntax of UNION ALL, UNION DISTINCT, INTERSECT ALL, INTERSECT DISTINCT, EXCEPT ALL, and EXCEPT DISTINCT, and provides examples of query statements.

The following are example syntax statements:

```
select_statement UNION ALL select_statement;
select_statement UNION [DISTINCT] select_statement;
select_statement INTERSECT ALL select_statement;
select_statement INTERSECT [DISTINCT] select_statement;
select_statement EXCEPT ALL select_statement;
select_statement EXCEPT [DISTINCT] select_statement;
select_statement MINUS ALL select_statement;
select_statement MINUS [DISTINCT] select_statement;
```

-   UNION: Returns the union of two data sets.
-   INTERSECT: Returns the intersection of two data sets.
-   EXCEPT: Returns the supplementary set of a data set in another set.
-   MINUS: Means the same as EXCEPT.

Examples

-   UNOIN ALL

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4) t(a, b) 
     UNION ALL 
    SELECT * FROM VALUES (1, 2), (1, 4) t(a, b);
    ```

    Result: The two data sets are merged.

    ```
    +------------+------------+
      | a          | b          |
      +------------+------------+
      | 1          | 2          |
      | 1          | 4          |
      | 1          | 2          |
      | 1          | 2          |
      | 3          | 4          |
      +------------+------------+
    ```

-   UNION DISTINCT

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4) t(a, b) 
    UNION 
    SELECT * FROM VALUES (1, 2), (1, 4) t(a, b);
    ```

    Result: The result is the same as `SELECT DISTINCT * FROM (< Result of UNOIN ALL>) t;`.

    ```
    +------------+------------+
     | a          | b          |
     +------------+------------+
     | 1          | 2          |
     | 1          | 4          |
     | 3          | 4          |
     +------------+------------+
    ```

-   INTERSECT ALL

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (5, 6) t(a, b) 
    INTERSECT ALL 
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (5, 7) t(a, b);
    ```

    Result: Repeated rows are not deleted. Each repeated row can be regarded as a unique row with a unique number.

    ```
    +------------+------------+
     | a          | b          |
     +------------+------------+
     | 1          | 2          |
     | 1          | 2          |
     | 3          | 4          |
     +------------+------------+
    ```

-   INTERSECT DISTINCT

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (5, 6) t(a, b) 
    INTERSECT 
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (5, 7) t(a, b);
    ```

    Result: The result is the same as `SELECT DISTINCT * FROM (< Result of INTERSECT ALL>) t;`.

    ```
    +------------+------------+
     | a          | b          |
     +------------+------------+
     | 1          | 2          |
     | 3          | 4          |
     +------------+------------+
    ```

-   EXCEPT ALL

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b) 
    EXCEPT ALL 
    SELECT * FROM VALUES (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);
    ```

    Result: Repeated rows are not deleted. Each repeated row can be regarded as a unique row with a unique number.

    ```
    +------------+------------+
      | a          | b          |
      +------------+------------+
      | 1          | 2          |
      | 1          | 2          |
      | 3          | 4          |
      | 7          | 8          |
      +------------+------------+
    ```

-   EXCEPT DISTINCT

    ```
    SELECT * FROM VALUES (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b) 
    EXCEPT
    SELECT * FROM VALUES (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);
    ```

    Result: The result is the same as `SELECT DISTINCT * FROM left_branch EXCEPT ALL SELECT DISTINCT * FROM right_branch;`.

    ```
    +------------+------------+
    | a          | b          |
    +------------+------------+
    | 1          | 2          |
    | 7          | 8          |
    +------------+------------+
    ```

    **Note:** 

    -   The sequence of rows in the result set is not guaranteed to be in any specific order.
    -   The two data sets in one set operation must contain the same number of columns. If not, implicit type conversion is involved. Out of compatibility reasons, implicit type conversion of STRING and non-STRING data in set operations is disabled.
    -   Up to 256 tables can be contained in one set operation. If more than 256 tables are involved, an error is returned.
    -   If UNION is followed by CLUSTER BY, DISTRIBUTE BY, SORT BY, ORDER BY, or a LIMIT clause, when `set odps.sql.type.system.odps2=false;`, the clause applies to the last SELECT statement. When `set odps.sql.type.system.odps2=true;`, the clause applies to all the preceding UNION results. Consider the following example:

        ```
        set odps.sql.type.system.odps2=true;
          SELECT explode(array(3, 1)) AS (a) UNION ALL SELECT explode(array(0, 4, 2)) AS (a) ORDER 
          BY a LIMIT 3;
        ```

        Result:

        ```
        +------+
         | a    |
         +------+
         | 0    |
         | 1    |
         | 2    |
         +------+
        ```


