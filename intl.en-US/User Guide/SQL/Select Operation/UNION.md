# UNION {#concept_m5k_pkb_wdb .concept}

The UNION ALL/UNION \[DISTINCT\] operator is used to merge the result set of two or more SELECT statements.

**The syntax format is as follows**:

```
select_statement UNION ALL select_statement;
select_statement UNION [DISTINCT] select_statement;
```

-   UNION ALL: Combines two or multiple data sets returned by a SELECT operation into one data set. If the result contains duplicated rows, all rows that meet the conditions are returned, and deduplication of duplicated rows is not applied.
-   UNION \[DISTINCT\]: In this statement, DISTINCT can be ignored.  It combines two or multiple data sets returned by a SELECT operation into one data set. If the result contains duplicated rows, deduplication is applied.

Following is an example of the UNION ALL operation:

```
Select * From sale_detail where region = 'Hangzhou'
        union all
select * from sale_detail where region = 'shanghai';
```

Following is an example of the UNION operation:

```
SELECT * FROM src1 UNION SELECT * FROM src2; 
--The execution effect is equivalent to  
SELECT DISTINCT * FROM (SELECT * FROM src1 UNION ALL SELECT * FROM src2) t;
```

**Note:** 

-   The number and types of queried columns corresponding to the UNION ALL/UNION operation must be consistent.
-   Generally, MaxCompute allows UNION ALL/UNION operations performed on a maximum of 256 tables. A syntax error is returned if the number of tables exceeds this limit.

The meaning of LIMIT following UNION:

If UNION is followed by CLUSTER BY, DISTRIBUTE BY, SORT BY, ORDER BY, or a LIMIT clause, the clause has an effect on all the preceding UNION results rather than the last SELECT statement of UNION. Currently, MaxCompute adopts this action in `set  odps.sql.type.system.odps2=true;`. 

**For example**:

```
set odps.sql.type.system.odps2=true;
SELECT explode(array(3, 1)) AS (a) UNION ALL SELECT explode(array(0, 4, 2)) AS (a) ORDER BY a LIMIT 3;
```

The returned result is as follows:

```

| a |

| 0 |
| 1 |
| 2 |

```

