# Set操作 {#concept_yys_xby_rfb .concept}

本文向您介绍如何使用Set命令设置MaxCompute或用户自定义的系统变量，以及如何清除Set命令设置。

## Set {#section_pqs_bff_vdb .section}

**命令格式**

```
set <KEY>=<VALUE>
```

**行为说明**

您可以使用set命令设置MaxCompute或影响MaxCompute的行为。

MaxCompute支持的系统变量如下所示：

```
--MaxCompute SQL及新版本Mapreduce支持的Set命令
set odps.sql.allow.fullscan=  --设置是否允许对分区表进行全表扫描，false不允许，true为允许。
set odps.stage.mapper.mem=    --设置每个map worker的内存大小，单位是M，默认值1024M。
set odps.stage.reducer.mem=   --设置每个reduce worker的内存大小，单位是M，默认值1024M。
set odps.stage.joiner.mem=    --设置每个join worker的内存大小，单位是M，默认值1024M。
set odps.stage.mem =    --设置MaxCompute 指定任务下所有worker的内存大小。优先级低于以上三个set key，单位M，无默认值。
set odps.stage.mapper.split.size=    -- 修改每个map worker的输入数据量，即输入文件的分片大小，从而间接控制每个map阶段下worker的数量，单位M，默认值256M。
set odps.stage.reducer.num=       --修改每个reduce阶段worker数量，无默认值。
set odps.stage.joiner.num=        --修改每个join阶段worker数量，无默认值。
set odps.stage.num=               --修改MaxCompute指定任务的所有阶段的worker的并发度，优先级低于以上三者，无默认值。
set odps.sql.type.system.odps2=   -- 默认为false，SQL（Create、select、insert等操作）中涉及到新数据类型（TINYINT、SMALLINT、 INT、 FLOAT、VARCHAR、TIMESTAMP BINARY）时需要设置为true。
set odps.sql.executionengine.coldata.deep.buffer.size.max=   --调整MaxCompute写表过程中为单个复杂类型的列预先申请的缓存大小，以提高写入性能。
```

**说明：** Set操作命令仅支持小写。

**命令说明**

`set odps.sql.executionengine.coldata.deep.buffer.size.max`

-   **作用**：调整MaxCompute写表过程中为单个复杂类型的列预先申请缓存大小，以便提高写入性能。
-   **使用场景**
    1.  输出的表中含有的复杂数据类型过多。
    2.  输出的表中含有的某个单独的复杂类型变量size过大。
-   **使用说明**
    -   默认值为67108864（64M），默认单位为byte。
    -   如果输出的表有3个列的schema是复杂类型，如列类型为（string，map，struct，array，binary），则默认情况下MaxCompute将会为写表操作预留64M \* 3大小的内存。为每一列预先申请的缓存将会用来存放这一列batch row count行的数据。
    -   如果预先可知道表的复杂类型变量占用的空间比较小，则建议把这个值调小。例如，如果每个复杂类型变量大小不会超过1024 byte，同时batch row count值使用的是默认值（1024），则可以将flag设置为1024 \* 1024，如下所示：

        ```language-sql
        set odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
        ```

    -   如果您预先知道每个复杂类型的值都在7M到8M之间，同时指定了batch row count为32，则这个值可以被调整为8M \* 32。
    -   如果任务的输出带有复杂类型，或者任务的mapjoin小表带有复杂类型，这个值的调整会影响到任务执行过程中使用的内存。根据前面的计算方法，值设的过大有可能导致任务OOM内存溢出。

## Show Flags {#section_ft4_jff_vdb .section}

**命令格式**

```
show flags; --显示Set设置的参数
```

**行为说明**

运行Use Project命令会清除掉Set命令设置的配置。

