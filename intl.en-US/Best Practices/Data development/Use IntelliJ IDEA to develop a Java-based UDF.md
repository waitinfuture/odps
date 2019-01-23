# Use IntelliJ IDEA to develop a Java-based UDF {#task_lph_3mw_dgb .task}

IntelliJ IDEA is an integrated development environment for the Java language that helps us quickly develop this Java program. This article details how to use the IntelliJ IDEA for Java UDF development.

Before starting the UDF development practice, you first need to prepare the IntelliJ IDEA development tools, see

[Install IntelliJ IDEA and configure MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md), then [Connect to a MaxCompute project](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md) in IntelliJ IDEA MaxCompute Studio, and [Create a MaxCompute Java module](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md). UDF can be developed after the development environment is ready,The following example shows how to develop a UDF to convert uppercase letters to lowercase letters.

**Note:** For more information about UDF development, see [Java-based UDF](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

1.  In the IntelliJ IDEA, right-click the MaxCompute Java module directory, select **src** \> **main** \> **java** \> **New**, and click **MaxCompute Java**, as shown in the following figure. 
2.  Set **Name** to `package name.file name`, select UDF for **Kind**, and click **OK**. 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967734451_en-US.png)

    **Note:** 

    -   Name: name of the MaxCompute Java class. If you have not created a package, you can enter packagename.classname to create a package automatically.
    -   Kind: project type. Supported types include user-defined functions \(for example, UDF, UDAF, and UDTF\), MapReduce \(for example, Driver, Mapper, and Reducer\), and non-structural development frameworks \(for example, Storage Handler and Extractor\).
3.  Edit the code, as shown in the following figure. 

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967734458_en-US.png)

    The following is an example:

    ```
    package <package name>;
    import com.aliyun.odps.udf.UDF;
    public class Lower extends UDF {
       public String evaluate(String s) {
          if (s == null) { 
              return null; 
          }
          return s.toLowerCase();
       }
    }
    ```

    **Note:** You can customize the code template in your IntelliJ IDEA as needed. The method is as follows: Select **Settings** \> **Editor** \> **File Code Templates** and modify the target template in MaxCompute on the code tab page.

4.  Debug the UDF. After developing the UDF, you can test it through unit testing \(UT\) or local running.
    1.  Unit testing There are various UT examples in the examples directory in your module project. You can perform the UT according to these examples.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967734470_en-US.png)

        The following figure shows the test result.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834473_en-US.png)

        As shown in the preceding figure, a string of uppercase letters 'ALIYUN' is converted to a string of lowercase letters 'aliyun'.

    2.  Local running To run the UDF in your IntelliJ IDEA, you need to specify and test the data source using either of the following methods:

        -   Use MaxCompute Studio and Tunnel to automatically download table data from the specified project and save the data to the warehouse directory.
        -   Use the UDF project to provide the mock project and the table data. Then, you can customize the data source according to example\_project in the warehouse directory.
        Procedure

        1.  Before testing the Java-based UDF code, store some uppercase letters on MaxCompute. Create a test table named upperABC by using a script file or the `create table upperABC(upper string);` SQL statement on the odpscmd client, as shown in the following figure.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834592_en-US.png)

        2.  Right-click the UDF class and select **Run 'class name.main\(\)'**. The run configurations dialog box is displayed, as shown in the following figure.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834483_en-US.png)

            **Note:** 

            -   The UDF, UDAF, and UDTF are generally used to run the SELECT SQL statement for specific columns. Therefore, you need to configure the MaxCompute project, table, and columns. \(The metadata is obtained from the mock project under the project explorer and warehouse.\) Debugging for complex types is also supported.
            -   If the table data under the specified project is not downloaded and saved to the warehouse, you need to download the data first. By default, 100 pieces of data are downloaded. If more data is required, you can configure the download record limit item.
            -   The UDF local running framework uses data in specified columns in the warehouse as the UDF input and runs the UDF locally. You can view the log output and result in the console.
            -   You can run the UDF directly if the mock project is used or if the data is downloaded.
        Click **OK**. The following figure shows the result.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834510_en-US.png)

5.  Publish the UDF. After the Lower.java test is successful, pack this file into a JAR package and upload the package to MaxCompute. To publish a UDF to a server, you need to **pack this file** \> **add a resource** \> **register the function**. IntelliJ IDEA MaxCompute Studio allows you to publish a UDF easily. That is, MaxCompute Studio automatically runs the maven clean package command, uploads the JAR package, and registers the UDF in sequence. The procedure is as follows: Right-click the Java file of the UDF and select **Deploy to server**. In the displayed dialog box, select the target MaxCompute project and enter the Function name and the Resource name. You can change the resource name as needed, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834564_en-US.png)

    **Note:** 

    For more information about how to pack, upload, and register a UDF, see [Pack, upload, and register](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package、Upload and Register.md#).

    After completing the settings, click **OK**. You can find the registered function in the connected MaxCompute project, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834572_en-US.png)

6.  Use the UDF. You can use the UDF after it is successfully registered. In the module project, open the SQL script and run the `select Lower_test('ALIYUN');` command. The result is shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834603_en-US.png)

    You can also run the select Lower\_test\('ALIYUN'\) from uppperABC; command on the odpscmd client to test the Java-based UDF. If the following information is displayed, the Lower\_test Java-based UDF developed by using IntelliJ IDEA works properly.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154822967834582_en-US.png)


To use Eclipse to develop a Java-based UDF, see [Use Eclipse to develop a Java-based UDF](reseller.en-US/Best Practices/Data development/Use Eclipse to develop a Java-based UDF.md#).

