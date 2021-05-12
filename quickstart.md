# 快速入门


## 使用Docker创建单节点M3集群

### 前置条件
一些工具

### 启动M3DB
```shell
docker run  -p 7201:7201 -p 7203:7203 --name m3db  -d -v $(pwd)/m3db_data:/var/lib/m3db quay.io/m3db/m3dbnode:v1.0.0
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
在创建namespace的时候，M3DB会打印一些关于`Bootstrap`的引导日志。

#### 设置namespace为就绪状态

一旦namespace已经完成引导，需要手动去调用Ready接口：
```shell
http -v POST http://10.0.0.199:7201/api/v1/services/m3db/namespace/ready  name=default
```

#### 查看namesapce的细节

就可以看到该namespace的状态(包括上一个请求更新的Ready状态)
```
curl http://localhost:7201/api/v1/services/m3db/namespace | jq .
```

## Writing and Querying Metrics

M3支持写入Prometheus和statsd数据。但是从官网来看，支持Prometheus是主要重点。
插入数据的类型有两种，分别为protobuf和json格式，后者主要用于测试，性能方面不如前者(前者将数据压缩，肯定前者好呀)，不建议在生产上使用。

json数据写入的配置：
```shell
curl -X POST http://localhost:7201/api/v1/json/write -d '{
  "tags": 
    {
      "__name__": "third_avenue",
      "city": "new_york",
      "checkout": "1"
    },
    "timestamp": '\"$(date "+%s")\"',
    "value": 3347.26
}'
```
prometheus使用M3DB的配置：
```shell
global:
  scrape_interval: 15s
  scrape_timeout: 15s
  external_labels:
    monitor: 'codelab-monitor'

scrape_configs:
- job_name: cadvisor
  static_configs:
  - targets: ['10.0.0.108:8099']
- job_name: node-exporter
  static_configs:
  - targets: ['10.0.0.108:9100']
- job_name: m3
  static_configs:
  - targets:  ['10.0.0.199:7203']
remote_read:
  - url: "http://10.0.0.199:7201/api/v1/prom/remote/read"
    read_recent: true

remote_write:
  - url: "http://10.0.0.199:7201/api/v1/prom/remote/write"
```


与Promethues的查询API类似，调用M3DB进行查询：
```shell
curl -X "POST" -G "http://localhost:7201/api/v1/query_range" \
  -d "query=rate(node_disk_read_bytes_total[2m])" \
  -d "start=$(date "+%s" -d "45 minutes ago")" \
  -d "end=$( date +%s )" \
  -d "step=1m" | jq .
```

