# 开通Lightning服务 {#concept_vv5_qtt_z2b .concept}

MaxCompute Lightning是MaxCompute提供的交互式查询服务，您需要先开通并[创建MaxCompute项目](../../../../../cn.zh-CN/准备工作/创建项目.md#)方可使用MaxCompute Lightning。

MaxCompute Lightning服务现已开放华东1、华东2、华北2、亚太东南1等Region。在开放服务的Region内的项目如需使用，请单击[Lightning开通申请](https://account.aliyun.com/login/login.htm?oauth_callback=http%3A%2F%2Fpage.aliyun.com%2Fform%2FMaxCLightningPP%2Findex.htm)链接申请开通，我们会在1个工作日为您开通，同时您可以加入钉钉群“Lightning支持群”\(群号：23325499\)进行咨询、支持。

2019年2月1日起，MaxCompute将开始正式对Lightning服务进行计费。

Lightning查询任务的计费公式为：

```
一次Lightning查询费用=查询输入数据量*SQL价格
```

价格如下：

|计费项|价格|
|---|--|
|SQL价格|0.03元/GB|

**说明：** 

Lightning服务使用单独的计算资源，即使您购买了MaxCompute包年包月服务，使用Lightning时也需要对Lightning按查询付费。

Lightning按照每次SQL查询扫描的数据大小（每条查询至少10MB）付费。对于取消的查询，将按实际扫描的数据量收费。

不查询不产生任何费用。

MaxCompute默认对数据进行列式存储和数据压缩，Lightning的数据扫描大小按照压缩后的数据大小计算。

查询分区表应用分区过滤条件，也将减少数据扫描量并提升查询性能。

