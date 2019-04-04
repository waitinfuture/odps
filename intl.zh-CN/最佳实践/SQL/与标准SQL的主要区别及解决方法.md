# 与标准SQL的主要区别及解决方法 {#concept_tcb_h12_5db .concept}

本文将从习惯使用关系型数据库SQL用户的实践角度出发，列举用户在使用 MaxCompute SQL时比较容易遇见的问题。

## MaxCompute SQL基本区别 {#section_zt4_pb2_5db .section}

|主要区别|问题现象|解决办法|
|----|----|----|
|应用场景|不支持事务（没有 commit 和 rollback，不推荐使用 Insert Into）|建议代码具有等幂性支持重跑，推荐 Insert Overwrite 写入数据。|
|不支持索引和主外键约束|-|
|不支持自增字段和默认值|如果有默认值，请在数据写入时自行赋值。|
|表分区|单表支持 6 万个分区|-|
|一次查询输入的分区不能超过1万，否则执行会报错；另外如果是 2 级分区且查询时只根据 2 级分区进行过滤，总的分区数大于 1 万也可能导致报错|[一次查询输入的分区数不能大于 1 万](https://www.alibabacloud.com/help/zh/faq-detail/43152.htm)|
|[一次查询输出的分区数不能大于2048](https://www.alibabacloud.com/help/zh/faq-detail/44226.htm)|
|精度|DOUBLE类型存在精度问题|不建议在关联时候进行直接等号关联两个DOUBLE字段，推荐的做法是把两个数做减法，如果差距小于一个预设的值就认为是相同，比如 abs\(a1- a2\) < 0.000000001。|
|目前产品上已经支持高精度的类型DECIMAL|如果有更高精度要求的，可以先把数据存为 STRING类型，然后使用 UDF来实现对应的计算。|
|数据类型转换|各种预期外的错误，代码维护问题。|如果有2个不同的字段类型需要做Join，建议您先把类型转好后再Join。|
|日期型和字符串的隐式转换|在需要传入日期型的函数里如果传入一个字符串，字符串和日期类型的转换根据yyyy-mm-dd hh:mi:ss格式进行转换。|
|其他格式转换|[日期函数 \> TO\_DATE](http://kb.aliyun-inc.com/kb/48974)|

## DDL与DML的区别及解法 { .section}

|主要区别|问题现象|解决办法|
|----|----|----|
|表结构|不能修改分区列列名，只能修改分区列对应的值。|[分区和分区列的区别](https://www.alibabacloud.com/help/zh/faq-detail/40278.htm)|
|支持增加列，但是不支持删除列以及修改列的数据类型。|[SQL常见问题](https://www.alibabacloud.com/help/zh/faq-detail/40292.htm)|
|INSERT|语法上最直观的区别是：Insert into/overwrite 后面有个关键字Table。|-|
|数据插入表的字段映射不是根据Select的别名做的，而是根据Select的字段的顺序和表里的字段的顺序。|-|
|UPDATE/DELETE|目前不支持Update/Delete语句。|[更新和删除数据](https://www.alibabacloud.com/help/zh/faq-detail/40275.htm)|
|SELECT|[输入表的数量不能超过16张](https://www.alibabacloud.com/help/zh/faq-detail/44309.htm)|-|
|一个非分组列同一个Group By Key中的数据有多条，不使用聚合函数的话就没办法展示|Group by查询中的Select字段，应是Group By的分组字段，或者需要使用聚合函数。|
|子查询|子查询必须要有别名|建议查询不要带别名|
|IN/NOT IN|In/Not In,Exist/Not Exist，后面的子查询数据量不能超过 1000 条|[如何使用Not In](https://www.alibabacloud.com/help/zh/faq-detail/40282.html)|
|如果业务上已经保证了子查询返回结果的唯一性，可以考虑去掉Distinct，从而提升查询性能。|
|SQL返回10000条|MaxCompute限制了单独执行Select语句时返回的数据条数|[其他操作](http://kb.aliyun-inc.com/kb/27834)|
|需要查询的结果数据条数很多|[如何获取所有数据](https://www.alibabacloud.com/help/zh/faq-detail/40333.htm)|
|MAPJOIN|Join不支持笛卡尔积|Join必须要用on设置关联条件|
|如果有一些小表需要做广播表，需要用 Mapjoin Hint|
|[如何解决Join报错](https://www.alibabacloud.com/help/zh/faq-detail/40268.htm)|
|ORDER BY|Order By后面需要配合Limit n使用|如果希望做很大的数据量的排序，甚至需要做全表排序，可以把这个N设置的很大|
|[MaxCompute 查询数据的排序](https://www.alibabacloud.com/help/zh/faq-detail/40302.htm)|
|UNION ALL|参与UNION ALL运算的所有列的属性不同，抛异常|参与UNION ALL运算的所有列的数据类型、列个数、列名称必须完全一致|
|UNION ALL查询外面需要再嵌套一层子查询|-|

