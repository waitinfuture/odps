# Use Eclipse to develop a Java-based UDF {#concept_zkf_m1v_cgb .concept}

This topic describes how to develop a Java-based user-defined function \(UDF\) by using the Eclipse-integrated ODPS plug-in.

**Note:** For more information, see [Java-based UDF](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

## Use Eclipse to create an ODPS project {#section_xxl_q1v_cgb .section}

Before creating an ODPS project, you must install the ODPS plug-in. For more information, see [Install](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Install.md#).

In Eclipse, select **File** \> **New** \> **ODPS Project**, enter the project name, and click **Config ODPS console installation path** to configure the installation path of the [odpscmd client](../../../../../reseller.en-US/Tools and Downloads/Client.md#).

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083534302_en-US.png)

Enter the **installation package** path and click **Apply**. The ODPS plug-in automatically parses the version of the odpscmd client.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634304_en-US.png)

Click **Finish**.

## Create a Java-based UDF in the ODPS project {#section_ngg_tcv_cgb .section}

On the **Package Explorer** pane, right-click the ODPS Java-based UDF project you have created, select **New**, and select **UDF**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634311_en-US.png)

Set the UDF package to com.aliyun.example.udf and name to Upper2Lower, and click **Finish**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634316_en-US.png)

An automatic Java code is generated after you create a UDF. Note that **do not change the name of the evaluate\(\) function**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634317_en-US.png)

## Implement the evaluate\(\) function contained in the UDF file {#section_umr_4gv_cgb .section}

Write the function code to be implemented into the evaluate\(\) function. Do not change the name of the evaluate\(\) function. The following is an example of how to convert uppercase letters to lowercase letters.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634318_en-US.png)

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

Save the code.

## Test the Java-based UDF code {#section_bfw_zgv_cgb .section}

Before testing the Java-based UDF code, store some uppercase letters on MaxCompute. Create a test table named upperABC using the `create table upperABC(upper string);` SQL statement on the odpscmd client.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634320_en-US.png)

Use the `insert into upperABC values('ALIYUN');` SQL statement to insert the string of uppercase letters 'ALIYUN'.

Select **Run** \> **Run Configurations** to set the test parameters.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634322_en-US.png)

Set the test parameters. Set **Project** to the name of the Java ODPS project you have created, and set **Select ODPS project** to the MaxCompute project name. Note that the project name needs to match the name of that connected to the odpscmd client. Next, set **Table** to upperABC. Last, after completing all the settings, click **Run**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634324_en-US.png)

You can view the test result in the **Console** pane, as shown in the following figure.

**Note:** Eclipse obtains the string of uppercase letters from the table and converts them to a string of lowercase letters, which is 'aliyun'. However, the uppercase letters stored on MaxCompute are not converted.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634326_en-US.png)

## Export the JAR package {#section_url_sqv_cgb .section}

You can use the Java-based UDF after the test is successful. Right-click the ODPS project you have created and select **Export**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634328_en-US.png)

On the displayed page, select **JAR file** and click **Next**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634329_en-US.png)

Enter the JAR package name and click **Finish**. Then, the JAR package is exported to your workspace.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634330_en-US.png)

## Upload the JAR package to DataWorks {#section_nps_qrv_cgb .section}

Log on to the DataWorks console, find the MaxCompute\_DOC workspace, and go to the [Data Studio](../../../../../reseller.en-US/User Guide/Data development/Console functions/Introduction to console.md#) page. Select **Business Flow** \> **Resource** \> **Create Resource** \> **JAR**, and create a [JAR resource](../../../../../reseller.en-US/User Guide/Data development/Business flow/Resource.md#ul_u5d_411_t2b).

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083634331_en-US.png)

On the displayed page, upload the JAR package you have exported.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083734334_en-US.png)

The JAR package is uploaded to DataWorks. To upload it to MaxCompute, click the JAR package and click **Submit and Unlock**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083734335_en-US.png)

You can run the list resources command on the odpscmd client to view the uploaded JAR package.

After uploading the JAR package to your MaxCompute workspace, select **Business Flow** \> **Function** \> **Create Function**, and create a [function](../../../../../reseller.en-US/User Guide/Data development/Business flow/Register the UDFs.md#) named upperlower\_Java. This function corresponds to the JAR package. Set **Resources** to the name of the uploaded JAR package. After completing these settings, click **Save** and **Submit and Unlock**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083734337_en-US.png)

You can run the list functions command on the odpscmd client to view the registered function. Then, the upperlower\_Java Java-based UDF registered using Eclipse can be used.

## Check the Java-based UDF {#section_pkt_mvv_cgb .section}

In the odpscmd CLI, run the `select upperlower_Java('ABCD') from dual;` command. This function can convert a string of uppercase letters to lowercase letters, showing that the function works properly.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/79958/154823083734338_en-US.png)

To use IntelliJ IDEA to develop a Java-based UDF, see [Use IntelliJ IDEA to develop a Java-based UDF](reseller.en-US/Best Practices/Data development/Use IntelliJ IDEA to develop a Java-based UDF.md#).

