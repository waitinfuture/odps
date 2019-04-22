# Quick Start {#concept_186666 .concept}

This topic describes how to access MaxCompute Lightning services with major third-party tools, including how to view tables of a specified MaxCompute project, and how to perform BI analysis.

## Prerequisites {#section_3ec_5d5_ww0 .section}

-   Activate MaxCompute and create a project.

    Using MaxCompute Lightning requires that MaxCompute has been activated and a project has been created.

    If you have not activated MaxCompute, activate the service first. For more information, see [Activate MaxCompute](https://www.alibabacloud.com/help/doc-detail/58226.htm). Then, create a MaxCompute project.

-   Create a table and import data.

    Tables have been created in the project and data has been loaded. For more information, see [MaxCompute Quick Start](https://www.alibabacloud.com/help/doc-detail/27808.htm).

-   Obtain account information.

    The access ID and access key for the MaxCompute project have been obtained.

    You can log on to the Alibaba Cloud website, and click **Console** to view the AccessKey page. Contact the owner of the primary account if your RAM user is not granted permission to view AccessKey. You also need to ensure your RAM user is granted permission to view project tables.


## Prepare client tools for connection {#section_67t_sui_450 .section}

MaxCompute Lightning is compatible with PostgreSQL interfaces and is accessible to client tools that are connected to PostgreSQL databases.

[Tableau Desktop](https://www.tableau.com/products/desktop) BI tools are used in this tutorial. Download related tools from the Tableau official website.

Other commonly used client tools, such as SQL Workbench/J, PSQL, FineReport BI, and MicroStrategy BI tools, can be connected to MaxCompute Lightning in the same way as to PostgreSQL databases.

## Access services and perform analysis {#section_run_tgc_p6l .section}

After the connection service is successful, you can view the data table under the specified MaxCompute project for BI analysis.

1.  Select PostgreSQL when establishing a connection to a server.

    Start Tableau Desktop. In the left-side navigation pane, select **Select** \> **To Servers** \> **More** \> **PostgreSQL**.

2.  Enter service connection and user authentication information.

    |Parameter|Description|
    |:--------|:----------|
    |Server|Enter the MaxCompute Lightning endpoint of a specified region in the Server field. For example, enter the value `lightning.cn-shanghai.maxcompute.aliyun.com` as the endpoint for the China East 2 region.|
    | |443|
    |Database|MaxCompute project name|
    |ID Verification|User name and password|
    |Username/Password|User Access Key ID/Access Key Secret|
    |SSL connection|Select the **SSL connection** check box.|

3.  Obtain project table information and create a data source or model.

    After you configure the contact information and log on to the Tableau Desktop, this software loads tables of the connected MaxCompute project. You can choose tables to create data models and charts as required.

    The following figure shows an example of a chart created based on required dimensions and measures.


Now you have gained access to MaxCompute Lightning using Tableau Desktop. You can perform BI analysis on the data of the connected MaxCompute projects.

**Note:** For better performance, it is recommended that you customize the connection to the Lightning data source using the TDC file supported by Tableau. For more information, see [Tableau Desktop](reseller.en-US/User Guide/Interactive SQL (Lightning)/ Access services using JDBC interfaces/Access services using common tools.md#section_p1m_3yk_z2b).

