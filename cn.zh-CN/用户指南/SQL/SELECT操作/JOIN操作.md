# JOIN操作 {#concept_cxf_rkb_wdb .concept}

MaxCompute的JOIN支持多路链接，但不支持笛卡尔积，即无on条件的链接。

命令格式如下：

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
        on equality_expression ( and equality_expression )
```

**说明：** equality\_expression是一个等式表达式。

-   left join：

    左连接，会从左表（shop）中返回所有的记录，即使在右表（sale\_detail）中没有匹配的行。

    ```
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            left outer join sale_detail b on a.shop_name=b.shop_name;
        -- 由于表shop及sale_detail中都有shop_name列，因此需要在select子句中使用别名进行区分。
    ```

-   right outer join：

    右连接，返回右表中的所有记录，即使在左表中没有记录与它匹配。

    示例如下。

    ```
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            right outer join sale_detail b on a.shop_name=b.shop_name;
    ```

-   full outer join：全连接，返回左右表中的所有记录。

    示例如下。

    ```
    select a.shop_name as ashop, b.shop_name as bshop from shop a
            full outer join sale_detail b on a.shop_name=b.shop_name;
    ```


如下示例中，表A是test\_table\_a，表B是test\_table\_b，用户用到join查询和where的过滤查询。查询分区大于20180101的数据中origin和id一致的记录数，使用left join希望可以保留左表中test\_table\_a的全表记录。正常情况下left join ，左表会是全的。right join，右表会是全的。

错误SQL示例如下：

```
SELECT s.id
        ,s.name
        ,s.origin
        ,d.value
FROM    test_table_a s
LEFT JOIN   test_table_b d
ON      s.origin = d.id
WHERE   s.ds > "20180101" AND d.ds>"20180101";
```

如果join在where条件之前， 会先做join操作，然后对join的结果做where条件过滤用户会发现获取的结果是两个表的交集，而不是全表。

修正后的SQL如下，通过这种方式可以显示全表。

```
SELECT s.id
        ,s.name
        ,s.origin
        ,d.value
FROM  (select * from  test_table_a where ds > "20180101" )s
LEFT JOIN (select * from  test_table_b where ds > "20180101") d
ON      s.origin = d.id;
```

在表中存在至少一个匹配时，inner join返回行，关键字inner可省略。

```
select a.shop_name from shop a inner join sale_detail b on a.shop_name=b.shop_name;
select a.shop_name from shop a join sale_detail b on a.shop_name=b.shop_name;
```

连接条件，只允许and连接的等值条件。只有在MAPJOIN中，可以使用不等值连接或者使用or连接多个条件。

```
select a.* from shop a full outer join sale_detail b on a.shop_name=b.shop_name
        full outer join sale_detail c on a.shop_name=c.shop_name;
    -- 支持多路join链接示例
select a.* from shop a join sale_detail b on a.shop_name != b.shop_name;
    -- 不支持不等值Join链接条件，报错返回。
```

IMPLICIT JOIN，MaxCompute支持如下Join方式。

```
SELECT * FROM table1, table2 WHERE table1.id = table2.id;
--执行的效果相当于
SELECT * FROM table1 JOIN table2 ON table1.id = table2.id;
```

