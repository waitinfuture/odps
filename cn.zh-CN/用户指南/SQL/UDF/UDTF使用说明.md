# UDTF使用说明 {#concept_bvw_yk1_1hb .concept}

本文是适用于Java UDTF和Python UDTF的使用说明。

## 使用说明 {#section_yjs_hpf_vdb .section}

UDTF在SQL中的常用方式如下：

```
select user_udtf(col0, col1, col2) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

但使用UDTF有如下使用限制：

-   同一个SELECT子句中不允许有其他表达式。

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   UDTF不能嵌套使用。

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   不支持在同一个select子句中与group by / distribute by / sort by联用。

    ```
    select user_udtf(key) as mycol ... group by mycol
    ```


