# Bootstrapping & Crash Recovery
（程序引导 与 崩溃恢复）



## introduction

当M3dB加入到集群后，需要通过commitlog 来进行重写，或者从其他的peer来获取相同的丢失的数据 来完成初始化(bootstrap).

M3DB有5个bootstrappers:
```shell
filesystem
commitlog
peers
uninitialized_topology
noop-all
```
在bootstarpping process开始之前，M3DB Node 需要确定两件事情：
> 1. Node应该确定哪些shard需要进行bootstrap(引导)。哪些shard是由placement来决定的。
> 2. Node应该需要确定每个namespace shard 数据的事件范围，这个是通过namespace retention决定的。


每个bootstrapper会通知bootstrapping process哪个shard/range可以进行以引导并开始引导(bootstarping),直到所有需要的的shards/range都已经完成已完成(fulfilled).
否则M3DB节点启动失败。

### Filesystem Bootstrapper

determine which immutable [Fileset files](../M3DB/Storage.md) exist on disk

### Commitlog Bootstrapper


Commitlog Bootstrapper 通过磁盘上的**commitlog and snapshot (compacted commitlogs) files**来恢复数据。不像Filesystem Bootstrapper,
Commitlog Bootstrapper不能简单的通过持久化过的文件是否满足一个引导请求(bootstrap request),而是通过一个简单的启发法(simple heuristic)

//todo

In other words, the commit log bootstrapper is all-or-nothing for a given shard: it will either return that it can satisfy any time range for a given shard or none at all.

In addition, the commitlog bootstrapper assumes it is running after the filesystem bootstrapper

### Peer Bootstrapper

Peer Bootstrapper 是在M3集群中流式从其他node中同步shard/range的数据。Peer Bootstrapper需要在节点数大于1且副本数大于1的集群中使用。

因为是etcd作为共识，所以要保证CAP中的CP(一致性)，那么对于一致性策略，如果none, one, unstrict_majority（可通过etcd来进行设置）。

### Uninitialized Topology Bootstrapper

## Bootstrapper Configuration



## Recovery


//这个章节没有什么概念，还需要实操之后会有一定的概念。