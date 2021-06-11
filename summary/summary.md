# M3DB介绍

## 0. Reading Guide

1.5 为快速指引 





## 1.  Introduce

### 1.0 Motivation

The M3 platform aims to provide a turnkey, scalable, and configurable multi-tenant store for Prometheus, Graphite and other standard metrics schemas

M3 平台旨在为 Prometheus、Graphite 和其他标准指标模式提供交钥匙、可扩展和可配置的多租户存储。

当然，也是因为[promethue 的远程存储](https://schd.ws/hosted_files/cloudnativeeu2017/73/Integrating%20Long-Term%20Storage%20with%20Prometheus%20-%20CloudNativeCon%20Berlin%2C%20March%2030%2C%202017.pdf)



### 1.1 Overview





### 1.2  Component

[**M3 Coordinator**](https://github.com/m3db/m3/tree/master/src/query)

M3 Coordinator 是一个服务，提供了a global and placement specific level级别 //todo 读写M3DB的API接口。

简单来说，M3通过实现了Prometheus的[proto接口](https://github.com/m3db/m3/tree/master/src/query),M3DB和上游监控服务(主要是Promethues)的桥梁。



**M3DB**

M3DB是提供了可扩展存储和时间的倒排索引的分布式时序数据库。

//todo 描述的过于简单



**M3Query**

M3Query是一个分布式查询引擎，支持查询当前和历史(realtime and historical )的数据。同时M3 Query支持主流的查询SQL，与Prometheus的语法兼容，它支持低延时查询也支持比较大量的数值分析。

M3可以完全替代Prometheus来查询数据，M3 Query可以发挥查询[block processing](https://m3db.io/docs/architecture/m3query/blocks/)的优势。M3 Coordinator做为远程存储时，如果使用Prometheus进行数据查询，Prometheus是从M3 Coordinator中读取全量数据，然后通过自身服务来进行聚合计算。M3 Query可以替代Prometheus这一过程。



**M3Aggregator**

M3 Aggregator是一个分布式的聚合器，在将数据保存在M3DB之前，会对数据进行**一个基于流的有状态的降采样**。聚合规则动态保存在etcd中。

它使用leader选举和聚合窗口跟踪(uses leader election and aggregation window tracking)，利用etcd管理这个状态，对于长期存储的降采样指标至少进行一次聚合。

M3 Coordinator貌似也是可以实现降采样的，在服务上游讲请求下发下来的时候。不同点就是M3 Aggregator是实现了副本和切片(replica and sharding)的，另外M3 Aggregator也需要考虑高可用。

默认情况下，M3 Aggregator是支持集群和副本的。这就意味中，Metrics指标可以路由到指定的Aggregator来进行聚合，同时也不必担心Aggregator的单点故障问题导致聚合失败。



### 1.2.0  这属于啥呢

Namespace



Placement



M3Aggregator 



M3Coordinator



### 1.3 Storage Engine

https://docs.m3db.io/docs/architecture/m3db/engine/

### 1.4 Key Principle 





### 1.5 Quick Start 

> 需要预先安装一个`jq`工具，方便查看查看json格式数据。

可以通过部署单机的`M3DB`容器部署，如下所示：

```
docker run -p 7801:7201 -p 7803:7203  -d  --name single-m3db -v $(pwd)/m3db_data:/var/lib/m3db quay.io/m3db/m3dbnode:v1.0.0
```

通过如下命令初始化`Namespace`和`Placement`：

```
curl -X POST http://localhost:7201/api/v1/services/m3db/namespace/ready -d '{
  "name": "default"
}'  | jq .
```

改变`Namespace`状态为`Ready`：

```
curl -X POST http://localhost:7201/api/v1/services/m3db/namespace/ready -d '{
>   "name": "default"
> }' | jq .
```

查看`Namespace`：

```
curl http://localhost:7201/api/v1/services/m3db/namespace | jq .
```

调用`openAPI`想`M3`中写入数据：

```shell
curl -X POST http://localhost:7201/api/v1/json/write -d '{
  "tags": 
    {
      "__name__": "third_avenue",
      "city": "new_york",
      "checkout": "1"
    },
    "timestamp": '\"$(date "+%s")\"',
    "value": 7347.26
}'
```

查询近45s的数据：

```
curl -X "POST" -G "http://localhost:7201/api/v1/query_range" \
  -d "query=third_avenue" \
  -d "start=$(date "+%s" -d "45 seconds ago")" \
  -d "end=$( date +%s )" \
  -d "step=5s" | jq .  
```

## 2. Usage

### 2.1 OpenAPI



#### Placement

Placement为拓扑结构，在M3中和topology是等同的。如果集群拓扑之初节点A拥有分片1、2和3，那么节点A将拥有集群中所有配置的`namespace`的分片1、2和3.

##### Initalize Placement 

```

```



##### View Placement

```
curl http://<m3-server>:<m3-port>/api/v1/services/m3db/placement | jq .
```







#### Namespace

Namespace相当于其他数据库中表，每个Namespace的命名都是唯一的，且每一个Namespace有独立的数据保留时间(data retention)和块大小(blocksize)。

##### View Namespace

通过调用以下方法查看M3下的Namespace详情：

```
curl http://<m3-server>:<m3-port>/api/v1/services/m3db/namespace
```

##### Create Namespace

先说推荐的创建方式：

```
curl -XPOST http://<m3-server>:<m3-port>/api/v1/database/namespace/create -d '{"namespaceName":"test-namespace", "retentionTime":"240h" }'
```

在推荐的创建Namespace方法中，可以只填入Namespace名称和retentionTime(数据保留时间)进行配置，其他的参数值都是根据retentionTime来配置成推荐值。

当然，如果想要自定义Namespace中的各项参数，可使用`Advanced`方式来进行创建Namespace,可参考[官方文档](https://docs.m3db.io/docs/operational_guide/namespace_configuration/#advanced-hard-way)。

##### Ready Namespace

```
curl -X POST http://<m3-server>:<m3-port>/api/v1/services/m3db/namespace/ready -d '{
  "name": "default_unaggregated"
}' | jq .
```

##### Modify Namespace

修改Namespace并不是一个原子操作，当前可以理解为不支持修改Namespace配置。

##### Delete Namespace

执行以下命令：

```
curl -XDELETE  http://<m3-server>:<m3-port>/api/v1/services/m3db/namespace/test-namespace
```

需要注意的是，在删除后M3并没有真正的删除数据，只有重启M3才会生效。







### 2.2 Benchmark Testing



## 3.Deployment



### 3.1 Single Container 



### 3.2 Container Clusters



### 3.3 Kubernetes Operator 



## 4. M3DB Client Usage



### 4.1 Prometheus SDK



#### push 

https://pkg.go.dev/github.com/prometheus/client_golang/prometheus/push#Pusher.Push





### 4.2 M3 SDK

https://github.com/uber-go/tally/tree/v3.4.1

https://github.com/chronosphereio/tally  改成GoModule来管理了，所以可以参考这个



## 5. Reference



[openAPI Doc](http://m3-pro.meetwhale.com:7201/api/v1/openapi)





