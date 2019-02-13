# Lateral View {#concept_sqj_zhk_52b .concept}

Lateral view is used in conjunction with UDTF such as split, explode, etc. It can split a row of data into multiple rows, and aggregate the split data on this basis.

## Single Lateral View statement {#section_ih1_33k_52b .section}

Syntax:

```
lateralView: LATERAL VIEW [OUTER] udtf(expression) tableAlias AS columnAlias (',' columnAlias) * fromClause: FROM baseTable (lateralView)*
```

Notes:

-   Lateral view outer: When the table function does not output any rows, the corresponding Input rows remain in the Lateral View results, and all table function output lists are null.

Example:

Suppose we have a table called "pageAds" which has two columns of data.The first column is "pageid string" and the second column is "adid\_list", a comma-separated collection of AD IDs.

|  string pageid| Array<int\> adid\_list|
|:--------------|:----------------------|
|“front\_page”|\[1, 2, 3\]|
|“contact\_page”|\[3, 4, 5\]|

The requirement is to count the number of times all AD IDs have appeared. The implementation process is as follows.

1.  Split the AD IDs as follows:

    ```
    SELECT pageid, adid 
     FROM pageAds LATERAL VIEW explode(adid_list) adTable AS adid;
    ```

    The execution result is as follows:

    |string pageid|int adid|
    |:------------|:-------|
    |“front\_page”|1|
    |“front\_page”|2|
    |“front\_page”|3|
    |“contact\_page”|3|
    |“contact\_page”|4|
    |“contact\_page”|5|

2.  The statistics for the aggregation:

    ```
    SELECT adid, count(1) 
        FROM pageAds LATERAL VIEW explode(adid_list) adTable AS adid
    GROUP BY adid;
    ```

    Result:

    |int adid|count\(1\)|
    |:-------|:---------|
    |1|1|
    |2|1|
    |3|2|
    |4.|1|
    |50|1|


## Multiple Lateral View statements {#section_khl_tkk_52b .section}

A from statement can be followed by multiple Lateral View statements, the subsequent Lateral View statement can reference all the former tables and columns.

The following table is an example:

|Array<int\> col1|Array<string\> col2|
|:---------------|:------------------|
|\[1, 2\]|\[“a”, “b”, “c”\]|
|\[3, 4\]|\[“d”, “e”, “f”\]|

-   Execute a single statement:

    ```
    SELECT myCol1, col2 FROM baseTable
        LATERAL VIEW explode(col1) myTable1 AS myCol1;
    ```

    Result:

    |int mycol1|Array<string\> col2|
    |:---------|:------------------|
    |1|\[“a”, “b”, “c”\]|
    |2|\[“a”, “b”, “c”\]|
    |3| \[d”, “e”, “f”\]|
    |4| \[d”, “e”, “f”\]|

-   Add a Lateral View statement as follows:

    ```
    SELECT myCol1, myCol2 FROM baseTable
        LATERAL VIEW explode(col1) myTable1 AS myCol1
        LATERAL VIEW explode(col2) myTable2 AS myCol2;
    ```

    Result is as follows:

    |int myCol1|string myCol2|
    |:---------|:------------|
    |1|“a”|
    |1|“b”|
    |1|“c”|
    |2|“a”|
    |2|“b”|
    |2|“c”|
    |3|“d”|
    |3|“e”|
    |3|“f”|
    |4|“d”|
    |4|“e”|
    |4|“f”|


