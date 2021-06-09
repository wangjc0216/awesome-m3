# About M3

scale, reliability and efficiency(伸缩、可靠、高效)。M3由三个Components组成。



Ingestion & streaming aggregation

Timeseries Database

Real time query engine



The first iteration of the storage layer featured Cassandra and ElasticSearch.



## Intro
feature  Cassandra and ElasticSearch.

## Storatge

因为打车业务的飞速发展，Uber耗费了大量的人力无力用来灭火，所以Uber团队内部创建了M3DB，一个定制的**嵌入的倒排索引**的TSDB。



## Query

OOM和Slow Query阻碍了团队有效的监控服务。新的查询引擎(也称作是M3Query)，利用了M3DB  `stores data in highly compressed blocks`.
和 在客户端进行解压。M3Query也支持PromQL和Graphite 语法，对于查看它们的指标信息非常的方便。


## Ingestion

Ingestion是引入的意思，数据的引入。

监控的数据量暴涨，没有一种one-size-fits-all的磁盘存储策略。

不同的团队对监控数据要求的**精度**(resolutions)和**保留时间**(retention periods)不同，这是需要需求方来自行判定和取舍的。

这是通过两个组件的组合M3Coordinator和M3Aggregator来实现的，它们俩协同工作(work in tandem)按照自定义的设置(in a highly customizable way)
来获取指标，并对指标进行聚合(aggregation)。 **while providing high availability and efficiency.**(同时提供高可用和高效率)

