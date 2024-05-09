[Quorum (distributed computing) - Wikipedia](https://en.wikipedia.org/wiki/Quorum_(distributed_computing))

### Quorum-based voting in commit protocols

### Quorum-based voting for replica control

为了满足serializabilty，对一个data item（考虑分布式有多副本的情况），并发的R+W是不允许的（属于不同的transaction）

每个copy有一个vote，voting需要满足如下条件（Vr，Vw含义也就是transaction要写读一个item时需要拿到的vote数量）：
	1. Vr + Vw > V  => 禁止并发读写，同时该条件还保证了读、写集合有重合，所以读一定包含了至少一个拥有最新版本item的节点
	2. Vw > V/2 => 禁止并发写写

### Majority和Quorum系统的区别
[周刊（第17期）：Read-Write Quorum System及在Raft中的实践 - codedump的网络日志](https://www.codedump.info/post/20220528-weekly-17/)