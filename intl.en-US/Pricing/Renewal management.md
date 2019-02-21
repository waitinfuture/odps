# Renewal management {#concept_xpc_gxm_xdb .concept}

If your instance expires, the corresponding items in the expired instance are immediately locked by the system. You can renew your instance by using the Manually Renew option.

The renewal operation is described as follows:

**Note:** 

-   If an instance is not renewed in the 15 days after the expiration date, the corresponding resources in the instance are released, and all MaxCompute project data associated with the instance cannot be recovered.
-   If your instance expires, the corresponding items in the expired instance are immediately frozen by the system, and the instance enters the **Expiration** status.

## Manual renewal {#section_kfm_lxm_xdb .section}

You can renew instances on the **renewal management** page or the **project list** page.

**Renewal management page**

1.  Log on to the [DTplus console](https://renew-intl.console.aliyun.com/). In the left-side navigation pane, click **MaxCompute**.
2.  Go to **Bill Management** \> **Renew**.
3.  Click **Manually Renew** and select **MaxCompute**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/15507142303742_en-US.png)

4.  Select the instance to renew, and click **Renew**.
5.  Select the required renewal duration, and click **Pay**.

**Manual renewal using the project list page**

1.  Log on to the [DTplus console](https://renew-intl.console.aliyun.com/). In the left-side navigation pane, click **MaxCompute**, Click Project List.

2.  In the **Subscribed service** column, select the service icon that is displayed red, which indicates an expired instance. Click **Renew**.

     

3.  Select the required renewal duration, and click **Pay**.

## Auto-Renew {#section_rfm_lxm_xdb .section}

You can select **Auto-Renew** to renew instances automatically.

**Procedure**

1.  Log on to the [DTplus console](https://renew-intl.console.aliyun.com/). In the left-side navigation pane, click **MaxCompute**.
2.  Go to **Bill Management** \> **Renew**.
3.  Select **MaxCompute** in **Renew**,  click **Manually Renew**. 
4.  Click **Enable Auto-Renew**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/15507142303745_en-US.png)

5.  Select the Auto-renewal period, and click **Activate auto-renewal**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/15507142303746_en-US.png)

6.  After configuration is complete, the page displays the notification that the **Auto-renewal feature has been activated**. Enter the auto-renewal page to see the auto-renewal instance information.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/15507142303747_en-US.png)

7.  If you need to modify or cancel the Auto-Renew, click **Modify Auto-Renew** to **modify renewal cycle time**, click **Don’t Renew** to cancel the renewal activity, and click **OK**.

    **Note:** 

    -   The system attempts to process fees at three intervals: first, one day after expiration, second, the 6th day after expiration, and third, the 14th day after expiration. If all three deduction attempts fail, you must renew your instance manually, or your instance will be released and all MaxCompute data associated to the instance will be irrecoverable.

    -   To enable auto-renewal of an instance, you must configure auto-renew before 08:00 a.m. at least one day before the instance’s expiration date.


## No Renewal {#section_yfm_lxm_xdb .section}

If you do not want to renew your instances, or if you want to renew your instances manually, go to the Don’t Renew tab, and in the **Actions column** click Enable Manually Renew.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/15507142303748_en-US.png)

**Note:** If you select the Don’t Renew option, the system does not send an expiration reminder. However, 3 days before expiration, the system will send a reminder to you to verify that the Don’t Renew option matches your requirements.

