# Lateral View {#concept_sqj_zhk_52b .concept}

Lateral View和split，explode等UDTF一起使用，它能够将一行数据拆成多行数据，并在此基础上对拆分后的数据进行聚合。

## 单个Lateral View语句 {#section_ih1_33k_52b .section}

语法定义如下：

```
lateralView: LATERAL VIEW [OUTER] udtf(expression) tableAlias AS columnAlias (',' columnAlias) * fromClause: FROM baseTable (lateralView)*
```

说明如下：

-   Lateral view outer：当table function不输出任何一行时，对应的输入行在Lateral view结果中依然保留，且所有table function输出列为null。

示例如下：

假设我们有一张表pageAds，它有两列数据，第一列是pageid string，第二列是adid\_list，即用逗号分隔的广告ID集合。

|string pageid|Array<int\> adid\_list|
|:------------|:---------------------|
|“front\_page”|\[1, 2, 3\]|
|“contact\_page”|\[3, 4, 5\]|

需求是要统计所有广告ID在所有页面中出现的次数，实现过程如下所示。

1.  拆分广告ID，如下所示：

    ```
    SELECT pageid, adid 
     FROM pageAds LATERAL VIEW explode(adid_list) adTable AS adid;
    ```

    执行结果如下：

    |string pageid|int adid|
    |:------------|:-------|
    |“front\_page”|1|
    |“front\_page”|2|
    |“front\_page”|3|
    |“contact\_page”|3|
    |“contact\_page”|4|
    |“contact\_page”|5|

2.  进行聚合的统计，语句如下：

    ```
    SELECT adid, count(1) 
        FROM pageAds LATERAL VIEW explode(adid_list) adTable AS adid
    GROUP BY adid;
    ```

    执行结果如下：

    |int adid|count\(1\)|
    |:-------|:---------|
    |1|1|
    |2|1|
    |3|2|
    |4|1|
    |5|1|


## 多个Lateral View语句 {#section_khl_tkk_52b .section}

一个from语句后可以跟多个Lateral View语句，后面的Lateral View语句能够引用它前面的所有表和列名。

以下面的表为例：

|Array<int\> col1|Array<string\> col2|
|:---------------|:------------------|
|\[1, 2\]|\[“a”, “b”, “c”\]|
|\[3, 4\]|\[“d”, “e”, “f”\]|

-   执行单个语句：

    ```
    SELECT myCol1, col2 FROM baseTable
        LATERAL VIEW explode(col1) myTable1 AS myCol1;
    ```

    执行结果如下所示：

    |int mycol1|Array<string\> col2|
    |:---------|:------------------|
    |1|\[“a”, “b”, “c”\]|
    |2|\[“a”, “b”, “c”\]|
    |3|\[“d”, “e”, “f”\]|
    |4|\[d”, “e”, “f”\]|

-   加上一个Lateral View语句，如下所示：

    ```
    SELECT myCol1, myCol2 FROM baseTable
        LATERAL VIEW explode(col1) myTable1 AS myCol1
        LATERAL VIEW explode(col2) myTable2 AS myCol2;
    ```

    执行结果如下所示：

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


