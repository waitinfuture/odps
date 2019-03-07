# 更新表数据（INSERT OVERWRITE and INTO） {#concept_yzd_ndb_wdb .concept}

本文向您介绍如何使用INSERT INTO和INSERT OVERWRITE两种命令更新表数据。

## INSERT操作 {#section_o5d_vdy_gfb .section}

**命令格式**

```
INSERT OVERWRITE|INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [(col1,col2 ...)]
select_statement
FROM from_statement;
```

**说明：** 

-   MaxCompute的INSERT语法与通常使用的MySQL或Oracle的INSERT[语法有差别](../../../../../cn.zh-CN/最佳实践/SQL/与标准SQL的主要区别及解决方法.md#)，在INSERT OVERWRITE|INTO后需要加入table关键字，而非直接使用tablename。
-   当Insert的目标表是分区表时，指定分区值`[PARTITION (partcol1=val1, partcol2=val2 …)]`语法中不允许使用函数等表达式。
-   目前INSERT OVERWRITE还不支持指定插入列的功能，暂时只能用INSERT INTO。
-   不支持INSERT INTO到hash clustering表

在MaxCompute SQL处理数据的过程中，INSERT OVERWRITE/INTO用于将计算的结果保存目标表中。

INSERT INTO与INSERT OVERWRITE的区别是：INSERT INTO会向表或表的分区中追加数据，而INSERT OVERWRITE会在向表或分区中插入数据前清空表中的原有数据。

**说明：** 在反复对同一个分区进行INSERT OVERWRITE的时候，通过DESCRIBE查看到的数据分区size会不同。这是因为从同一个表的同一个分区SELECT出来再INSERT OVERWRITE回相同分区时，文件切分逻辑会发生变化，因此会导致数据的size发生变化。但数据的总长度在INSERT OVERWRITE前后是不变的，因此您不必担心存储计费会存在问题。

在使用MaxCompute处理数据的过程中，INSERT OVERWRITE/INTO是最常用到的语句，它们会将计算的结果保存到一个表中，以供下一步计算使用。比如计算sale\_detail表中不同地区的销售额，操作如下：

```
create table sale_detail_insert like sale_detail;
alter table sale_detail_insert add partition(sale_date='2013', region='china');
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select shop_name, customer_id,total_price from sale_detail;
```

**说明：** 在进行INSERT更新表数据时，源表与目标表的对应关系依赖于在select子句中列的顺序，而不是表与表之间列名的对应关系。下面的SQL语句仍然是合法的：

```
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select customer_id, shop_name, total_price from sale_detail;
-- 在创建sale_detail_insert表时，列的顺序为：
-- shop_name string, customer_id string, total_price bigint
-- 而从sale_detail向sale_detail_insert插入数据是，sale_detail的插入顺序为：
-- customer_id, shop_name, total_price
-- 此时，会将sale_detail.customer_id的数据插入sale_detail_insert.shop_name
-- 将sale_detail.shop_name的数据插入sale_detail_insert.customer_id
```

向某个分区插入数据时，分区列不允许出现在select列表中。

```
insert overwrite table sale_detail_insert partition (sale_date='2013', region='china')
select shop_name, customer_id, total_price, sale_date, region  from sale_detail;
-- 报错返回，sale_date，region为分区列，不允许出现在静态分区的insert语句中。
```

同时，partition的值只能是常量，不可以出现表达式。以下用法是非法的：

```
insert overwrite table sale_detail_insert partition (sale_date=datepart('2016-09-18 01:10:00', 'yyyy') , region='china')
select shop_name, customer_id, total_price from sale_detail;
```

## 更新表数据到动态分区 {#section_anh_h5q_pgb .section}

动态分区使用注意事项：

-   在您insert into partition时，如果分区不存在，会自动创建分区。
-   如果多个insert into partition作业并发，同时发现分区不存在，都会主动创建分区，但是同时只有一个会创建成功，其它的都会失败。
-   insert into partition作业如果不能控制并发，只能通过预创建分区来避免问题。

更多动态分区详情请参考[输出到动态分区（DYNAMIC PARTITION）](cn.zh-CN/用户指南/SQL/INSERT操作/输出到动态分区（DYNAMIC PARTITION）.md#)。

