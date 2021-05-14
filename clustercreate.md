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
创建etcd集群：
```shell
kubectl apply -f https://raw.githubusercontent.com/m3db/m3db-operator/master/example/etcd/etcd-basic.yaml
```
检查etcd集群健康状态：
```shell
kubectl exec etcd-0 -- env ETCDCTL_API=3 etcdctl endpoint health
```
### Install the Operator 
```shell
kubectl apply -f https://raw.githubusercontent.com/m3db/m3db-operator/master/bundle.yaml
```
### Create an M3 Cluster
```shell
kubectl apply -f https://raw.githubusercontent.com/m3db/m3db-operator/master/example/m3db-local.yaml
```
检查M3集群启动情况(bootstrap需要等待一段时间)：
```shell
kubectl exec simple-cluster-rep2-0 -- curl -sSf localhost:9002/health
```
### Deleting a Cluster

```shell
kubectl delete m3dbcluster simple-cluster
```
默认在删除集群之前，`operator`会使用[finalizers](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#finalizers)
来删除`placement`和`namespace`。如果不想这样的操作，在集群配置中设置`keepEtcdDataOnDelete`为`true`

### Organizing Data with Placements and Namespaces
作为分布式TSDB，M3会通过通过shard来进行分片
### Create a Placement and Namespace
创建namespace：
```shell
curl -X POST http://10.43.84.192:7201/api/v1/database/create -d '{
  "type":"cluster"
  "namespaceName": "default",
  "retentionTime": "12h"
}' | jq .
```
以上需要注意，type为cluster，而不是官方文档中写的local。

查看placement和namespace的状态：
```shell
curl http://10.43.84.192:7201/api/v1/services/m3db/placement | jq .

curl http://10.43.84.192:7201/api/v1/services/m3db/namespace | jq .
```

将namespace设置为就绪状态：
```shell
curl -X POST http://10.43.84.192:7201/api/v1/services/m3db/namespace/ready -d '{
  "name": "default"
}' | jq .
```

### Writing and Querying Metrics 



