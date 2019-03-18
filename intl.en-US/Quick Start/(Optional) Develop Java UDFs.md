# \(Optional\) Develop Java UDFs {#concept_edn_3yy_5db .concept}

MaxCompute user-defined functions \(UDFs\) include User Defined Scalar Functions \(UDFs\), User Defined Aggregation Functions \(UDAFs\), and User Defined Table Valued Functions \(UDTFs\).

**Note:** MaxCompute currently supports [JAVA UDFs](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#), [Python UDFs](../../../../../reseller.en-US/User Guide/SQL/UDF/Python UDF.md#), [UDJs](../../../../../reseller.en-US/User Guide/SQL/UDJ.md#), and [UDTs](../../../../../reseller.en-US/User Guide/SQL/UDT.md#). For more information, see [JAVA UDFs](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

Users who use Maven can search odps-sdk-udf at [Maven library](http://search.maven.org/) to download the preferred Java SDK \(available in different versions\). The configuration information is as follows:

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
    <version>0.20.7</version>
</dependency>
```

You can use the following methods to develop Java UDFs:

-   Use [MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Develop and debug UDF.md).
-   Use the [Eclipse plug-in to develop and debug Java UDFs](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/UDF.md), export .jar packages, use commands or DataWorks to [add resources](../../../../../reseller.en-US/User Guide/Common commands/Resources operations.md), and then [register the functions](../../../../../reseller.en-US/User Guide/Common commands/Functions operations.md).

This topic provide code examples for UDFs, UDAFs, and UDTFs and the whole development procedures by using the two methods.

**Note:** 

-   For more information about statements for custom function registration and logout and function list display, see [Functions operations](../../../../../reseller.en-US/User Guide/Common commands/Functions operations.md).
-   For more information about the data type mapping between Java and MaxCompute, see [Parameters and return value types](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md).

## UDF example {#section_q5g_1zy_5db .section}

The following example shows how to develop a UDF to realize character lowercase conversion.

-   **Use MaxCompute Studio to develop a UDF.**
    1.  **Prepare the development environment and create a Java module.**

        Environment preparations include [installing MaxCompute Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md), [creating a MaxCompute project link](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md) on MaxCompute Studio, and [creating a MaxCompute Java module](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md).

    2.  **Compile code.**

        Create a Java file under the configured Java module.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15528708701573_en-US.png)

        As shown in the preceding figure, **java** \> **** **MaxCompute Java**. In the displayed dialog box, enter a `package name. file name` and set **Kind** to **UDF**. Then, compile the following code:

        ```
        package <package name>;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
        Public String evaluate (string s ){
         if (s == null) { return null; }
         return s.toLowerCase();
        }
        }
        ```

        **Note:** If you want to debug the Java UDF on your local PC, perform the operations mentioned in [Develop and debug UDF](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Develop and debug UDF.md).

    3.  **Register a MaxCompute UDF.**

        Right-click the UDF Java file and click **Deploy to server**. In the displayed dialog box, select the MaxCompute project to be registered for the **MaxCompute project** field, enter a **function name**. Then, click **OK**.

        **Note:** You can modify **Resource name** if you want to.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15528708701574_en-US.png)

    4.  **Test the UDF.**

        Open the SQL script and execute the code, for example, `select Lower_test(‘ABC’);`. The execution result is shown in the following figure:

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15528708701575_en-US.png)

        **Note:** For more information about compile an SQL script in MaxCompute Studio, see [Write SQL](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Develop SQL procedure/Write SQL.md).

-   **Use the Eclipse plug-in to develop a UDF.**
    1.  **Creating a project.**

        Fro more information, see [Create a project](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Create a project.md).

    2.  **Compile code.**

        Compile the following code to achieve UDF functions according to the MaxCompute UDF frame specifications:

        ```
        package <package name>;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
        public String evaluate(String s) {
        if (s == null) { return null; }
        return s.toLowerCase();
        }
        }
        ```

        Name the .jar package **my\_lower.jar**.

        **Note:** 

        -   For more information about development and debugging code, see [UDF](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/UDF.md).
        -   For more information about the SDK usage, see [UDF Summary](../../../../../reseller.en-US/User Guide/SQL/UDF/UDF Summary.md).
    3.  **Add resources.**

        Specify the referenced UDF code before running the UDF. The code is added to MaxCompute in the form of resources. The Java UDF must be compiled into the a .jar package and added in MaxCompute as a JAR resource. The UDF framework then automatically loads the .jar package and runs the custom UDF.

        **Note:** MaxCompute MapReuce also defines "resource" in this way. For more information, see [MapReduce](../../../../../reseller.en-US/User Guide/MapReduce/Summary/MapReduce.md).

        Run the following command:

        ```
        
        add jar my_lower.jar;
        -- If the resource name already exists, rename the JAR package.
        -- Pay attention to modifying related name of JAR package in following command.
        -- Alternatively, use –f option directly to overwrite original JAR resource.
        ```

    4.  **Register a UDF.**

        Run a command similar to the following:

        ```
        CREATE FUNCTION <function_name> AS <package_to_class> USING <resource_list>;
        ```

        **Parameter description:**

        -   **function\_name**: The UDF name, which is also referenced and used in SQL.
        -   **package\_to\_class**: In the case of a Java UDF, the name contains the top-level package name and the fully qualified class name that implements the UDF class name. In case of a Python UDF, the name is the "Python script name.class name". The name must be quoted.
        -   **resource\_list**: The list of resources used by the UDF.
            -   The resource list must include the resources in which the UDF code is located.
            -   If your code reads resource files through the distributed cache interface, the list must include a list of resource files read by the UDF.
            -   The name of the resource list consists of multiple resource names, which are separated by commas \(,\) and must be quoted.
            -   If you need to specify the project to which the resource belongs, the command is `<project_name>/resources/<resource_name>`.
        After the .jar package is uploaded, MaxCompute can automatically obtain the code and run it. However, the UDF still cannot be used because MaxCompute lacks the UDF information. In this case, you need to register a unique UDF name in MaxCompute, and specify the mapping between the UDF name and the type of the target JAR resource.

        Run the following command:

        ```
        CREATE FUNCTION test_lower AS 'org.alidata.odps.udf.examples.Lower' USING 'my_lower.jar';
        ```

        **Note:** 

        -   Each registered UDF name is unique, being similar to resource file names.
        -   Usually, only the owner of the project space can override a built-in UDF. However, if you use a custom UDF to override the built-in UDF, warning information is printed in the command summary after the SQL statement.
    5.  In SQL, verify the UDF by running the following command:

        ```
        select test_lower('A') from my_test_table;
        ```


## UDAF example {#section_zvj_3bz_5db .section}

The registration method for a UDAF is similar to that for a UDF. You can use a UDAF by following the instructions provided in [Aggregate functions](../../../../../reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md). The following shows an example of the UDAF code for calculating the average value:

```language-java
package org.alidata.odps.udf.examples;
import com.aliyun.odps.io.LongWritable;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.io.Writable;
import com.aliyun.odps.udf.Aggregator;
import com.aliyun.odps.udf.UDFException;
/**
 * project: example_project
 * table: wc_in2
 * partitions: p2=1,p1=2
 * columns: colc,colb,cola
 */
public class UDAFExample extends Aggregator {
  @Override
  public void iterate(Writable arg0, Writable[] arg1) throws UDFException {
    LongWritable result = (LongWritable) arg0;
    for (Writable item : arg1) {
      Text txt = (Text) item;
      result.set(result.get() + txt.getLength());
    }
  }
  @Override
  public void merge(Writable arg0, Writable arg1) throws UDFException {
    LongWritable result = (LongWritable) arg0;
    LongWritable partial = (LongWritable) arg1;
    result.set(result.get() + partial.get());
  }
  @Override
  public Writable newBuffer() {
    return new LongWritable(0L);
  }
  @Override
  public Writable terminate(Writable arg0) throws UDFException {
    return arg0;
  }
}
```

## UDTF example {#section_a1j_lbz_5db .section}

The registration and use methods of a UDTF is similar to those of a UDF. The following is an code example:

```
package org.alidata.odps.udtf.examples;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.UDTFCollector;
import com.aliyun.odps.udf.annotation.Resolve;
import com.aliyun.odps.udf.UDFException;
// TODO define input and output types, e.g., "string,string->string,bigint".
@Resolve({"string,bigint->string,bigint"})
public class MyUDTF extends UDTF {
  @Override
  public void process(Object[] args) throws UDFException {
    String a = (String) args[0];
    Long b = (Long) args[1];
    for (String t: a.split("\\s+")) {
      forward(t, b);
    }
  }
}
```

