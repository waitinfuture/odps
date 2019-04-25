# 扩展MapReduce {#concept_ngt_sxf_vdb .concept}

相比于传统的MapRudce，MaxCompute提供的扩展MapReduce模型（简称MR2）改变了底层的调度和IO模型，可避免作业时冗余的IO操作。

与[MaxCompute](intl.zh-CN/用户指南/MapReduce/概要/MapReduce概述.md)相比，MR2在Map/Reduce等函数编写方式上基本一致，较大的不同点发生在作业时。更多详情请参见[扩展MapReduce示例](intl.zh-CN/用户指南/MapReduce/示例程序/Pipeline示例.md)。

## MR2 模型产生背景 {#section_9qj_uiq_46y .section}

传统的MapReduce模型要求在经过每一轮MapReduce操作后，得到的数据结果必须存储到分布式文件系统中（例如，HDFS或MaxCompute数据表）。MapReduce模型通常由多个MapReduce作业组成，每个作业执行完成后都需要将数据写入磁盘，然而后续的Map任务很可能只需要读取一遍这些数据，为之后的Shuffle阶段做准备，这种情况就产生了冗余的磁盘IO操作。

MaxCompute的计算调度逻辑可以支持更复杂的编程模型， 针对上述情况，可以在Reduce后直接执行下一次的Reduce操作，而不需要中间插入一个Map操作。因此，MaxCompute提供了扩展的MapReduce模型，即可以支持Map后连接任意多个Reduce操作，比如Map\>Reduce\>Reduce。

## 与Hadoop Chain Mapper/Reducer对比 {#section_7gv_tuk_wsm .section}

Hadoop Chain Mapper/Reducer也支持类似的串行化Map或Reduce操作，但和MaxCompute的扩展MapReduce（MR2）模型有本质的区别。

Chain Mapper/Reducer基于传统的MapReduce模型，仅可以在原有的Mapper或Reducer后增加一个或多个Mapper操作（不允许增加Reducer）。这样的优点是：您可以复用之前的Mapper业务逻辑，把一个Map或Reduce拆成多个Mapper阶段，同时本质上并没有改变底层的调度和I/O模型。

