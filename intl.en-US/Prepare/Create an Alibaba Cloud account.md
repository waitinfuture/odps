# Create an Alibaba Cloud account {#task_skw_3pw_5db .task}

Before using MaxCompute, you must create an Alibaba Cloud account.

1.  Create an Alibaba Cloud account. 

    If you have not yet created an Alibaba Cloud account, you can go to the [Alibaba Cloud official website](https://www.alibabacloud.com/?spm=5176.8142029.388261.9.56bb6d3eCUYquH) and click **Free Account** to create an Alibaba Cloud account.

    **Note:** An Alibaba Cloud account has all operation permissions over its cloud resources. We recommend that you rotate your password periodically and do not share the password of your Alibaba Cloud account with other users to maintain your account security.

2.  Verify your identity. 

    You need to verify your identity before you can purchase and use Alibaba Cloud products. To do so, go to the page and follow the instructions.

3.  Create an AccessKey. 

    You need to create an AccessKey to run tasks in DataWorks. An AccessKey consists of an AccessKey ID and an AccessKey Secret, and is used for access permission verification between various Alibaba Cloud products. To create an AccessKey, follow these steps:

    1.   Log on to the Alibaba Cloud console, move the pointer over the user icon, and click **accesskeys**, as shown in the following figure.![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/15514327411361_en-US.png)

 
    2.   On the displayed page, click **Create AccessKey**, enter the verification code sent to your device, and click **Confirm**. On the displayed page, click **Save AccessKey Information**, as shown in the following figure.![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/15514327411363_en-US.png)

 
    You can then view the AccessKey status, and enable, disable, or delete an AccessKey as needed, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/15514327411368_en-US.png)

    If an AccessKey is disabled, services that use the AccessKey may not work properly, and an error might be reported. Exercise caution before disabling an AccessKey.

    **Note:** We recommend that you do not share your AccessKey with other users to maintain your account security. If an AccessKey is mistakenly disclosed, lost, or stolen, disable or update the AccessKey immediately.

4.   To create a MaxCompute project as a RAM user, [create an AccessKey](../../../../../reseller.en-US/Preparation/RAM User Operations.md#section_kdx_f4p_r2b) and attach the AccessKey to the RAM user. 

-   Create a RAM user.

    You can use your Alibaba Cloud account to create a RAM user to log on to the DTplus platform and use DataWorks.

    -   Create a RAM user named DataWorks-admin and save the AccessKey, as shown in the following figure.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/155143274138878_en-US.png)

    -   Check whether **Console Password Logon** is enabled. If not, click **Modify Logon Settings**, as shown in the following figure.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/155143274138879_en-US.png)

    -   Grant permission to the RAM user.

        To activate MaxCompute and create projects as a RAM user, you need to grant relevant permissions to the RAM user. To do so, follow these steps: Choose **Permissions** \> **Grants** and click **Grant Permission**. On the displayed page, set **Principal** to DataWorks-admin, select the AliyunDataWorksFullAccess or administrator policy, and click **Ok**, as shown in the following figure.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11942/155143274138880_en-US.png)

        **Note:** If a RAM user creates a project, the RAM user is the project administrator and the Alibaba Cloud account is the project owner.

        For more information about how to create a RAM user, see [Create a RAM user](../../../../../reseller.en-US/Preparation/Administrator Operations/Create a RAM user.md#).


