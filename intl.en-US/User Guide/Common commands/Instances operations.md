# Instances operations {#concept_bvs_hm2_vdb .concept}

## Show instances/Show P {#section_owc_pv2_vdb .section}

**Command format:**

```
SHOW INSTANCES [FROM startdate TO enddate] [number];
SHOW P [FROM startdate TO enddate] [number];
SHOW INSTANCES [-all];
SHOW P [-all];
SHOW P -p <project name>;
```

**Action:**

Displays the information about the instances created by the current users.

**Parameters:**

-   startdate、enddate: Returns the instance information during the specified period \(from startdate to enddate\) in the yyyy-mm-dd format and the unit is ‘day’. The parameters are optional. If the parameters are not specified, instances submitted within three days are returned by default.
-   number: Specifies the number of instances to be displayed. Based on the scheduled time, return N \(number\) instances nearest to the current time. If not specified, all instances that meet the requirements are shown.
-   -all: The information of all instances that meet requirements is returned. To execute the command, you must have the 'list' permission  for the project. This command can only return 50 records by default. You can -limit number to show more record. For example, use `show p -all -limit 100` to show 100 instance records in the project.
-   The output items: Include StartTime \(the time accurate to seconds\), RunTime \(s\) and Status \(including Waiting, Success, Failed, Running, Cancelled, and Suspended\).

**InstanceID and corresponding SQL are as follows:**

```
StartTime RunTime Status InstanceID Query
2015-04-28 13:57:55 1s Success 20150428xxxxxxxxxxxxxxxxxx ALIYUN$xxxxx@aliyun-inner.com select * from tab_pack_priv limit 20;
...    ...    ...    ...    ...    ...
...    ...    ...    ...    ...    ......
```

**The probable status of an instance is as follows:**

-   Running
-   Success
-   Waiting
-   Failed \(job failed but data in the target table is modified\)
-   Suspended
-   Canceled

**Note:** The commands from the preceding example run in MaxCompute client.

##  Status Instance {#section_abq_rx2_vdb .section}

**Command format:**

```
 
status <instance_id>; -- instance_id: the unique identifier of an instance, to specify which instance to be queried.
```

**Action:**

-   Query the status of specified instance, such as Success, Failed, Running, and Cancelled.
-   If this instance is not created by the current user, exception is returned.

**Example:**

```
odps@ $project_name>status 20131225123xxxxxxxxxxxxxxx;
Success
```

Query the status of an instance which ID is 20131225123xxxxxxxxxxxxxxx, and the result is Success.

**Note:** The commands from the preceding example run in MaxCompute client.

## Top Instance {#section_ifc_2y2_vdb .section}

**Command format:**

```
 top instance;top instance -all;
```

**Action:**

Permission requirements: The user must be a project owner or administrator.

top instance: Displays the job information of the current account that is running in the project. It  is displayed, includesding ISNTANCEID , Owner, Type, StartTime, Progress, Status, Priority, RuntimeUsage \(CPU/MEM\), TotalUsage \(CPU/MEM\), QueueingInfo \(POS/LEN\) and so on.

top instance-all : Returns all jobs that are currently being executed in the current project. This command can only return 50 records by default. You can user -limit number to show more record.

**Example:**

```
odps@ $project_name>top instance;
```

**Note:** The commands from the preceding example run in MaxCompute client \(version 0.29.0 or later\).

##  Kill Instance {#section_apd_ty2_vdb .section}

**Command format:**

```
kill <instance_id>; -- instance_id: The unique identifier of an instance,  which must be ID of an instance whose status is 'Running', otherwise, an error is returned.
```

**Action:**

Stop specified instance. The instance must be in the Running status.

**Example:**

```
odps@ $project_name>kill 20131225123xxxxxxxxxxxxxxx;
```

Stop the instance which ID is 20131225123xxxxxxxxxxxxxxx.

**Note:** 

-   The commands from the preceding example run in MaxCompute client.
-   This is an asynchronous process. It does not mean that the distributed task has stopped after the system accepts the request and returns the result. You can check whether the instance is deleted by using the status command.

## Desc Instance {#section_a1l_fz2_vdb .section}

**Command format:**

```
desc instance <instance_id>; -- instance_id: The unique identifier of an instance.
```

**Action:**

Get the job information according to instance ID, including SQL, owner, startime, endtime, status.

**Example:**

```
odps@ $project_name> desc instance 20150715xxxxxxxxxxxxxxx;
ID 20150715xxxxxxxxxxxxxxx
Owner ALIYUN$XXXXXX@alibaba-inc.com
StartTime 2015-07-15 18:34:41
EndTime 2015-07-15 18:34:42
Status Terminated
console_select_query_task_1436956481295 Success
Query select * from mj_test;
```

Query all the job information related to the instance whose ID is 20150715xxxxxxxxxxxxxxx.

**Note:** The commands from the preceding example run in MaxCompute client.

## Wait instance {#section_llj_zz2_vdb .section}

**Command format:**

```
wait <instance_id>;  -- instance_id: The unique identifier of an instance.
```

**Action:**

Get running task information, including logs based on the instance ID and a logview link. View task details by accessing the logview link.

**Example:**

```
wait 201709251611xxxxxxxxxxxxxx;
ID = 201709251611xxxxxxxxxxxxxx
Log view:
http://logview.odps.aliyun.com/logview/?h=http://service.odps.aliyun.com/xxxxxxxxxx
Job Queueing...
Summary:
resource cost: cpu 0.05 Core * Min, memory 0.05 GB * Min
inputs:
        alian.bank_data: 41187 (588232 bytes)
outputs:
        alian.result_table: 8 (640 bytes)
Job run time: 2.000
Job run mode: service job
Job run engine: execution engine
M1:
        instance count: 1
        run time: 1.000
        instance time:
                min: 1.000, max: 1.000, avg: 1.000
        input records:
                TableScan_REL5213301: 41187 (min: 41187, max: 41187, avg: 41187
)
        output records:
                StreamLineWrite_REL5213305: 8 (min: 8, max: 8, avg: 8)
R2_1:
        instance count: 1
        run time: 2.000
        instance time:
                min: 2.000, max: 2.000, avg: 2.000
        input records:
                StreamLineRead_REL5213306: 8 (min: 8, max: 8, avg: 8)
        output records:
                TableSink_REL5213309: 8 (min: 8, max: 8, avg: 8)
```

