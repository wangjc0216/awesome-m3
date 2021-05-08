# 快速入门


## 使用Docker创建单节点M3集群

### 前置条件
一些工具

### 启动M3DB
```shell
docker run -p 7201:7201 -p 7203:7203 --name m3db -v $(pwd)/m3db_data:/var/lib/m3db quay.io/m3db/m3dbnode:v1.0.0
```

### 配置文件
配置文件，分别为coordinator和db两部分。
```shell
coordinator: {}
db: {}
```

### 使用Placement和Namespace来组织数据

TSDB通常是一个节点来保存metrics数据。使用起来简单，但是随着时间推移，metrics读写的增加会导致scalability(可扩展性)问题。

M3，作为分布式TSDB，解决问题通过将metrics data分发到多个节点上。M3通过certain criteria(适当的标准)进行分片。

M3会使用不同的terminology(术语)来表示一下分布式数据库中的概念。

**placement**： 每个集群都有一个，来映射shards(切片)到集群中的节点(node)。
**namespace**： 每个集群有0或者多个，类似于数据库table的概念，每个节点为它拥有的分片(shard)的命名空间(namespace)服务,每个namespace都
有一些配置项，如name，retention time for the data(数据的保留时间)

### 创建Placement和Namespace

```shell
//创建namespace
http -v POST http://10.0.0.199:7201/api/v1/database/create type=local namespaceName=default retentionTime=12h


//查看placement的状态
curl http://localhost:7201/api/v1/services/m3db/placement | jq .

```
//todo Read more about the bootstrapping process


Replication factor: 3

                                 ┌─────────────────┐          ┌─────────────────┐        ┌─────────────────┐       ┌─────────────────┐
                                 │     Node A      │          │     Node B      │        │     Node C      │       │     Node D      │
┌──────────────────────────┬─────┴─────────────────┴─────┬────┴─────────────────┴────┬───┴─────────────────┴───┬───┴─────────────────┴───┐
│                          │ ┌─────────────────────────┐ │ ┌───────────────────────┐ │ ┌──────────────────────┐│                         │
│                          │ │                         │ │ │                       │ │ │                      ││                         │
│                          │ │                         │ │ │                       │ │ │                      ││                         │
│                          │ │   Shard 1: Available    │ │ │  Shard 1: Available   │ │ │  Shard 1: Available  ││                         │
│  1) Initial Placement    │ │   Shard 2: Available    │ │ │  Shard 2: Available   │ │ │  Shard 2: Available  ││                         │
│                          │ │   Shard 3: Available    │ │ │  Shard 3: Available   │ │ │  Shard 3: Available  ││                         │
│                          │ │                         │ │ │                       │ │ │                      ││                         │
│                          │ │                         │ │ │                       │ │ │                      ││                         │
│                          │ └─────────────────────────┘ │ └───────────────────────┘ │ └──────────────────────┘│                         │
├──────────────────────────┼─────────────────────────────┼───────────────────────────┼─────────────────────────┼─────────────────────────┤
│                          │                             │                           │                         │                         │
│                          │ ┌─────────────────────────┐ │ ┌───────────────────────┐ │ ┌──────────────────────┐│┌──────────────────────┐ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │    Shard 1: Leaving     │ │ │   Shard 1: Available  │ │ │  Shard 1: Available  │││Shard 1: Initializing │ │
│   2) Begin Node Add      │ │    Shard 2: Available   │ │ │   Shard 2: Leaving    │ │ │  Shard 2: Available  │││Shard 2: Initializing │ │
│                          │ │    Shard 3: Available   │ │ │   Shard 3: Available  │ │ │  Shard 3: Leaving    │││Shard 3: Initializing │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ └─────────────────────────┘ │ └───────────────────────┘ │ └──────────────────────┘│└──────────────────────┘ │
│                          │                             │                           │                         │                         │
├──────────────────────────┼─────────────────────────────┼───────────────────────────┼─────────────────────────┼─────────────────────────┤
│                          │                             │                           │                         │                         │
│                          │ ┌─────────────────────────┐ │ ┌───────────────────────┐ │ ┌──────────────────────┐│┌──────────────────────┐ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │   Shard 2: Available    │ │ │  Shard 1: Available   │ │ │  Shard 1: Available  │││  Shard 1: Available  │ │
│  3) Complete Node Add    │ │   Shard 3: Available    │ │ │  Shard 3: Available   │ │ │  Shard 2: Available  │││  Shard 2: Available  │ │
│                          │ │                         │ │ │                       │ │ │                      │││  Shard 3: Available  │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ │                         │ │ │                       │ │ │                      │││                      │ │
│                          │ └─────────────────────────┘ │ └───────────────────────┘ │ └──────────────────────┘│└──────────────────────┘ │
│                          │                             │                           │                         │                         │
└──────────────────────────┴─────────────────────────────┴───────────────────────────┴─────────────────────────┴─────────────────────────┘
