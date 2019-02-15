# Visualization of operating the tables {#concept_ngz_dg2_vdb .concept}

The Project Explorer of MaxCompute Studio provides the visualized table structure editor used to create and modify tables.

## Visualization of creating a table {#section_nqb_c32_vdb .section}

**Procedure**

1.  Right-click the project that you want to create the table, and select **create a new table**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12123/15502205451707_en-US.png)

2.  In the dialog box that appears, enter a table name and column information. Click **Generate CreateTable  Statement**generates the corresponding pant statement, click **Execute** to execute the build table.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12123/15502205451708_en-US.png)

    When you set the table name, column name, type, and lifecycle, observe the related requirements of MaxCompute. For more information, see[Table Operations](../../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table Operations.md#).

3.  After the table is created, view the table metadata in **table&view** of the **Project Explorer**. If no metadata is displayed, refresh the list.


## Visualization of modifying a table {#section_rqb_c32_vdb .section}

**Procedure**

1.  In**table&view** of the **Project Explorer**, right-click the expected table and select **Open table  Editor**:

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12123/15502205461710_en-US.png)

2.  In the dialog box that appears, edit the table. You can modify the table comments, table lifecycle, column name, and column description, and add columns. Specific rules follow the MaxCompute table requirements and can be found in[Table Operations](../../../../../reseller.en-US/User Guide/SQL/DDL SQL/Table Operations.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12123/15502205461711_en-US.png)

3.  After completing the modifications, click **Alter Table Statement** Generate a specific alter statement and click **Execute**to perform the table modify operation. After successful execution, view the table metadata.


## Visualization of deleting a table {#section_vqb_c32_vdb .section}

In **table&view** of the**Project Explorer**, right-click the expected table and select **Drop table from server**:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12123/15502205461712_en-US.png)

Select **OK**in the bullet box to remove the table from the MaxCompute service.

