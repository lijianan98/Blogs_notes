[三篇文章了解 TiDB 技术内幕 - 说存储 | PingCAP](https://cn.pingcap.com/blog/tidb-internal-1/#Region)
[三篇文章了解 TiDB 技术内幕 - 谈调度 | PingCAP](https://cn.pingcap.com/blog/tidb-internal-3/)

### PD的作用
分布式高可用存储系统设计：
第一点：需要满足什么需求
第二点：以及需要做什么优化
https://cn.pingcap.com/blog/tidb-internal-3/#:~:text=%E4%BD%9C%E4%B8%BA%E4%B8%80%E4%B8%AA%E5%88%86%E5%B8%83,%E5%AE%8C%E6%88%90%E8%B0%83%E5%BA%A6%E8%AE%A1%E5%88%92%E3%80%82

### 如何满足第一类需求
也即实现多副本容错、动态扩容/缩容、容忍节点掉线以及自动错误恢复。引用自上述链接原文：

“调度需求看似复杂，但整理下来最终落地的无非下面三件事：

- 增加一个 Replica
- 删除一个 Replica
- 将 Leader 角色在一个 Raft Group 的不同 Replica 之间 transfer

刚好 Raft 协议能够满足这三种需求，通过 AddReplica、RemoveReplica、TransferLeader 这三个命令，可以支撑上述三种基本操作”

### 调度需要考虑的问题及实现
总体来说，TiKV的节点和Raft Group会心跳定时向PD发送信息，PD再据此作出调度建议返回给Region leader，具体采纳与否取决于Region leader

**综合层面：
* 控制调度速度，避免影响在线服务
**节点层面：
* Store 的存储空间占用大致相等；
* 支持手动下线节点
**Raft group层面：
* Region replica数量正确，均匀分配到不同节点；
* Leader在Store间均匀分配；
* 访问热点数量在 Store 之间均匀分配；
