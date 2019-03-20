# 使用临时查询快速查询SQL（可选） {#concept_a13_chp_dhb .concept}

如果您已经创建了MaxCompute项目（DataWorks工作空间），可以直接使用DataWorks临时查询功能，快速书写SQL语句操作MaxCompute。

关于**临时查询**功能的具体信息，请参见[临时查询](../../../../../cn.zh-CN/使用指南/数据开发/临时查询.md#)。

## 进入临时查询 {#section_aqm_nhp_dhb .section}

点击[DataWorks控制台工作空间列表](https://workbench.data.aliyun.com/consolenew#/projectlist)，选择您需要进入的项目，点击**进入数据开发**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341133_zh-CN.png)

直接点击**临时查询**，右键**临时查询**，点击**新建节点** \> **ODPS SQL**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341134_zh-CN.png)

在弹框中输入节点名称，点击**提交**，创建您的临时查询节点。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341136_zh-CN.png)

## 运行SQL {#section_ibs_kjp_dhb .section}

现在您可以在刚刚创建的临时查询节点中运行[MaxCompute支持的SQL语句](../../../../../cn.zh-CN/用户指南/SQL/SQL概述.md#)了，我们以运行一个DDL语句[创建表](../../../../../cn.zh-CN/用户指南/SQL/DDL语句/表操作.md#)为例。

输入建表语句，点击**运行**即可。

```language-sql
create table if not exists sale_detail
(
shop_name     string,
customer_id   string,
total_price   double
)
partitioned by (sale_date string,region string);
-- 创建一张分区表sale_detail
```

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341138_zh-CN.png)

在弹框中您可以看到本次运行的费用预估，继续点击**运行**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341139_zh-CN.png)

您可以在下方的日志窗口，看到运行情况和最终结果：本次运行成功，结果为**OK**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/144329/155305251341140_zh-CN.png)

使用同样的方法，您也可以执行[查询语句](../../../../../cn.zh-CN/用户指南/SQL/SELECT操作/Select语法介绍.md#)。

