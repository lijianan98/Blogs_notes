一个TiKV instance有两个RocksDB instance，分别是存储数据和raft log，底层采用LSM tree架构：

![[1699220889156.png]]
核心是写入流程：
1. wal，memtable，immutable memtable，SST table四大部分
2. 对于wal写入有两种： `sync-log`和 `wal-bytes-per-sync`
3. L(i-1)层满了后compact到Li层
4. 一个问题：为什么引入sorted run，是否是sorted run写放大问题导致了多级的设计（个人觉得是的）
	因为多级 => 减小write amplification，解释如下，假设就两层，每次L0 compact到L1，可能覆盖范围是所有data，如果有三层则不同，绝大部分data在L2层，write amplification大大减小，
[LevelDB之Compaction实现 | Calvin's Marbles (calvinneo.com)](http://www.calvinneo.com/2021/04/18/leveldb-compaction/)
https://zhuanlan.zhihu.com/p/112574579
	上述链接是关于tiered(overlapping)和leveled(non-overlapping)的设计的对比，从write/read/space amplification三个角度进行了对比
	注意RocksDB是leveled的设计。
5. 每个文件有个bloom filter

读取：
1. block cache缓存热点读取数据
2. 比B+ tree慢

RocksDB还有Column Family：简单的说，就是每个CF负责不同的表：
![[1699220823104.png]]