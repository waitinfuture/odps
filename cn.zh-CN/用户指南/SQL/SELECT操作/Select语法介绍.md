# Select语法介绍 {#concept_i1q_lkb_wdb .concept}

本文向您介绍MaxCompute Select语法格式及使用Select语法执行嵌套查询、排序操作、分组查询等操作的注意事项。

## Select语法格式 {#section_dcq_y11_hfb .section}

命令格式：

```
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
FROM table_reference
[WHERE where_condition]
[GROUP BY col_list]
[ORDER BY order_condition]
[DISTRIBUTE BY distribute_condition [SORT BY sort_condition] ]
[LIMIT number]
```

使用Select语句时的**注意事项**请参见本章后续部分。

## 列表达式select\_expr {#section_xwx_cv2_ggb .section}

**Select**操作从表中读取数据，要读的列可以用列名指定，或者用\*代表所有的列，一个简单的Select语句，如下所示：

```
select * from sale_detail;
```

若您只读取sale\_detail的一列shop\_name，如下所示：

```
select shop_name from sale_detail;
```

在where中可以指定过滤的条件，如下所示：

```
select * from sale_detail where shop_name like 'hang%';
```

当使用Select语句屏显时，目前最多只能显示10000行结果。当Select作为子句时，无此限制，Select子句会将全部结果返回给上层查询。

**注意：**

-   Select分区表时**禁止全表扫描**。

    2018-01-10 20点后创建的新项目，默认情况下执行SQL时，针对该project里的分区表不允许全表扫描，必须有分区条件指定需要扫描的分区，由此减少SQL的不必要I/O，从而减少计算资源的浪费，同时也减少了不必要的后付费模式的计算费用（后付费模式中，数据输入量是计量计费参数之一）。

    例如表定义是`t1(c1,c2) partitioned by(ds)`，在新项目里执行如下语句会被禁止，返回error：

    ```
    Select * from t1 where c1=1;
    Select * from t1 where (ds=‘20180202’ or c2=3);
    Select * from t1 left outer join t2 on a.id =b.id and a.ds=b.ds and b.ds=‘20180101’);  
    --Join进行关联时，若分区剪裁条件放在where中，则分区剪裁生效，若放在on条件中，从表的分区剪裁会生效，主表则进行全表扫描。
    ```

    若实在需要对分区表进行全表扫描，可以在对分区表全表扫描的SQL语句前加一个set语句`set odps.sql.allow.fullscan=true;`，并和SQL语句一起提交执行。假设sale\_detail表为分区表，则要全表扫描需同时提交如下简单查询命令：

    ```
    set odps.sql.allow.fullscan=true;
    select * from sale_detail;
    ```

-   在table\_reference中支持使用嵌套子查询，如下所示：

    ```
    select * from (select region from sale_detail) t where region = 'shanghai';
    ```

    **说明：** 当使用Select语句屏显时，目前最多只能显示10000行结果。当select作为子句时，无此限制，select子句会将全部结果返回给上层查询。


**Distinct去重**

**Distinct**：如果有重复数据行时，在字段前使用distinct，会将重复字段去重，只返回一个值，而使用all将返回字段中所有重复的值，不指定此选项时默认效果和all相同。

使用distinct只返回一行记录，如下所示：

```
select distinct region from sale_detail;
select distinct region, sale_date from sale_detail;
-- distinct多列，distinct的作用域是 Select 的列集合，不是单个列。
```

## Where子句过滤 {#section_lwx_cv2_ggb .section}

**Where**子句支持的过滤条件，如下表所示：

|过滤条件|描述|
|:---|:-|
|\> 、 < 、 =、 \>=、 <=、 <\>|关系操作符|
|like、rlike|like和rlike的source和pattern参数均仅接受String类型。|
|in、not in|如果在in/not in条件后加子查询，子查询只能返回一列值，且返回值的数量不能超过1000。|

在Select语句的where子句中，您可以指定分区范围，这样可以仅仅扫描表的指定部分，避免全表扫描，如下所示。

```
SELECT sale_detail.* FROM sale_detail WHERE sale_detail.sale_date >= '2008'AND sale_detail.sale_date <= '2014';
```

MaxCompute SQL的where子句支持`between…and`条件查询，上述SQL可以重写：

```
SELECT sale_detail.* FROM sale_detail WHERE sale_detail.sale_date BETWEEN '2008' AND '2014';
```

## Group by分组查询 {#section_akw_f5f_ggb .section}

**group by**：分组查询，一般group by和[聚合函数](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md#)配合使用。在Select中包含聚合函数时有以下规则：

1.  用group by的key可以是输入表的列名。
2.  也可以是由输入表的列构成的表达式，不允许是Select语句的输出列的别名。
3.  规则1的优先级高于规则2。当规则1和规则2发生冲突时，即group by的key既是输入表的列或表达式，又是select的输出列，以规则1为准。

**示例**：

```
select region from sale_detail group by region;
-- 直接使用输入表列名作为group by的列，可以运行
select sum(total_price) from sale_detail group by region;
-- 以region值分组，返回每一组的销售额总量，可以运行
select region, sum(total_price) from sale_detail group by region;
-- 以region值分组，返回每一组的region值(组内唯一)及销售额总量，可以运行
select region as r from sale_detail group by r;
-- 使用select列的别名运行，报错返回
select 2 + total_price as r from sale_detail group by 2 + total_price;
-- 必须使用列的完整表达式
select region, total_price from sale_detail group by region;
-- 报错返回，select的所有列中，没有使用聚合函数的列，必须出现在group by中
select region, total_price from sale_detail group by region, total_price;
-- 可以运行
```

之所以有这样的限制，是因为在SQL解析中，group by操作通常是先于Select操作的，因此group by只能接受输入表的列或表达式为key。

**说明：** 

-   关于聚合函数的详情请参见[聚合函数](intl.zh-CN/用户指南/SQL/内建函数/聚合函数.md)。
-   **Order by**：对所有数据按照某几列进行全局排序。如果您希望按照降序对记录进行排序，可以使用DESC关键字。由于是全局排序，order by必须与limit共同使用。在使用order by排序时，Null会被认为比任何值都小，这个行为与MySQL一致，但是与Oracle不一致。

    与group by不同，order by后面必须加Select列的别名，当Select某列时，如果没有指定列的别名，将列名作为列的别名。

    ```
    select * from sale_detail order by region;
    -- 报错返回，order by没有与limit共同使用
    select * from sale_detail order by region limit 100;
    select region as r from sale_detail order by region limit 100;
    -- 报错返回，order by后面必须加列的别名。
    select region as r from sale_detail order by r limit 100;
    ```


## Limit number限制输出行数 {#section_dwy_n5f_ggb .section}

**\[Limit number\]**的number是常数，限制输出行数。当使用无limit的Select语句直接从屏幕输出查看结果时，最多只输出10000行。每个项目空间的这个屏显最大限制限制可能不同，可以通过setproject命令控制。

## Order by/Sort by/Distribute by {#section_elb_wx2_ggb .section}

-   **Distribute by**：对数据按照某几列的值做Hash分片，必须使用Select的输出列别名。

    ```
    select region from sale_detail distribute by region;
    -- 列名即是别名，可以运行
    select region as r from sale_detail distribute by region;
    -- 报错返回，后面必须加列的别名。
    select region as r from sale_detail distribute by r;
    ```

-   **Sort by**：局部排序，语句前必须加distribute by。实际上sort by是对distribute by的结果进行局部排序。必须使用Select的输出列别名。

    ```
    select region from sale_detail distribute by region sort by region;
    select region as r from sale_detail sort by region;
    -- 没有distribute by，报错退出。
    ```

-   **Order by**不和distribute by/sort by共用，同时**group by**也不和distribute by/sort by共用，**必须使用Select的输出列别名**。

**说明：** 

-   Order by/Sort by/Distribute by的key必须是Select语句的输出列，即列的别名。**列的别名可以为中文**。
-   在MaxCompute SQL解析中，Order by/Sort by/Distribute by是后于Select操作的，因此它们**只能接受Select语句的输出列为key**。

