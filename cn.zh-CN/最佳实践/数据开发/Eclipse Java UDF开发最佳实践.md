# Eclipse Java UDF开发最佳实践 {#concept_zkf_m1v_cgb .concept}

本文为您详细介绍使用Eclipse配合ODPS插件进行Java UDF开发的全流程操作。

**说明：** 更多资料请参考[Java UDF](../../../../../intl.zh-CN/用户指南/SQL/UDF/Java UDF.md#)。

## 使用Eclipse创建ODPS Project {#section_xxl_q1v_cgb .section}

在进行ODPS Project创建之前，请先确保您已成功安装[ODPS插件](../../../../../intl.zh-CN/工具及下载/Eclipse开发插件/安装Eclipse插件.md#)。

在Eclipse中单击**File** \> **New** \> **ODPS Project**，输入您的项目名称后，点击**Config ODPS console installation path**，配置您的[odpscmd客户端](../../../../../intl.zh-CN/工具及下载/客户端.md#)安装路径。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534302_zh-CN.png)

输入您的客户端**整体安装包**的路径后，点击**Apply**。ODPS插件会为您自动解析出客户端Version。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534304_zh-CN.png)

点击**Finish**完成项目创建。

## 在ODPS Project中创建Java UDF {#section_ngg_tcv_cgb .section}

在左侧**Package Exploer**中找到我们刚刚创建的ODPS Java UDF项目，右键选**New**，并在列表中选择**UDF**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534311_zh-CN.png)

输入UDF的Package名称（本例中为com.aliyun.example.udf）和Name（本例中为Upper2Lower），单击**Finish**完成UDF创建。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534316_zh-CN.png)

完成UDF创建后，您可以看到生成了默认Java代码，请注意**不要改变实现evaluate\(\)方法名称**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534317_zh-CN.png)

## 实现UDF类文件中的evaluate方法 {#section_umr_4gv_cgb .section}

将您想要实现的功能代码写到evaluate方法中，且不要改变evaluate\(\)方法的名称。这里为您简单示例一个大写字母转化为小写字母的功能。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534318_zh-CN.png)

```
package com.aliyun.example.udf;

import com.aliyun.odps.udf.UDF;

public class Upper2Lower extends UDF {
    public String evaluate(String s) {
        if (s == null) { return null; }
        return s.toLowerCase();
    }
}
```

完成代码后，请及时保存。

## 测试Java UDF代码 {#section_bfw_zgv_cgb .section}

为了测试Java UDF代码，我们可以首先在MaxCompute上存放一些大写字母作为输入数据。您可以利用odpscmd客户端使用SQL语句`create table upperABC(upper string);`新建一个名为upperABC的测试表格。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534320_zh-CN.png)

使用SQL语句`insert into upperABC values('ALIYUN');`在表格中插入测试用的大写字母“ALIYUN”。

完成测试数据准备后，点击**Run** \> **Run Configurations**配置测试参数。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534322_zh-CN.png)

配置测试参数：**Project**一栏中填写我们创建的Java ODPS Project名称，**Select ODPS project**中填写您的MaxCompute项目名称（请注意与odpscmd客户端当前连接的MaxCompute项目保持一致），**Table**一栏填写我们刚才创建的测试表格名称。完成配置后点击**Run**进行测试。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081534324_zh-CN.png)

您可以在下方的**Console**中看到测试结果，如下图所示。

**说明：** 测试结果只是Eclipse获取表格中的数据后在本地转换的结果，并不代表MaxCompute中的数据已经转换为小写的aliyun了。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634326_zh-CN.png)

## 导出Jar包 {#section_url_sqv_cgb .section}

确定测试结果正确后，就可以开始正式使用Java UDF了。首先您需要在左侧您新建的ODPS Project上右键单击，选择**Export**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634328_zh-CN.png)

在弹框中选择**JAR file**，点击**Next**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634329_zh-CN.png)

在对话框中JAR file处填写Jar包名称，单击**Finish**即可导出至当前workspace目录下。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634330_zh-CN.png)

## 使用DataWorks引用Jar包 {#section_nps_qrv_cgb .section}

登录您的DataWorks控制台，进入同一个项目（本例中为项目MaxCompute\_DOC）的[数据开发](../../../../../intl.zh-CN/使用指南/数据开发/界面功能/界面功能点介绍.md#)页面。选择**业务流程** \> **资源** \> **新建资源** \> **JAR**，新建一个[JAR类型资源](../../../../../intl.zh-CN/使用指南/数据开发/业务流程/资源.md#ul_u5d_411_t2b)。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634331_zh-CN.png)

在弹窗中上传您刚导出的Jar资源。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634334_zh-CN.png)

刚才只是将Jar资源上传到DataWorks，接下来您需要点击进入Jar资源，点击**提交并解锁（提交）**按钮将资源上传至 MaxCompute。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634335_zh-CN.png)

完成上传后，您可以在odpscmd客户端使用list resources命令查看您上传的Jar资源。

现在Jar资源已经存在于您的MaxCompute项目中了，接下来您需要点击**业务流程** \> **函数** \> **新建函数**，新建一个Jar资源对应的[函数](../../../../../intl.zh-CN/使用指南/数据开发/业务流程/注册函数.md#)，本例中命名函数名称为upperlower\_Java。函数的**资源列表**即为刚上传的Jar资源名称。完成后，依次点击**保存**和**提交并解锁（提交）**。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634337_zh-CN.png)

完成提交后，您可以在odpscmd客户端使用list functions命令查看您注册的函数。到此，您使用Eclipse开发工具完成注册的Java UDF函数upperlower\_Java已经可用了。

## Java UDF函数结果验证 {#section_pkt_mvv_cgb .section}

打开您的odpscmd命令行界面，运行`select upperlower_Java('ABCD') from dual;`命令，可以观察到该Java UDF已经可以转换字母的大小写了，函数运行正常。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823081634338_zh-CN.png)

如果您要使用Intellij IDEA开发工具完成完整的Java UDF开发过程，请参见[IntelliJ IDEA Java UDF开发最佳实践](intl.zh-CN/最佳实践/数据开发/IntelliJ IDEA Java UDF开发最佳实践.md#)。

