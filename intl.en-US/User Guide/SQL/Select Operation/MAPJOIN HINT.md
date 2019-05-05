# MAPJOIN HINT {#concept_bf5_tkb_wdb .concept}

This topic describes how to use a `MAPJOIN` statement to join a large table with one or more small tables. It is faster than common `JOIN` operations.

A typical scenario of `MAPJOIN` is as follows: When the data volume is small, SQL loads all your specified small tables into the memory of the program by performing the `JOIN` operation to join your tables faster.

**Note:** When you use `MAPJOIN`, note the following:

-   The left table of a `LEFT OUTER JOIN` clause must be a large table.
-   The right table of a `RIGHT OUTER JOIN` clause must be a large table.
-   Both the left and right tables of an `INNER JOIN` clause can be large tables.
-   `MAPJOIN` cannot be used in a `FULL OUTER JOIN` clause.
-   `MAPJOIN` supports small tables as subqueries.
-   When `MAPJOIN` is used and a small table or subquery must be referenced, the alias must be referenced.
-   `MAPJOIN` supports non-equivalent `JOIN` conditions or multiple conditions connected by using `OR` statements. You can choose not to use `ON` statements. You can also use mapjoin on 1 = 1 to express a Cartesian product, for example, select /\* + mapjoin\(a\) \*/ a.id from shop a join table\_name b on 1=1, which may cause data expansion.
-   Currently, MaxCompute allows up to 256 small tables to be specified in a `MAPJOIN` statement. If you specify more than 256 small tables, a syntax error is returned.
-   If `MAPJOIN` is used, the total memory occupied by all small tables cannot exceed 640 MB. Note that MaxCompute uses compressed storage, so the data size is sharply expanded after small tables are loaded into the memory. The limit of 640 MB refers to the size after small tables are loaded into the memory.

**Example:** 

```
select /* + mapjoin(a) */
        a.shop_name,
        b.customer_id,
        b.total_price
from shop a join sale_detail b
on a.shop_name = b.shop_name;
```

MaxCompute SQL does not support complex `JOIN` conditions, such as non-equivalent expressions and the `OR` logic, in the `ON` conditions of common JOIN operations. However, `MAPJOIN` supports such operations.

**Example:** 

```
select /*+ mapjoin(a) */
        a.total_price,
        b.total_price
from shop a join sale_detail b
on a.total_price < b.total_price or a.total_price + b.total_price < 500;
```

