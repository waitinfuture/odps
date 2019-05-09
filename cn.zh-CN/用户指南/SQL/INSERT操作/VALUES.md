# VALUES {#concept_uhn_rdb_wdb .concept}

本文向您介绍`INSERT … VALUES`命令操作。

通常在业务测试阶段，需要给一个小数据表准备些基本数据，您可以通过`INSERT … VALUES`的方法在测试表中快速写入一些测试数据。

**说明：** 

目前`INSERT OVERWRITE`不支持这种指定插入列的功能，只能使用`INSERT INTO`实现。

命令格式如下。

```
INSERT INTO TABLE tablename 
[PARTITION (partcol1=val1, partcol2=val2,...)][(co1name1,colname2,...)] 
[VALUES (col1_value,col2_value,...),(col1_value,col2_value,...),...]
```

## 特定分区内插入数据 {#section_s9z_7zl_lnu .section}

示例

``` {#codeblock_j64_r6q_8il}
drop table if exists srcp;
create table if not exists srcp (key string,value bigint) partitioned by (p string);
insert into table srcp partition (p='abc') values ('a',1),('b',2),('c',3);
```

`INSERT … VALUES`语句执行成功后，查询表srcp分区`p=abc`，结果如下。

``` {#codeblock_u23_ogp_d1u}
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| a   | 1          | abc |
| b   | 2          | abc |
| c   | 3          | abc |
+-----+------------+---+
```

## 非特定分区内插入数据 {#section_bc7_alo_60d .section}

当表有很多列，而准备数据的时候希望只插入部分列的数据，此时可以使用插入列表功能。

示例

``` {#codeblock_kdn_eba_wx5}
drop table if exists srcp;
create table if not exists srcp (key string,value bigint) partitioned by (p string);
insert into table srcp partition (p)(key,p) values ('d','20170101'),('e','20170101'),('f','20170101');
```

`INSERT … VALUES`语句执行成功后，查询表srcp分区`p=20170101`，结果如下。

``` {#codeblock_6jc_ap2_oc9}
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| d   | NULL       | 20170101 |
| e   | NULL       | 20170101 |
| f   | NULL       | 20170101 |
+-----+------------+---+
```

**说明：** 

-   目前`INSERT OVERWRITE`不支持这种指定插入列的功能，只能使用`INSERT INTO`实现。
-   对于在values中没有指定的列，取缺省值NULL。插入列表功能不一定和values一起用，对于`insert into…select…`，同样可以使用。

## Values table功能 {#section_mie_trv_jec .section}

Values table并不限于在insert语句中使用，任何DML语句都可以使用。

`INSERT … VALUES`有一个限制：values必须是常量。但是当需要在插入的数据中进行一些简单的运算时，可使用MaxCompute的values table功能。

示例

``` {#codeblock_ejw_tke_c50}
drop table if exists srcp;
create table if not exists srcp (key string,value bigint) partitioned by (p string);
insert into table srcp partition (p) select concat(a,b), length(a)+length(b),'20170102' from values ('d',4),('e',5),('f',6) t(a,b);
```

`INSERT … VALUES`语句执行成功后，查询表srcp分区`p=20170102`，结果如下。

``` {#codeblock_a1c_pva_uwu}
+-----+------------+---+
| key | value      | p |
+-----+------------+---+
| d4  | 2          | 20170102 |
| e5  | 2          | 20170102 |
| f6  | 2          | 20170102 |
+-----+------------+---+
```

**说明：** 其中的`values (…), (…) t(a, b)`相当于定义了一个名为t，列为a、b的表，类型分别为STRING、BIGINT，其中的类型从values列表中推导。这样在不准备任何物理表时，可以模拟一个有任意数据的、多行的表，并进行任意运算。

`VALUES TABLE`这个用法还可以取代`select * from dual`与`union all`组合的方式，来拼出常量表，如下所示。

``` {#codeblock_74q_ssc_okp}
select 1 c from dual 
union all
select 2 c from dual;
--等同于 
select * from values (1), (2) as t(c);
```

还有一种values表的特殊形式，如下所示。

``` {#codeblock_3e5_pkt_yz5}
select abs(-1), length('abc'), getdate();
```

如上述语句所示，可以不写from语句，直接执行select（只要select表达式列表中不出现上游表的数据）。其底层实现为从一个1行，0列的匿名values表选取。这样，在您试图测试UDF或其他函数时，可免去手工创建DUAL表的过程。

**说明：** 

-   values只支持常量不支持函数，类似ARRAY复杂类型目前无法构造对应的常量，可以写为如下语句。

    ``` {#codeblock_6oj_pqv_mak}
    insert into table srcp (p='abc') select 'a',array('1', '2', '3');
    ```

    可达到一样的效果。

-   通过values写入DATETIME、TIMESTAMP类型，需要在values中指定类型名称，如下所示。

    ``` {#codeblock_xvq_xpq_u0r}
    insert into table srcp (p='abc') values (datetime'2017-11-11 00:00:00',timestamp'2017-11-11 00:00:00.123456789');
    ```


