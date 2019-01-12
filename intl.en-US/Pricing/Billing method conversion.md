# Billing method conversion {#concept_ttt_3dd_5db .concept}

This topic describes how to switch between the Pay-As-You-Go and Subscription billing methods.

MaxCompute allows you to switch the billing methods between Pay-As-You-Go and Subscription at any time.

**Note:** To switch between the two methods, you need to activate them up front.

-   The main difference between CU-based prepayment and I/O-based post payment is the billing and running methods of the computing resources. The billing standards for storage and data downloads are the same. If you select the CU-based prepayment method, you can only use purchased CU resources for your computing tasks. If you select the I/O-based post payment method, you can use public computing resources for your computing tasks. Note that the running speed of computing tasks depends on the total number of running tasks.
-   After you change the billing method, the new method generally takes effect immediately. However, for any tasks that are currently running, the new billing method takes effect during the next running period of the tasks.
-   To switch from I/O-based post payment to CU-based prepayment, you need to purchase MaxCompute CU resources in advance. The billing method can only be switched between workspaces in the same region.
-   After you switch from CU-based prepayment to I/O-based post payment for your workspaces, any fees that have been incurred will not be refunded. However, you can create other workspaces to use the purchased CU resources. If you have purchased MaxCompute CU resources, you can create multiple workspaces, which can share the CU resources.
-   We recommend that you do not switch the billing methods frequently, as it may affect your task running time.

## Procedure {#section_ils_pdd_5db .section}

1.  Log on to the DataWorks console.
2.  Select the region where the workspace is located and click **Modify Service** on the right of the workspace. On the **Choose Calculation Engine Services** page, select the required billing method. An example selection is shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11937/15472576481125_en-US.png)

    You can also click **Workspace Config** and select **Advanced Settings \> QuotaGroup** to switch the billing method.


