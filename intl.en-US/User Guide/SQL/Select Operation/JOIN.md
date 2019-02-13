# JOIN {#concept_cxf_rkb_wdb .concept}

The JOIN operation of MaxCompute supports n-way join, but does not support Cartesian product, that is, a link without the ON condition.

Function definition.

```
join_table:
        table_reference join table_factor [join_condition]
        | table_reference {left outer|right outer|full outer|inner} join table_reference join_condition
    table_reference:
        table_factor
        | join_table
    table_factor:
        tbl_name [alias]
        | table_subquery alias
        | ( table_references )
    join_condition:
        on equality_expression ( and equality_expression )*
```

**Note:** equality\_expression is an equality expression.

LEFT OUTER JOIN: Returns all records from the left table \(shop\) even if no matching row exists in the right table \(sale\_detail\).

```
select a.shop_name as ashop, b.shop_name as bshop from shop a
        left outer join sale_detail b on a.shop_name=b.shop_name;
    -- As the tables shop and sale_detail both have the shop_name column, aliases must be used in the select clause for distinguishing.
```

RIGHT OUTER JOIN: indicates the right join. It returns all records from the right table even if no matching record exists in the left table.

For example.

```
select a.shop_name as ashop, b.shop_name as bshop from shop a
        right outer join sale_detail b on a.shop_name=b.shop_name;
```

FULL OUTER JOIN: indicates the full join. It returns all records from both the left and the right table.

For example.

```
select a.shop_name as ashop, b.shop_name as bshop from shop a
        full outer join sale_detail b on a.shop_name=b.shop_name;
```

If at least one matching record exists in the table, INNER  JOIN returns the row. The keyword INNER can be ignored.

```
select a.shop_name from shop a inner join sale_detail b on a.shop_name=b.shop_name;
select a.shop_name from shop a join sale_detail b on a.shop_name=b.shop_name;
```

The join condition only allows equivalent conditions connected using and. Only MAPJOIN supports non-equivalent join conditions or multiple conditions connected using or.

```
select a.* from shop a full outer join sale_detail b on a.shop_name=b.shop_name
        full outer join sale_detail c on a.shop_name=c.shop_name;
     -- Supports n-way JOIN examples
select a.* from shop a join sale_detail b on a.shop_name ! = b.shop_name;
    -- Returns an error because non-equivalent JOIN conditions are not supported
```

IMPLICIT JOIN, MaxCompute supports the following JOIN method:

```
SELECT * FROM table1, table2 WHERE table1.id = table2.id;
--The execution effect is equivalent to
SELECT * FROM table1 JOIN table2 ON table1.id = table2.id;
```

