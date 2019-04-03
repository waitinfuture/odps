# UDTF usage {#concept_bvw_yk1_1hb .concept}

This topic describes how to use Java user-defined table-valued functions \(UDTFs\) and Python UDTFs.

## Usage {#section_yjs_hpf_vdb .section}

In typical cases, a UDTF is used as follows in Structured Query Language \(SQL\):

```
select user_udtf(col0, col1, col2) as (c0, c1) from my_table; 
select user_udtf(col0, col1, col2) as (c0, c1) from (select * from my_table distribute by key sort by key) t;
select reduce_udtf(col0, col1, col2) as (c0, c1) from (select col0, col1, col2 from (select map_udtf(a0, a1, a2, a3) as (col0, col1, col2) from my_table) t1 distribute by col0 sort by col0, col1) t2;
```

UDTFs are subject to the following limits:

-   A select clause cannot contain any other expressions.

    ```
    select value, user_udtf(key) as mycol ...
    ```

-   A UDTF cannot be nested.

    ```
    select user_udtf1(user_udtf2(key)) as mycol...
    ```

-   A select clause cannot be used with a group by, distribute by, or sort by clause.

    ```
    select user_udtf(key) as mycol ... group by mycol
    ```


