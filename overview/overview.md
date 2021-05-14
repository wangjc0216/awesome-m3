# 概述

## 组件

### M3 Coordinator
`M3 Coordinator`用来协调上游服务(如`prometheus`)与`M3DB`读写操作。
它作为一个桥梁让M3DB与其他监控系统(如promethues)与M3DB让用户获取长期存储和多中心构建。
可以参考promethues关于存储的[ppt](https://schd.ws/hosted_files/cloudnativeeu2017/73/Integrating%20Long-Term%20Storage%20with%20Prometheus%20-%20CloudNativeCon%20Berlin%2C%20March%2030%2C%202017.pdf)

### M3DB
M3DB是一个分布式数据库，具有scalable storage(弹性存储) 与 a reverse index of time series(时间序列的倒排索引)。
它在实时性和长期保留指标是有较好的优势。


### M3 Query
M3 Query是一个查询实时和历史时序数据的分布式Query Engine。

### M3 Aggregator 
M3 Aggregator 是一个专门的指标聚合器，它在M3DB将metrics保存在节点之前提供基于有状态的基于流的下采样。它使用保存在etcd的动态规则。

## 目的
大概内容： 做一个prometheus的存储

## Media
关于博客、MeetUp、Recorded Talks

## RoadMap









