# Use IntelliJ IDEA to develop a Java-based UDF {#task_lph_3mw_dgb .task}

This topic describes how to use IntelliJ IDEA to develop a Java-based user-defined function \(UDF\). IntelliJ IDEA is an integrated development environment that can be used for developing Java programs.

1.  Prepare the IntelliJ IDEA development tool. For more information, see [Install IntelliJ IDEA and configure MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md).
2.  [Connect to a MaxCompute project](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md) through IntelliJ IDEA MaxCompute Studio.
3.  Make sure that you have successfully connected to the MaxCompute project. Once completed, [create a MaxCompute Java module](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md).

You can develop a Java-based UDF after your development environment has been prepared. The following is an example of how to develop a UDF for converting uppercase letters to lowercase letters.

**Note:** For more information about Java-based UDF development, see [Java-based UDF](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

1.  **Create a Java-based UDF project.** 

    In the IntelliJ IDEA, right-click the MaxCompute Java module directory, choose **src** \> **main** \> **java** \> **New** and click **MaxCompute Java**, as shown in the following figure.

    Set **Name** to `package name.file name`, select UDF for **Kind**, and click **OK**, as shown in the following figure.

    **Note:** 

    -   Name: The name of the MaxCompute Java class you have created. If you have not created a package, you can enter packagename.classname and a package will be automatically generated.
    -   Kind: The project type. The project types that are supported are user-defined functions \(such as UDF, UDAF, and UDTF\), MapReduce \(such as Driver, Mapper, and Reducer\), and non-structural development frameworks \(such as Storage Handler and Extractor\).
2.  **Edit the Java-based UDF code.** You can edit the porjectLower code in the Java-based UDF project you have created, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134458_en-US.png)

    The following is example code:

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

    **Note:** You can customize the code template in your IntelliJ IDEA as needed. The method is as follows: Choose **Settings** \> **Editor** \> **File Code Templates** and modify the target template in MaxCompute on the code tab page.

3.  **Test the UDF.** After developing the Java-based UDF, you can test it through unit testing \(UT\) or local running. The procedure is as follows:
    1.  **Unit testing** Your module project contains several UT examples under the examples directory. You can follow these examples when you perform UT.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134470_en-US.png)

        The following figure shows the test result.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134473_en-US.png)

        As you can see from the preceding figure, the string of uppercase letters 'ALIYUN' was converted to the lowercase letter string of 'aliyun'.

    2.  **Local running** To run the Java-based UDF in your IntelliJ IDEA, you need to specify and test the data source by using either of the following methods:

        -   Use MaxCompute Studio and Tunnel to downlaod table data from the specified project and save the data to your warehouse directory.
        -   Use the UDF project to provide the mock project and the table data. Then, you can customize the data source according to example\_project in the warehouse directory.
        Procedure

        1.  Before testing the Java-based UDF code, store some uppercase letters on MaxCompute. Create a test table named upperABC by using a script file or the `create table upperABC(upper string);` SQL statement on the odpscmd client, as shown in the following figure.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134592_en-US.png)

        2.  Right-click the UDF class and select **Run 'class name.main\(\)'**. The run configurations dialog box is displayed, as shown in the following figure.

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134483_en-US.png)

            **Note:** 

            -   UDF, UDAF, and UDTF are generally used to run SELECT SQL statements for specific columns. As a result, you need to configure the MaxCompute project, table, and columns before using these functions. Note that the metadata is obtained from the mock project under the project explorer and warehouse. Debugging for complex types is also supported.
            -   If the table data under the specified project is not downloaded and saved to your warehouse, then you need to download the data first before continuing. 100 pieces of data are downloaded automatically. If you require more data, you can configure the download record limit.
            -   The framework for UDF local running uses data in specified columns of the warehouse as input and runs the function locally. You can view the log output and results in the console.
            -   If you are using the mock project or if your data is already downloaded, you can run the UDF directly.
        Click **OK**. The following figure shows the result.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134510_en-US.png)

4.  **Publish the UDF.** After the Lower.java test is successful, pack this file into a JAR package and upload the package to MaxCompute. To publish a UDF to a server, you need to pack this file, add a resource, and register the function. IntelliJ IDEA MaxCompute Studio makes publishing a UDF easier in that this program can help you to run a maven clean package command, upload the JAR package, and register the UDF in sequence. The procedure to do so is as follows: Right-click the Java file of the UDF and select **Deploy to server**. In the displayed dialog box, select the target MaxCompute project and enter the Function name and the Resource name. You can change the resource name as needed. The following is an example.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134564_en-US.png)

    **Note:** 

    For more information about how to pack, upload, and register a UDF, see [Pack, upload, and register](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Package、Upload and Register.md#).

    After completing the settings, click **OK**. You can find the registered function in the connected MaxCompute project, as shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134572_en-US.png)

5.  **Use the UDF.** You can use the UDF after it is successfully registered. In the module project, open the SQL script and run the `select Lower_test('ALIYUN');` command. The result is shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134603_en-US.png)

    You can also run the select Lower\_test\('ALIYUN'\) from uppperABC;command on the odpscmd client to test the Java-based UDF. If the following information is displayed, the Lower\_test Java-based UDF developed by using IntelliJ IDEA works properly.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/155056972134582_en-US.png)


To use Eclipse to develop a Java-based UDF, see [Use Eclipse to develop a Java-based UDF](reseller.en-US/Best Practices/Data development/Use Eclipse to develop a Java-based UDF.md#).

