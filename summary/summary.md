# M3DB介绍

## 1.  Introduce

### 1.1 Overview



### 1.2 M3DB component



Namespace



Placement



M3Aggregator 



M3Coordinator



### 1.3 Storage Engine

https://docs.m3db.io/docs/architecture/m3db/engine/

### 1.4 Key Principle 



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





