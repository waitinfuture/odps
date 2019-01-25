# MAPJOIN HINT {#concept_bf5_tkb_wdb .concept}

MapJoin helps to join a large table with one or multiple small tables.It is faster than common Join operations.

A typical scenario of MapJoin,  is as follows: When the data volume is small, SQL loads all your specified small tables into the memory of the program performing the Join operation to speed up JOIN execution.

**Note:** When you use the MapJoin, note the following:

-   The left table of ‘left outer join’ must be a big table.
-   The right table of right outer join must be a big table.
-   For INNER JOIN, both the left and right tables can be large tables.
-   For FULL OUTER JOIN, MapJoin cannot be used.
-   MapJoin supports small tables as subqueries.
-   When MapJoin is used and a small table or subquery must be referenced, the alias must be referenced.
-   MapJoin supports non-equivalent JOIN conditions or multiple conditions connected using OR.
-   Currently, MaxCompute allows a maximum of eight small tables to be specified in MapJoin. Otherwise, a syntax error is returned.
-   If MapJoin is used, the total memory occupied by all small tables cannot exceed 512 MB. Note that MaxCompute uses compressed storage, so the data size is sharply expanded after small tables are loaded into the memory. The limit of 512 MB refers to the size after small tables are loaded into the memory. We can config the total memory by SET odps.sql.mapjoin.memory.max=2048;. The maximum configurable memory is 2048MB.
-   When JOIN is performed on the multiple tables, the two leftmost tables cannot be tables for MapJoin at the same time.

**For example:**

```
select /* + mapjoin(a) */
        a.shop_name,
        b.customer_id,
        b.total_price
    from shop a join sale_detail b
    on a.shop_name = b.shop_name;
```

MaxCompute SQL does not support complex JOIN conditions, such as non-equivalent expressions and the OR logic, in the ON condition of common JOIN operations. However, MapJoin supports such operations.

**For example:**

```
select /*+ mapjoin(a) */
        a.total_price,
        b.total_price
    from shop a join sale_detail b
    on a.total_price < b.total_price or a.total_price + b.total_price < 500;
```

