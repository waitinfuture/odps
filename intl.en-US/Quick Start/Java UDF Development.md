# Java UDF development {#concept_edn_3yy_5db .concept}

MaxCompute user-defined functions \(UDFs\) include User Defined Scalar Function \(UDF\), User Defined Aggregation Function \(UDAF\), and User Defined Table Valued Function \(UDTF\).  In general, all these functions are called UDF.

Users who use Maven can search odps-sdk-udf in the [Maven Library](http://search.maven.org/) to get the required Java SDK \(available in different versions\).

```
<dependency>
    <groupId>com.aliyun.odps</groupId>
    <artifactId>odps-sdk-udf</artifactId>
    <version>0.20.7</version>
</dependency>
```

Currently, JAVA UDF can be developed in the following ways:

-   Using [MaxCompute Studio completes Java UDF development throughout the process](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Develop and debug UDF.md).
-   Use the [develop and debug JAVA UDF with Eclipse plug-ins](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/UDF.md), export jar packages, then use the command or DataWorks to [add the resource](../../../../../reseller.en-US/User Guide/Common commands/Resources.md) and then [register the function](../../../../../reseller.en-US/User Guide/Common commands/Functions.md).

The code examples for UDF, UDAF, udtf are given separately in this article, and the complete process example for developing UDF in two methods will be displayed\(The steps of UDAF, UDTF are the same as that of UDF \).

**Note:** 

-   For related statements about custom function registration and logout, and viewing function list, see [function](../../../../../reseller.en-US/User Guide/Common commands/Functions.md).
-   For the data types mapping for Java and MaxCompute, see [parameters and return types](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md).

## UDF example {#section_q5g_1zy_5db .section}

The following example shows how to develop a UDF to realize character lowercase conversion.

-   **Developing using MaxCompute Studio**
    1.  **Prepare tools and environment**.

        Suppose that the environment has been prepared, which includes [installing Studio](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Tools Installation and version history/Install IntelliJ IDEA.md), creating a [MaxCompute project link](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Project space connection management.md) on Studio, and creating a [MaxCompute Java module](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Create MaxCompute Java Module.md).

    2.  **Write program**.

        Create a Java file under the configured Java Module.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15505704311573_en-US.png)

        Select MaxCompute Java directly and enter the `package name in name text box, file name`.  Select UDF for Kind. Edit the code as following:

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

        **Note:** If you want to debug Java locally UDF, see [develop and debug UDF](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Developing Java/Develop and debug UDF.md)

    3.  **Register MaxCompute UDF**.

        As shown in the following figure, right-click the UDF's Java file and select **Deploy to server**，select the MaxCompute project to be registered in the dialog box. enter a `function name`. The  resource name also can be modified.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15505704311574_en-US.png)

        When all configurations are finished, click **OK**. There are prompts after the registration is successful.

    4.  **Try the UDF**.

        Open the SQL script and execute the code such as `select Lower_test(‘ABC’);`. The result is shown in the following figure:

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/11953/15505704311575_en-US.png)

        **Note:** To write SQL scripts in Studio, see [writing SQL scripts](../../../../../reseller.en-US/Tools and Downloads/MaxCompute Studio/Develop SQL procedure/Write SQL.md).

-   **Developing using the Eclipse plug-in**
    1.  **Creating a project**

        Suppose that a MaxCompute \(formerly ODPS\) project has been created in the Eclipse plug-in, for more information, see [creating a MaxCompute project](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/Create a project.md).

    2.  **Writing program**

        To archive function, write a program and compile in terms of MaxCompute UDF frame. For examples:

        ```
        
        package org.alidata.odps.udf.examples;
        import com.aliyun.odps.udf.UDF;
        public final class Lower extends UDF {
        Public String evaluate (string s ){
        if (s == null) { return null; }
        return s.toLowerCase();
        }
        }
        ```

        Name the JAR package **my\_lower.jar**.

        **Note:** 

        -   For more detailed introduction of developing debugging code, see [UDF](../../../../../reseller.en-US/Tools and Downloads/Eclipse Plugins/UDF.md).
        -   For more information about the SDK, see [UDF SDK](../../../../../reseller.en-US/User Guide/SQL/UDF/UDF Summary.md) .
    3.  **Add Resource：**

        Specify the referenced UDF code before running UDF. Your code is added to MaxCompute in the form of resources.  Java  UDF must be compiled into the Jar package and added in MaxCompute as a Jar resource. The UDF framework loads the JAR package automatically and runs UDF.

        **Note:** MaxCompute [MapReduce](../../../../../reseller.en-US/User Guide/MapReduce/Summary/MapReduce.md) also describes the use of resources.

        Run the command:

        ```
        
        add jar my_lower.jar;
        -- If the resource name already exists, rename the JAR package.
        -- Pay attention to modifying related name of JAR package in following command.
        -- Alternatively, use –f option directly to overwrite original JAR resource.
        ```

    4.  **Register UDF:**

        The format of the command is as follows:

        ```
        CREATE FUNCTION <function_name> AS <package_to_class> USING <resource_list>;
        ```

        **Parameter description：**

        -   **The name of the function\_name**:UDF function is the name used in the function referenced by SQL.
        -   **Package\_to\_class**: In the case of Java UDF, the name is from the top-level package name to the fully qualified class name that implements the UDF class name. If it is Python UDF, the name is Python script name. Class name. And the name must be quoted.
        -   **resource\_list**:The list of resources used by UDF.
            -   This resource list must include the resources in which the UDF code is located.
            -   If your code reads resource files through the distributed cache interface, the list also includes a list of resource files read by UDF.
            -   Resource lists consist of multiple resource names separated by commas and must be quoted.
            -   If you need to specify the project where the resource is located, it is written as <project\_name\>/resources/<resource\_name\>.
        After the JAR package has been uploaded, MaxCompute can obtain a user’s code and run it. Note that, for the UDF to be usable, MaxCompute requires the user to register a unique function name in MaxCompute and specify which function is corresponding to this function name in the Jar resource.

        Next, run the command:

        ```
        CREATE FUNCTION test_lower AS 'org.alidata.odps.udf.examples.Lower' USING 'my_lower.jar';
        ```

        **Note:** 

        -   Like resource files, the same name function can only be registered once.
        -   In general, your self built function cannot overwrite the built in function of the system. Only the Owner of the project space has the right to cover the built in function. If you use a custom function that overrides the built-in function, warning information is printed in Summary after SQL execution.
    5.  Use this function in SQL:

        ```
        select test_lower('A') from my_test_table;
        ```


## UDAF example {#section_zvj_3bz_5db .section}

The registration method for a UDAF is similar to for a UDF. Its use is consistent with the [Aggregation Function](../../../../../reseller.en-US/User Guide/SQL/Builtin functions/Aggregate functions.md).  The following example shows a UDAF code to calculate the average:

```

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

The registration method and usage of a UDTF is similar to a UDF. The code example is as follows:

```

package org.alidata.odps.udtf.examples;
import com.aliyun.odps.udf.UDTF;
Import com. aliyun. ODPS. UDF. udtfcollector;
import com.aliyun.odps.udf.annotation.Resolve;
import com.aliyun.odps.udf.UDFException;
// TODO define input and output types, e.g., "string,string->string,bigint".
@ Resolve ({"string, bigint-> string, bigint "})
public class MyUDTF extends UDTF {
@Override
public void process(Object[] args) throws UDFException {
String A = (string) ARGs [0];
Long B = (long) ARGs [1];
For (string T: A. Split ("\ s + ")){
Forward (T, B );
}
}
}
```

MaxCompute provides many built-in functions to meet your computing needs, while you can also create custom functions. For more information, see [Create UDFs](https://www.alibabacloud.com/help/doc-detail/30270.html).

