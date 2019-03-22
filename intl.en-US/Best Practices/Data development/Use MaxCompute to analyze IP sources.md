# Use MaxCompute to analyze IP sources {#concept_q25_ygl_5fb .concept}

This topic describes how to use MaxCompute to analyze IP sources. The procedure includes downloading and uploading data from an IP address library, writing a user-defined function \(UDF\), and writing a SQL statement.

## Background {#section_rfz_jz1_pgb .section}

The query APIs of [Taobao IP address library](http://ip.taobao.com/) are [IP address strings](http://ip.taobao.com/service/getIpInfo.php?ip=[ip%E5%9C%B0%E5%9D%80%E5%AD%97%E4%B8%B2]). The following is an example.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206731905_en-US.png)

HTTP requests are not directly allowed in MaxCompute. However, you can query IP addresses in MaxCompute using one of the following methods:

1.  Run a SQL statement and then initiate an HTTP request. This method is inefficient. The request will be rejected if the query frequency is lower than 10 QPS.
2.  Download the IP address library to the local server. This method is inefficient and will affect the data analysis in data warehouses.
3.  Maintain the IP address library regularly and upload it to MaxCompute. This method is relatively effective. However, you need to maintain the IP address library regularly.

The following further describes the third method.

## Download an IP address library {#section_xwc_y3l_5fb .section}

1.  You need to obtain data from an IP address library. This section provides a [demo of an incomplete UTF-8 IP address library](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/attach/102762/cn_zh/1547530733280/ipdata.txt.utf8).
2.  Download the **UTF-8** IP address library and check the data format, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206731907_en-US.png)

    The first four strings of data are the starting and ending IP addresses, among which the first two are decimal integers and the second two are expressed in dot-decimal notation. The decimal integer format is used to check whether an IP address belongs to the target network segment.


## Upload data from the IP address library {#section_eqh_pjl_5fb .section}

1.  Create a table data definition language \(DDL\) on the [MaxCompute client](../../../../../reseller.en-US/Tools and Downloads/Client.md#), or [create a table on the GUI](../../../../../reseller.en-US/User Guide/Data development/Table Management.md#) in DataWorks.

    ```
    DROP TABLE IF EXISTS ipresource ;
    
    CREATE TABLE IF NOT EXISTS ipresource 
    (
        start_ip BIGINT
        ,end_ip BIGINT
        ,start_ip_arg string
        ,end_ip_arg string
        ,country STRING
        ,area STRING
        ,city STRING
        ,county STRING
        ,isp STRING
    );
    ```

2.  Run the [Tunnel commands](../../../../../reseller.en-US/User Guide/Data upload and download/Tunnel commands.md#) to upload the ipdata.txt.utf8 file, which is stored on the D drive.

    ```
    odps@ workshop_demo>tunnel upload D:/ipdata.txt.utf8 ipresource;
    ```

    You can use the `select count(*) from ipresource;` SQL statement to view the uploaded data. Generally, the quantity of data increases in the library due to regular updates and maintenance.

3.  Use the `select * from ipresource limit 10;` SQL statement to view the first 10 pieces of data in the ipresource table, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206731909_en-US.png)


## Write a UDF {#section_uht_3kl_5fb .section}

1.  Choose **Data Studio** \> **Business Flow** \> **Resource**. Right-click **Resource** and choose **Create Resource** \> **Python**. In the displayed dialog box, enter the name of the Python resource, select **Upload to ODPS** and click **OK**, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206731910_en-US.png)

2.  Write code for the Python resource. The following is an example:

    ```
    from odps.udf import annotate
    @annotate("string->bigint")
    class ipint(object):
    	def evaluate(self, ip):
    		try:
    			return reduce(lambda x, y: (x << 8) + y, map(int, ip.split('.')))
    		except:
    			return 0
    ```

    Click **Submit and Unlock**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206731911_en-US.png)

3.  Choose **Data Studio** \> **Business Flow** \> **Function**. Right-click **Function** and select **Create Function**.

    Set the function class name to `ipint.ipint`, and the folder to the resource name, and click **Submit and Unlock**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206831913_en-US.png)

4.  Create an ODPS SQL node and run the SQL statement to check whether the ipint function works as expected. The following is an example.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206831914_en-US.png)


You can also create a local ipint.py file and use the [MaxCompute client](../../../../../reseller.en-US/Tools and Downloads/Client.md#) to upload the resource.

```

odps@ MaxCompute_DOC>add py D:/ipint.py;
OK: Resource 'ipint.py' have been created.

```

After uploading the resource, use the client to [register the function](../../../../../reseller.en-US/User Guide/Common commands/Functions operations.md#).

```

odps@ MaxCompute_DOC>create function ipint as ipint.ipint using ipint.py;
Success: Function 'ipint' have been created.

```

The function can be used after registration. You can use `select ipint('1.2.24.2');` on the client to test the function.

**Note:** You can perform [cross-project authorization](../../../../../reseller.en-US/Security/Configure security features/Resource share across project space/Resource sharing across projects based on package.md#) to share the UDF with other projects under the same Alibaba Cloud account.

1.  Create a package named ipint.

    ```
    odps@ MaxCompute_DOC>create package ipint;
    OK
    ```

2.  Add the UDF to the package.

    ```
    odps@ MaxCompute_DOC>add function ipint to package ipint;
    OK
    ```

3.  Allow a bigdata\_DOC project to install the package.

    ```
    odps@ MaxCompute_DOC> allow project bigdata_DOC to install package ipint;
    OK
    ```

4.  Switch to a bigdata\_DOC project that needs to use the UDF and install the package.

    ```
    odps@ MaxCompute_DOC>use bigdata_DOC;
    odps@ bigdata_DOC>install package MaxCompute_DOC.ipint;
    OK
    ```

5.  Then, the UDF can be used. If a user \(such as Bob\) of the bigdata\_DOC project wants to access the resource, the administrator can grant the access permission to the user by using the ACL.

    ```
    odps@ bigdata_DOC>grant Read on package MaxCompute_DOC.ipint to user aliyun$bob@aliyun.com; --Use the ACL to grant the package access permission to Bob.
    ```


## Use the IP address library in SQL {#section_rmm_bml_5fb .section}

**Note:** This section uses the IP address 1.2.254.2 as an example. You can use a specific field to query an IP address as needed.

You can use the following SQL code to view the test result:

```
select * from ipresource
WHERE ipint('1.2.24.2') >= start_ip
AND ipint('1.2.24.2') <= end_ip
```

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/63437/155323206831915_en-US.png)

To ensure the data accuracy, you can regularly obtain data from the Taobao IP address library to maintain the ipresource table.

