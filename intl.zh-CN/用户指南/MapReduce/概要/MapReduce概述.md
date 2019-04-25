# MapReduce概述 {#concept_fml_smf_vdb .concept}

本文为您介绍MaxCompute支持的MapReduce编程接口及使用限制。

MaxCompute提供三个版本的MapReduce编程接口。

-   MaxCompute MapReduce：MaxCompute的原生接口，执行速度更快，开发更便捷，不暴露文件系统。
-   MaxCompute 扩展MapReduce[MR2](intl.zh-CN/用户指南/MapReduce/概要/扩展MapReduce.md)）：对MaxCompute MapReduce的扩展，支持更复杂的作业调度逻辑。MapReduce的实现方式与MaxCompute原生接口一致。
-   [Hadoop兼容版本MapReduce](intl.zh-CN/用户指南/MapReduce/概要/开源兼容MapReduce.md)：高度兼容[Hadoop MapReduce](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html) ，与MaxCompute MapReduce [MR2](intl.zh-CN/用户指南/MapReduce/概要/扩展MapReduce.md)不兼容。

以上三个版本在[基本概念](intl.zh-CN/用户指南/MapReduce/功能介绍/基本概念.md)、[作业提交](intl.zh-CN/用户指南/MapReduce/功能介绍/作业提交.md)、[输入与输出](intl.zh-CN/用户指南/MapReduce/功能介绍/输入与输出.md)、[资源使用](intl.zh-CN/用户指南/MapReduce/功能介绍/资源使用.md)等方面基本一致，仅各版本的Java SDK有所不同。本文仅对MapReduce的基本原理做简单介绍，更多详情请参见[Hadoop Map/Reduce教程](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)。

**说明：** 您无法通过MapReduce读写[外部表](intl.zh-CN/用户指南/外部表/访问OSS非结构化数据.md#)中的数据。

## 应用场景 {#section_m5s_rwf_vdb .section}

MapReduce支持下列场景：

-   搜索：网页爬取、倒排索引、PageRank。
-   Web访问日志分析：
    -   分析和挖掘用户在Web上的访问、购物行为特征，实现个性化推荐。
    -   分析用户访问行为。
-   文本统计分析：
    -   热门小说的字数统计（WordCount）、词频TFIDF分析。
    -   学术论文、专利文献的引用分析和统计。
    -   维基百科数据分析等。
-   海量数据挖掘：非结构化数据、时空数据、图像数据的挖掘。
-   机器学习：监督学习、无监督学习、分类算法如决策树、SVM等。
-   自然语言处理：
    -   基于大数据的训练和预测。
    -   基于语料库构建单词同现矩阵，频繁项集数据挖掘、重复文档检测等。
-   广告推荐：用户点击（CTR）和购买行为（CVR）预测。

## 处理流程 {#section_l5j_vwf_vdb .section}

MapReduce处理数据过程主要分成Map和Reduce两个阶段。首先执行Map阶段，再执行Reduce阶段。Map和Reduce的处理逻辑由用户自定义实现，但要符合MapReduce框架的约定。处理流程如下所示：

1.  在正式执行Map前，需要将输入数据进行**分片**。所谓分片，就是将输入数据切分为大小相等的数据块，每一块作为单个Map Worker的输入被处理，以便于多个Map Worker同时工作。
2.  分片完毕后，多个Map Worker便可同时工作。每个Map Worker在读入各自的数据后，进行计算处理，最终输出给Reduce。Map Worker在输出数据时，需要为每一条输出数据指定一个Key，这个Key值决定了这条数据将会被发送给哪一个Reduce Worker。Key值和Reduce Worker是多对一的关系，具有相同Key的数据会被发送给同一个Reduce Worker，单个Reduce Worker有可能会接收到多个Key值的数据。
3.  在进入Reduce阶段之前，MapReduce框架会对数据按照Key值排序，使得具有相同Key的数据彼此相邻。如果您指定了**合并操作（Combiner）**，框架会调用Combiner，将具有相同Key的数据进行聚合。Combiner的逻辑可以由您自定义实现。与经典的MapReduce框架协议不同，在MaxCompute中，Combiner的输入、输出的参数必须与Reduce保持一致，这部分的处理通常也叫做**洗牌（Shuffle）**。
4.  接下来进入Reduce阶段。相同Key的数据会到达同一个Reduce Worker。同一个Reduce Worker会接收来自多个Map Worker的数据。每个Reduce Worker会对Key相同的多个数据进行Reduce操作。最后，一个Key的多条数据经过Reduce的作用后，将变成一个值。

**说明：** 上文仅是对MapReduce框架做简单介绍，更多详情请查阅[功能介绍](intl.zh-CN/用户指南/MapReduce/功能介绍/基本概念.md#)。

下文将以WordCount为例，为您介绍MaxCompute MapReduce各个阶段的概念。

假设存在一个文本a.txt，文本内每行是一个数字，您要统计每个数字出现的次数。文本内的数字称为Word，数字出现的次数称为Count。如果MaxCompute MapReduce完成这一功能，需要经历以下流程，如下图所示：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12013/15561790651922_zh-CN.jpg)

**操作步骤** 

1.  输入数据：对文本进行分片，将每片内的数据作为单个Map Worker的输入。
2.  Map阶段：Map处理输入，每获取一个数字，将数字的Count 设置为1，并将此<Word, Count\>对输出，此时以Word作为输出数据的Key。
3.  Shuffle\>合并排序：在Shuffle阶段前期，首先对每个Map Worker的输出，按照Key值（即Word值）进行排序。排序后进行Combiner操作，即将Key值（Word值）相同的Count累加，构成一个新的<Word, Count\>对。此过程被称为合并排序。
4.  Shuffle\>分配Reduce：在Shuffle阶段后期，数据被发送到Reduce端。Reduce Worker收到数据后依赖Key值再次对数据排序。
5.  Reduce阶段：每个Reduce Worker对数据进行处理时，采用与Combiner相同的逻辑，将Key值（Word 值）相同的Count累加，得到输出结果。
6.  输出结果数据。

**说明：** 由于MaxCompute的所有数据都被存放在表中，因此MaxCompute MapReduce的输入、输出只能是表，不允许您自定义输出格式，不提供类似文件系统的接口。

## 使用限制 {#section_fcf_mcn_sfb .section}

MapReduce[使用限制汇总](intl.zh-CN/用户指南/MapReduce/MR限制项汇总.md#)。

有关本地运行的MapReduce使用限制，您可以参考[本地运行和分布式环境运行差异](intl.zh-CN/用户指南/MapReduce/功能介绍/本地运行.md#section_k3l_t3g_vdb)。

