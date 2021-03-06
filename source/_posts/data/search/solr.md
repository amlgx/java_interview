# Lucene

## 介绍一下lucene?

Lucene全文检索库成熟、高性能、可扩展、轻量级、功能强大，而且使用简单、不依赖第三方，单个的Lucene库文件可以在程序中直接使用。结构概念包括：文档(document，包括多个字段的数据)、字段(field，包括字段的名称和内容)、词项(term)和词条(token，包括词项文本、开始和结束的偏移量、词条类型)。Lucene索引的数据结构为倒排索引，功能上通过把词项映射到文档及其数量，结构上包括一个有序的“词项-频率”字典和与词项对应的postings文件列表。以每个索引由多个创建后不修改的段组成，多个段会在段合并时被合并到一起。

# ES

## es包括那些概念？

es对Lucene的封装实现了自动扩展的分布式集群功能。

逻辑概念包括：索引(数据库)，文档(表的一行记录)，类型(表结构)，映射(搜索文本处理规则，包括分词)。

物理概念包括：集群，节点，分片(表)，副本。

## es的架构

节点：es集群基于对等架构，每个物理节点Node默认是集群的一部分。集群默认使用Zen发现机制管理，发现方式包括默认的多播和单播。

索引：逻辑上的索引默认分为5个主分片(相当于kafka分区的首领副本)，分布于不同的物理节点上，主分片数在索引创建后不可修改。每个主分片有0到多个副本(主分片不叫副本，计数方式与kafka不一样)，默认1个，可以动态调整。

分片：一个es分片就是一个Lucene索引，包括多个setment。segment内部包括倒排索引(文档查找)、保存的字段(字段查找)、文档值(排序、聚合、切面)和缓存。

## es的工作过程

es节点启动时使用发现模块查找同一集群的节点，默认向网络广播请求。集群中节点编号小的节点优先被选举为主节点，负责管理集群状态，并在节点增减时重新分配索引分片。如果管理节点读取集群的状态信息后发现需要恢复，就检查所有索引的分片，并决定主分片，使系统进入可查询的黄色状态。然后寻找或创建足够的分片副本，使系统进入全功能的绿色状态。

es运行时会启动2个探测进程，分别用于主节点和其他节点通过发送ping请求来相互检测节点是否可用。理想的集群拓扑包括3个专用的主节点

(master:true,data:false)，并且设置主节点的最小数量(discovery.zen.minimum_master_nodes:2).

es索引一个文档时，会为文件建立相应的缓存，定期刷新到新的segments磁盘文件，就可以被搜索到，而过多的segment会在段合并时删掉。

