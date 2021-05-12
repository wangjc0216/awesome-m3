# M3DB集群创建

使用k8s来创建M3DB集群，可以参考 [kubecon的ppt分享](https://kccna18.sched.com/event/Gsxn/keynote-smooth-operator-large-scale-automated-storage-with-kubernetes-celina-ward-software-engineer-matt-schallert-site-reliability-engineer-uber)

这里推荐使用M3的`kubernetes operator`来部署M3集群，这是更简单的设置。

## M3 Architecture

M3集群主要是两种节点类型：

- [ ] Storage Node： 工作节点，workhorse，存储数据并且支持读写功能。

- [ ] Coordinator: 调度整个集群的读写，轻量进程，不保存数据。


m3 coordinator 主要暴露两个端口：

>7201：管理集群拓扑，该断点负责集群的大部分API请求。
>7203：M3DB 和 M3Coordinator的prometheus监控指标。

还有两个不常用的Node类型：

- [ ] Query nodes
- [ ] Aggregator nodes

## 前置条件

a. M3使用etcd作为分布式key-value存储，用于以下功能：
```shell
1. 实时更新集群配置
2. 管理distributed and sharded集群的placement
```
b. 创建一个k8s集群

## 搭建M3DB集群
### Create an etcd Cluster
```shell
kubectl apply -f https://raw.githubusercontent.com/m3db/m3db-operator/master/example/etcd/etcd-basic.yaml
```
### Install the Operator 
### Create an M3 Cluster 
### Deleting a Cluster 
### Organizing Data with Placements and Namespaces 
### Create a Placement and Namespace
### Writing and Querying Metrics 



