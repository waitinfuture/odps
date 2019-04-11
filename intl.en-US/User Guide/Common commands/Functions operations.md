# Functions operations {#concept_hb1_wdf_vdb .concept}

This article explains how to use common commands to operate functions in the MaxCompute client. 

You can also operate functions using the visualized online data development tools in DataWorks. For more information, see [Function Management](../../../../../reseller.en-US/User Guide/Data management/Create a table.md#).

## Create a Function {#section_rrq_12f_vdb .section}

**Command format:**

```
CREATE FUNCTION <function_name> AS <package_to_class> USING <resource_list>;
```

**Parameters**

-   function\_name: An UDF name referenced in SQL.
-   package\_to\_class: For Java UDF, this name is a fully qualified class name \(from top-level package name to UDF class name\). This parameter must be in double quotation marks.  And, for Python UDF, this name is a python script name. classname. For both Java UDF and python script, use double quotation \(““\) marks to indicate this parameter. And for the name, use quotation marks.
-     resource\_list: Provides resource list used by UDF.
    -   Resources that contain UDF code must be included in the list.
    -   If the code reads the resource file by the distributed cache interface, this list also contains the list of resource files read by the UDF.
    -   The resource list is composed of multiple resource names, separated by a comma \(,\). The resource list must be in double quotation \(“”\) marks.
    -   Specify the project in which the resource is located as follows: `<project_name>/resources/<resource_name>`.

**Example:**

-   Suppose a Java UDF class org.alidata.odps.udf.examples.Lower is in my\_lower.jar, create function my\_lower as follows:

    ```
    CREATE FUNCTION my_lower AS 'org.alidata.odps.udf.examples.Lower' USING 'my_lower.jar';
    ```

-   Suppose a Python UDF MyLower is used in project pyudf\_test.py,create function my\_lower  as follows:

    ```
    create function my_lower as 'pyudf_test.MyLower' using 'test_project/resources/pyudf_test.py';
    ```

-   Suppose a Java UDF class com.aliyun.odps.examples.udf.UDTFResource is in udtfexample1.jar, and it depends on file resource file\_resource.txt and table resource table\_resource1,create function  test\_udtf as follows:

    ```
    create function test_udtf as 'com.aliyun.odps.examples.udf.UDTFResource' using 'udtfexample1.jar, file_resource.txt, table_resource1,test_archive.zip';
    ```


**Note:** 

-   Similar to the resource files, the UDF duplicate name can be registered only once.
-   Generally UDF cannot overwrite system built-in functions. Only the project owner has right to overwrite the built-in functions. If you are using a UDF which overwrites the built-in function, the warning is triggered in Summary after SQL execution.

## Drop a Function {#section_esm_m2f_vdb .section}

**Command format:**

```
DROP FUNCTION <function_name>;
```

**Example:**

```
DROP FUNCTION test_lower;
```

## List Functions {#section_o2r_42f_vdb .section}

**Command format:**

```
list functions; --View all user-defined functions in current project.
list functions -p my_project; --View all user-defined functions in the project 'my_project'.
```

