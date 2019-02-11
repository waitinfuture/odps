# Logview {#concept_kkg_xgn_vdb .concept}

Logview is a tool to view and debug tasks once the MaxCompute job is submitted.

Using Logview, you can see the following details about a job:

-   The Run Status of the task.
-   The operation result of the task.
-   Details of the task and the progress of each step.

When the job is submitted to MaxCompute, a link to the Logview is generated. You can open the Logview link directly on the browser to view information about the job for each job. The Logview page is valid for seven days.

## Features {#section_a35_tln_vdb .section}

The following is a combination of a specific Logview web UI interface to introduce you to each component.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667532353_en-US.png)

A Logview home page is divided into two upper and lower sections:

-   Instance information
-   Task information

**Instance info**

On the Logview home page, the upper half is the MaxCompute instance that you submit to generate SQL. A unique ID is generated after each SQL commit. Latency refers to the amount of time it takes to run, and the latency of other pages is similar.

The following are the four states:

-   Waiting: Indicates that the current job is being processed in MaxCompute and is not committed to Fuxi to run.
-   Waiting List: N indicates that the job was submitted to Fuxi and queued in Fuxi, is in the n-bit in the queue.
-   Running: The Job runs in Fuxi.
-   Terminated: The job has ended with no queue information.

Click the non-terminated status of a job to view detailed queue information.

Click status to view queue details:

-   Sub status: The current sub-status information.
-   Waitpos: The queuing location, where 0 indicates that it is running, and \(-\) indicates that it has not yet arrived Fuxi.
-   Queuing length: The total queue length in the Fuxi.
-   Total priority: The priority granted by the job runtime after it has been judged by the system.
-   SubStatus history: When clicked, you can view the detailed history of job execution. It contains status codes, status descriptions, start time, duration, and so on. \(Currently, some versions have no historical information.\)

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667532356_en-US.png)


**Task information**

In the Logview home page, the lower section is the task description followed by the result description and other details.

Result:

You can view the result after a job executed, such as the results of a select SQL as shown in the following figure:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667532365_en-US.png)

Detail:

After a job is executed, click detail to view the running status of the task.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667532366_en-US.png)

-   A MaxCompute task consists of one or more Fuxi jobs. For example, when your SQL task is complex, MaxCompute goes to Fuxi and submit multiple Fuxi jobs.
-   Each Fuxi job consists of one or more Fuxi tasks. Simple Map Reduce usually produces two Fuxi tasks, namely Map and Reduce. You can view the two Fuxi task names as M1 and R2, respectively. When SQL is complex, more than one Fuxi may generate Task as shown in the preceding figure.
-   Each task displays name of the task. For example, M1 is a map task, 4 In r54 means that it relies on J4. Execution will not begin until the last execution is complete. Similarly, j4\_1\_2\_3 indicates that join4 has to rely on M1, M2, M3 three tasks to start the operation completely.
-   I/O Records represents the number of records for the input and output of this task.

Click any Fuxi task to view the Fuxi instance content, as shown in the following figure:

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667532367_en-US.png)

Each Fuxi task consists of one or more Fuxi instances. When your input data levels are large MaxCompute activates more nodes to process the data. Each node is a Fuxi instance. Double-click the right-side column of the Fuxi task to view, or double-click the row to open the specific Fuxi instance information.

Towards the lower-end of the page, Logview is grouped for different stages of instance. Select the failed column to view the wrong node.

In the stdout and stderr columns, you can view standard output and standard error messages along with the information to be printed.

## Troubleshooting through Logview {#section_xf2_1pn_vdb .section}

-   **Wrong tasks**

    When a task error occurs, a prompt message for the errors in the result on the Logview page pops up. Use the detail page Stderr for Fuxi instance to view information about a specific instance error.

-   **Data skew**

    Slow operation is usually because of individual instances in all Fuxi instances of a certain Fuxi task. Long Tail is caused by the uneven allocation of tasks within the same task. You can view the run results in the summary tab after the task runs. The output of each task is as follows:

    ```
    output records:
    R2_1_Stg1: 199998999 (min: 22552459, max: 177446540, avg: 99999499)
    ```

    In the preceding figure, the large difference between min and max suggests that a data tilt has occurred at this stage. Meaning, if one word with high frequency appears, a tilt appears when you join this word.


