# Logview {#concept_kkg_xgn_vdb .concept}

Logview是MaxCompute Job提交后查看和Debug任务的工具。

通过Logview可看到一个Job的如下内容：

-   任务的运行状态。
-   任务的运行结果。
-   任务的细节和每个步骤的进度。

Job提交到MaxCompute后，会生成Logview的链接，您可以直接在浏览器上打开Logview链接，进入查看Job的信息，每个Job的Logview页面的有效期为一周。

## Logview功能组件 {#section_a35_tln_vdb .section}

下面结合具体的Logview Web UI界面，为您介绍每个组件的含义。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667422353_zh-CN.png)

Logview的首页分成上下两部分：

-   Instance信息。
-   Task信息。

**Instance信息**

Logview首页中，上半部分是您提交的SQL生成的MaxCompute Instance，每个SQL提交后会生成唯一的ID。Latency指运行总共消耗的时间，其他页面的latency含义类似。下面重点介绍Status。

Status包含四种状态：

-   Waiting：说明当前作业正在MaxCompute中处理，并没有提交到Fuxi中运行。
-   Waiting List : n说明作业已提交到Fuxi并在Fuxi中排队，在队列中处于第n位。
-   Running：作业在Fuxi中运行。
-   Terminated：作业已结束，此时无队列信息。

其中非Terminated状态都可以单击查看详细队列信息。

单击Status查看队列详细信息：

-   Sub Status：表示当前子状态信息。
-   WaitPos：表示排队位置，如果是`0`表示正在运行，如果为`-`表示尚未到Fuxi。
-   QueueLength：表示Fuxi中总的队列长度。
-   Total Priority：表示作业运行时经过系统判断后授予的优先级。
-   SubStatus History：单击后，可以查看作业执行的详细历史状态，包含状态码、状态描述、开始时间、持续时间等（某些版本暂时无历史信息）。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667422356_zh-CN.png)


**Task 信息**

Logview首页中，下半部分是该task的说明，下面重点说明Result与Detail。

Result：

在Job运行结束后，可以看到运行结果，如一条select SQL的结果，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667422365_zh-CN.png)

Detail：

一个Job在运行中和结束后，均可以单击**Detail**来查看任务运行的具体情况。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667422366_zh-CN.png)

-   一个MaxCompute Task由一个或者多个Fuxi Job组成。例如当您的SQL任务十分复杂时，MaxCompute会向Fuxi提交多个Fuxi Job。
-   每个Fuxi Job由一个或者多个Fuxi Task组成。简单的MapReduce通常会产生两个Fuxi Task，一个是Map，一个是Reduce，您会见到两个Fuxi Task的名字分别为M1和 R2，当SQL比较复杂时，可能会产生多个Fuxi Task，如上图所示。
-   在每个Task中，可以看到Task的名字，对于M1，表示这是一个Map task，R5\_4中的4表示它依赖J4执行结束才能开始执行。同理，J4\_1\_2\_3表示Join4这个阶段要依赖M1、M2、M3三个task完全成才能启动运行。
-   I/O Records表示这个task的输入和输出的records数。

单击任一Fuxi Task，可查看Fuxi Instance内容，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12086/15498667422367_zh-CN.png)

每个 Fuxi Task由一个或者多个Fuxi Instance组成，当您的输入数据量变大时，MaxCompute会在每个Task启动更多的节点来处理数据。每个节点就是一个Fuxi Instance。双击Fuxi Task最右边一栏 **查看**，或者直接双击该行，就可以打开具体的Fuxi Instance信息。

在页面的下方，Logview为不同阶段的Instance进行了分组，查看出错的节点可以选择Failed栏。

在StdOut和StdErr两栏中，可以查看标准输出和标准错误信息，您自己打印的信息也可以在这里查看。

## 通过Logview进行问题排查 {#section_xf2_1pn_vdb .section}

-   **出错的任务**

    当有任务出错时，您可以在Logview页面的Result中看到错误的提示信息，也可以在Detail页面中通过Fuxi Instance的stderr，查看具体某个Instance出错的信息。

-   **数据倾斜**

    运行缓慢有时是由于在某个Fuxi Task的所有Fuxi Instance中，有个别Instance形成长尾造成，长尾的现象就是同一个Task内任务分配不均。这时可以在任务运行完后，在Summay标签页中看运行结果。在每个Task中都可以看到形如这样的输出：

    ```
    output records:
    R2_1_Stg1: 199998999  (min: 22552459, max: 177446540, avg: 99999499)
    ```

    在这里如果看到min和max相差很大，就说明在这一阶段出现了数据倾斜。比如在Join时，某个字段中有一个值出现的比例很高，在这一字段上做Join就会出现数据倾斜。


