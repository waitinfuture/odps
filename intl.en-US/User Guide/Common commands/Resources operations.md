# Resources operations {#concept_pps_h1f_vdb .concept}

This article explains how to use common commands to operate resources in the MaxCompute client.

You can also search and upload resources using the visualized online data development tools in DataWorks. For more information, see [Resource management](../../../../../reseller.en-US/User Guide/Data development/Manual business flow/Resource.md#).

## Add a resource {#section_lqp_41f_vdb .section}

**Command format:**

```
add file <local_file> [as alias] [comment 'cmt'][-f];
add archive <local_file> [as alias] [comment 'cmt'][-f];
add table <table_name> [partition <(spec)>] [as alias] [comment 'cmt'][-f];
add jar <local_file.jar> [comment 'cmt'][-f];
```

**Parameters**

-   file/archive/table/jar: Indicates the resource type. For more information, see [Resources](../../../../../reseller.en-US/User Guide/Definitions/Resource.md).
-   local\_file: Indicates path of the local file, and uses this file name as the resource name. Resource name also acts as a unique identifier of a resource.
-   table\_name: Indicates table name in MaxCompute. Currently, external tables cannot be added into resource.
-   \[PARTITION  \(spec\)\]: When the resource to be added is a partition table, MaxCompute only supports taking a partition as a resource, not the entire partition table.
-   alias: Specifies a resource name. If this parameter is not specified, the file name is used as a resource name by default. Jar and Python resources do not support this function.
-   \[comment ‘cmt’\]: Adds a comment for the resource.
-   \[-f\]: If a name is duplicated, this parameter can be added as a substitute to the original resource. If this parameter is not specified and the duplicate resource name exists, the operation fails.

**Example**

```
odps@ odps_public_dev>add table sale_detail partition (ds='20150602') as sale.res comment 'sale detail on 20150602' -f;
OK: Resource 'sale.res' have been updated.
---Add a resource named sale.res in MaxCompute.
```

**Note:** Each resource file size cannot exceed 500 MB. The resource size referenced by a single SQL or MapReduce task cannot exceed 2048 MB. For more information about, see [MR Restrictions](reseller.en-US/User Guide/MapReduce/MR limits.md#) .

## Delete a resource {#section_nbx_kbf_vdb .section}

**Command format:**

```
DROP RESOURCE <resource_name>; --resource_name：a specified resource name.
```

## View the resource list {#section_t5b_mbf_vdb .section}

****Command format**:**

```
LIST RESOURCES; 
```

**Action: **

View all resources in the current project.

**Example:**

```
odps@ $project_name>list resources;
Resource Name Comment Last Modified Time Type
1234.txt 2014-02-27 07:07:56 file
mapred.jar 2014-02-27 07:07:57 jar
```

## Download resources {#section_il1_rbf_vdb .section}

****Use the following command format to download resources:****

```
GET RESOURCE <resource_name> <path>;
```

**Action:**

Download resources to your local device. The resource type must be file, jar, archive, or py.

**Example:**

```
odps@ $project_name>get resource odps-udf-examples.jar d:\;
OK
```

