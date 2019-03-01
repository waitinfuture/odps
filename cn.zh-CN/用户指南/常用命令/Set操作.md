# Set操作 {#concept_yys_xby_rfb .concept}

本文向您介绍如何使用set命令设置MaxCompute或者用户自定义的系统变量，以及如何清除Set命令设置。

## Set {#section_pqs_bff_vdb .section}

命令格式如下：

```
set <KEY>=<VALUE>
```

行为说明如下：

您可以使用set命令设置MaxCompute或影响MaxCompute的行为。

**目前，MaxCompute支持的系统变量，如下所示：**

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
set odps.stage.num=               --修改MaxCompute 指定任务的所有阶段的worker的并发度，优先级低于以上三者，无默认值。
set odps.sql.type.system.odps2=   -- 默认为false，SQL（Create、select、insert等操作）中涉及到新数据类型（TINYINT、SMALLINT、 INT、 FLOAT、VARCHAR、TIMESTAMP BINARY）时需要设置为true。
```

**说明：** Set操作命令仅支持小写。

## Show Flags {#section_ft4_jff_vdb .section}

命令格式如下：

```
show flags; --显示Set设置的参数
```

行为说明如下：

运行Use Project命令会清除掉Set命令设置的配置。

