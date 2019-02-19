# 解决DataWorks 10M文件限制问题最佳实践 {#concept_jtw_kfy_bfb .concept}

本文向您介绍如何解决用户在DataWorks上执行MapReduce作业的时候，文件大于10M的JAR和资源文件不能上传到DataWorks的问题，方便您使用调度功能定期执行MapReduce作业。

**解决方案：**

1.  将大于10M的resources通过[MaxCompute客户端](../../../../../intl.zh-CN/工具及下载/客户端.md#)上传。
    -   客户端下载地址：[客户端](../../../../../intl.zh-CN/工具及下载/客户端.md#)。
    -   客户端配置AK、EndPoint请参考：[安装并配置客户端](../../../../../intl.zh-CN/准备工作/安装并配置客户端.md#)和[配置Endpoint](../../../../../intl.zh-CN/准备工作/配置Endpoint.md#)。

        向客户端添加资源命令：

        ```
        //添加资源
        add jar C:\test_mr\test_mr.jar -f;
        ```

2.  目前通过MaxCompute客户端上传的资源，在DataWorks左侧资源列表是找不到的，只能通过list resources查看确认资源：

    ```
    //查看资源
    list resources;
    ```

3.  减小Jar文件\(瘦身Jar\)，因为DataWorks执行MR作业的时候，一定要在本地执行，所以保留一个main函数就可以。

    ```
    jar 
    -resources test_mr.jar,test_ab.jar --resources在客户端注册后直接引用
    -classpath test_mr.jar --瘦身策略：在gateway上提交要有main和相关的mapper和reducer，额外的三方依赖可以不需要，其他都可以放到resources
    com.aliyun.odps.examples.mr.test_mr wc_in wc_out;
    ```


通过上述方法，我们可以在DataWorks上运行大于10M的MR作业。

