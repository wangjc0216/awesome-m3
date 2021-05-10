# Storage

对于M3DB来说，长期存储单元是fileset files，保存的是compressed streams of time series values(时间序列值的压缩流)

在 block time window 变成不可达后，数据就会落盘()。block time window就是在time window之后，这个block将不能被写入。
如果flush disk之前进程被杀掉，M3DB可以通过commitlog进行恢复。类似与mysql的binlog与redolog，不过mysql需要支持事务，所有有两个log日志来控制事务一致性。

FileSets

文件集包括以下文件：

>Info file
>Summaries file
>Index file
>Data file
> Bloom filter file
> Digests file
> Checkpoint file

//todo 这里还有布隆过滤器，可以看下这个数据结构有什么作用

每个shard/block中的FileSet都会在retention periodz中进行保存。一旦超过时间范围将会被删除。

 

