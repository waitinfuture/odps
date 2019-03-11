# Project operations {#concept_qg3_s32_vdb .concept}

This article introduces you to command operations for entering the project and setting space properties \( permissions and whitelist functions, etc. \).

## Enter the workspce {#section_mv5_sj2_vdb .section}

Command format:

```
use <project_name>;
```

Action:

-   Enter the specified workspce.  After entering the workspce, all objects in this workspce can be operated by the user.
-   If the workspce does not exist or the current user is not in this workspce, an exception is returned.

Example:

```
odps:my_project>use my_project; --my_project is a workspce the user has privilege to access.
```

**Note:** 

The preceding examples uses the MaxCompute client.  All  MaxCompute command keywords, workspce names, table names, column names are case insensitive.

Creating a project is creating a MaxCompute project

After running the command, you can access the objects of this workspce. In the following example, assume that test\_src exists in the project ‘my\_project’. Run the following command:

```
odps:my_project>select * from test_src;
```

MaxCompute automatically searches the table in my\_project. If the table exists, it returns the data of this table. If the table does not exist, an exception is thrown. To access the table test\_src in another workspce, such as ‘my\_project2’, through the project ‘my\_project’, you must first specify the workspce name as follows:

```
odps:my_project>select * from my_project2.test_src;
```

The returned data is the data in my\_project2, not the initial data of test\_src in my\_project.

MaxCompute does not support commands to create or delete workspce. You can use the MaxCompute console for additional configurations and operations as needed.For details, see [project list](../../../../../intl.en-US/User Guide/Workbench/Workspace list.md#)

## Query workspace {#section_znx_vxc_bhb .section}

Command format:

```
list projects;
```

Features:

Used to view the list of items created by the primary account.

**Note:** This command is supported from odpscmd 0.30.2.

## SetProject {#section_pyt_mff_vdb .section}

**Command format:**

```
setproject [<KEY>=<VALUE>];
```

**Action:**

-   Use setproject command to set workspce attributes.

    The following example sets the method that allows a full table scan.

    ```
    setproject odps.sql.allow.fullscan = true;
    ```

-   If the value of <KEY\>=<VALUE\> is not specified, the current workspce attribute configuration is displayed. **Command format:**

    ```
    setproject;  --Display the parameters set by the setproject command.
    ```


**Parameters**

|Property name|Configured permission|Description|Value range|
|:------------|:--------------------|:----------|:----------|
|odps.sql.allow.fullscan|ProjectOwner|Determines whether to allow a full table scan.|True \(permitted\) /false \(prohibited\)|
|odps.table.drop.ignorenonexistent|All users|Whether to report an error when deleting a table that does not exist. When the value is true, no error is reported.|True \(no error reported\)/false|
|odps.security.ip.whitelist|ProjectOwner|Specify an IP whitelist to access the workspce.|IP list separated by commas \(,\)|
|odps.instance.remain.days|ProjectOwner|Determines the duration of the retention of the instance information.|\[3- 30\]|
|READ\_TABLE\_MAX\_ROW|ProjectOwner|The number of data entries returned by running the Select statement in the client.|\[1-10000\]|

## Examples for odps.security.ip.whitelist {#section_nky_ycz_5fb .section}

MaxCompute supports a workspce level IP whitelist.

**Note:** 

-   If the IP whitelist is configured, only the IP \(console IP or IP of exit where SDK is located\) in the whitelist can access this workspce.
-   After setting the IP white list, wait for at least five minutes to let the changes take effect.
-   For further assistance, open a ticket to contact Alibaba Cloud technical support team.

The following are the three formats for an IP list in the whitelist, which can appear in the same command. Use commas \(,\) to separate these commands.

-   IP address: For example, 101.132.236.134.
-   Subnet mask: For example, 100.116.0.0/16.
-   Network segment: For example, 101.132.236.134-101.132.236.144.

Example of the command line tool set the IP white list:

```
setproject odps.security.ip.whitelist=101.132.236.134,100.116.0.0/16,101.132.236.134-101.132.236.144;
```

If no IP address is added in the whitelist, then the whitelist function is disabled.

```
setproject odps.security.ip.whitelist=;
```

