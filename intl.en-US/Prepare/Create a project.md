# Create a project {#task_zl2_mtx_5db .task}

A project is the basic unit of operation in MaxCompute.

1.  Log on to the [DataWorks console](https://partners-intl.aliyun.com) and create a MaxCompute project in either of the two following ways. 
    -   On the console overview page, go to **Common Functions** \> **Create Project**.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11944/15506529671414_en-US.png)

        **Note:** Creating a workspace is creating a MaxCompute project.

    -   On the console overview page, click **Project List**, select the region, and then click **Create Project**.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11944/15506529671416_en-US.png)

2.   Configure in the **Create Project** dialog box, select region and service. If  you do not buy related services for the selected region,  **'Service is not available for this region'** is displayed. The data development, O&M center, and data management are selected by default. as shown in the following figure:![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11944/15506529671417_en-US.png)

 

    **Select calculation engine services:**

    -   If you want to add and authorize users using DataWorks, see [Add Users and Roles](../../../../../reseller.en-US/Preparation/Administrator Operations/Add users and roles.md#).
    **DataWorks service:**

    -   Data integration: A data synchronization platform that provides stable, efficient, and elastically scalable services.  It is designed to implement fast and stable data movement and synchronization between various heterogeneous data sources in complex network environments. For more information, see [Data Integration Overview](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#).
    -   Data development: The data development helps you to design data computing processes according to your business demands and make mutually dependent tasks be automatically run in the scheduling system. For more information, see [Data Development Overview](../../../../../reseller.en-US/User Guide/Data development/Solution.md#).
    -   O&M center: The O&M Center is a place where tasks and instances are displayed and operated. You can view all your tasks in Task List and perform such operations on the displayed tasks.  For more information, see [O&M Center Overview](../../../../../reseller.en-US/User Guide/Operation center/Operation center overview.md#).
    -   Data management: The Data Management module of the Alibaba Cloud DTplus platform displays the global data view and metadata details of an organization, and enables operations such as divided permission management, data lifecycle management, and approval and management of data table/resource/function permissions.  For more information, see [data management overview](../../../../../reseller.en-US/User Guide/Data management/Introduction.md#).
3.   Configure the basic information and advanced settings for the new project.![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11944/15506529671418_en-US.png)

 

    **Basic Configuration:**

    -   project name: The length of the project name is between 3 and 27 characters.
    -   display name: The length of the display name is not more than 27 characters.
    -   Workspace mode: The new version of Dataworks workspace has introduced simple and standard modes. For a detailed description, see[Simple mode and standard mode](../../../../../reseller.en-US/Product Introduction/Simple mode and standard mode.md#).
    **Advanced Configuration:**

    -   Enable scheduling frequency: Control the current project whether to enable or disable the scheduling system, and if it is disabled, it can not periodically schedule tasks.
    -   Allow editing tasks and code in this project: if the permission of the current project member to create/edit code files in this project is disabled, you cannot create/edit a new/edited code file.
    -   Enable select result downloads in this project: Whether data results from select statement can be downloaded in this project, and if it is disabled, it cannot download the data query results from select statement.
    -   MaxCompute project name: A same project name of MaxCompute can be created in the process of creating the project.
    -   Maxcompute access identity: Personal account, system account. It is recommended that corporate users with high security requirements use their personal accounts.
    -   Quota group: Quota is used to implement disk quotas.
4.   Click Create, the project is displayed on the **project list**. 

    **Note:** 

    -   Once you become a project owner, it means that **everything** in the project is yours, and no one has permission to access your project before the authentication.
    -   For general users, it is not necessary to create a project. If you are added to a project, you can use the MaxCompute.

