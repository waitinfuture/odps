# View operations {#concept_wjt_sj1_wdb .concept}

## Create view {#section_jpr_4z1_wdb .section}

Statement format:

```
CREATE [OR REPLACE] VIEW [IF NOT EXISTS] view_name
    [(col_name [COMMENT col_comment], ...)]
    [COMMENT view_comment]
    [AS select_statement]
```

**Note:** 

-   To create a view, you must have ‘read’ privilege on the table referenced by view.
-   Views can only contain one valid ‘select’ statement.
-   Other views can be referenced by a view, but this view cannot reference itself. Circular reference is not supported.
-   Writing the data into a view is not allowed, such as, using insert into or insert overwrite to operate view
-   After a view was created,it may be inaccessable if the referenced table is altered, such as deleting a referenced table.  You must maintain corresponding relationship between referenced tables and views.
-   If the option ‘if not exists’ is not specified and the view has already existed, using create viewcauses abnormality.  If this situation occurs, use create or replace view to recreate a view.  After reconstruction, the privileges keep unchanged.

Example:

```
create view if not exists sale_detail_view
(store_name, customer_id, price, sale_date, region)
comment 'a view for table sale_detail'
as select * from sale_detail;
```

## Drop view {#section_ynd_xz1_wdb .section}

Statement format:

```
DROP VIEW [IF EXISTS] view_name;
```

**Note:** If the view does not exist and the option \[if exists\] is not specified, error occurs.

Example:

```
DROP VIEW IF EXISTS sale_detail_view;
```

## Rename view {#section_y2p_11b_wdb .section}

Statement format:

```
ALTER VIEW view_name RENAME TO new_view_name;
```

**Note:** If the same name view has already existed, error occurs.

Example:

```
create view if not exists sale_detail_view
        (store_name, customer_id, price, sale_date, region)
        comment 'a view for table sale_detail'
        as select * from sale_detail;
    alter view sale_detail_view rename to market;
```

