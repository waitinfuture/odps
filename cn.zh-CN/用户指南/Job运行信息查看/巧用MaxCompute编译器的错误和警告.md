# 巧用MaxCompute编译器的错误和警告 {#concept_ont_jpn_vdb .concept}

MaxCompute编译器基于MaxCompute2.0新一代的SQL引擎，显著提升了SQL语言编译过程的易用性与语言的表达能力。本文将为您介绍编译器的易用性改进。

## 编译器的易用性改进 {#section_lk4_ppn_vdb .section}

为了充分展示MaxCompute编译器的易用性改进，推荐您配合使用MaxCompute Studio。

首先，请[安装 MaxCompute Studio](../../../../../intl.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/IntelliJ IDEA安装步骤.md)，添加MaxCompute项目并创建工程，然后新建MaxCompute脚本文件，如下所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12087/15498667902373_zh-CN.png)

由上图可以发现以下问题：

-   第一个insert语句中wm\_concat函数使用有错误。
-   第二个insert语句中有一个错误和一个警告，错误是列名写错了，警告则是MaxCompute中，比较Bigint与Double时，会隐式转换为Double。因为从String到Double是有可能在运行时导致错误的转换，所以MaxCompute编译器会在此警告，请您确定此行为是否是您希望的。

鼠标停止在错误或者警告上，会直接提示具体错误或者警告信息。如果您不修改错误，直接提交，会被MaxCompute Studio阻拦，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12087/15498667902378_zh-CN.png)

所以，请您按照提示修改错误和警告，如下所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12087/15498667902374_zh-CN.png)

修改完毕后，再次提交脚本，便可以顺畅运行。

您也可以通过MaxCompute Studio把所有警告都设定为错误，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12087/15498667902375_zh-CN.png)

通过以上设置，可以保证不会不小心漏掉任何有可能的错误。

建议您在提交任何脚本之前，都使用MaxCompute Studio对脚本进行静态编译检查，并强烈推荐将警告设定为错误，在提交前修改所有的警告，这样可以避免浪费时间和资源。另外，提交有错误的脚本会被扣您的计算健康分，导致以后提交任务的优先级被下调，未来没有修改的警告也会被纳入到健康分体系，所以使用MaxCompute Compiler和Studio，可以永不降级。

警告中有很多情况是不安全的隐式类型转换，如果确实是您想要的转换，可以用cast \(xxx as\) 的方式消除警告；如果觉得这么写麻烦，MaxCompute编译器还提供一种简洁的方式（xxx），如上面修改过的脚本所示。

