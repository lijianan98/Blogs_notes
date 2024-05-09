## 相关背景知识（教科书）
### Lock vs latches (logical vs physical)
Project2 of B+ tree is about latches, low-level

lock requests  how to prevent starvation => pending transactions on an item store in queue (FIFO)

Project 4, use "2PL" to implement RC / RUC / RR
## Two-Phase Lock Protocol:

### what 2PL protocol is used for (use basic 2PL as an example):
A locking mechanism to ensure all valid(transaction schedule that meets 2PL requirement) schedules are conflict serializable, example,: prevents repeatable read problem, think about this using conflict serializability to explain.
The locking point is at the boundary of shrinking and growing phase and txns can be scheduled based on the locking point sequence

## basic 2PL problems:
1. dirty read, which further causes cascading abort => serious, should be solved
2. deadlock

## strict 2PL
hold all exclusive lock until the end (txn commit or abort), thus solving above problem 1:
1. cascading abort
rigorous 2PL => hold all locks until...

### lock conversion:
1. upgrade allowed in growing phase, downgrade allowed in shrinking phase
2. not allowed in other cases
[concurrency - Does upgrading after all locks and before all unlocks violate 2PL? - Computer Science Stack Exchange](https://cs.stackexchange.com/questions/131268/does-upgrading-after-all-locks-and-before-all-unlocks-violate-2pl)
(stackoverflow link about lock conversion)

### Common lock types:

S X IS IX SIX (IS IX SIX helps improving performance through hierarchy locking)

**Shared with intent exclusive (SIX)**
– when acquired, this lock indicates that the transaction intends to read all resources at a lower hierarchy and thus acquire the shared lock on all resources that are lower in hierarchy, and in turn, to modify part of those, but not all. In doing so, it will acquire an intent exclusive (IX) lock on those lower hierarchy resources that should be modified. In practice, this means that once the transaction acquires a SIX lock on the table, it will acquire intent exclusive lock (IX) on the modified pages and exclusive lock (X) on the modified rows.

Only one shared with intent exclusive lock (SIX) can be acquired on a table at a time and it will block other transactions from making updates, but it will not prevent other transactions to read the lower hierarchy resources they can acquire the intent shared (IS) lock on the table

![[1698367536697.png]]

Problem:
Why are IX and S locks incompatible? Obviously. 
Then are they at the same level? No, both IX and S can only upgrade to SIX or X.

## 实验分析
### Part 1 
![[1698475458232.png]]

#### 基本框架
LockManager class维护了Table-level和tuple-level的哈希队列如上，一个OID或一个RID对应着唯一的一个table或tuple。
所有的txn发出的LockRequest都存在相应的队列LockRequestQueue中，如上图所示，并维护了granted状态表示该request是否成功获得了锁。

对于阻塞的事务，采用经典的消费者-生产者模型，在LockRequestQueue中维护了相应的condition variable，wait所有阻塞的事务。当有事务完成UnlockTable或UnlockRow，notify_all()唤醒所有等待的事务线程并尝试获得相应的锁，这里需要考虑18.16中的comptability matrix。另外需要注意的细节就是，具体实现时按照教科书及代码hints里的指导，upgrade txn是最高优先级，其次就是每一个request必须等待前面所有的request被grant lock，也就是遵循FIFO顺序

出于性能考虑，采用Hierarchy locking的方法，本实验仅有两个层级，table-level和tuple-level，具体差别为tuple-level不允许获得Intention lock，且加锁之前需要确保所属table已经获得“对应”的锁。

解锁时，该txn在table-level需要确保所有的row-level均已解锁

```
   *    REPEATABLE_READ:
   *        The transaction is required to take all locks.
   *        All locks are allowed in the GROWING state
   *        No locks are allowed in the SHRINKING state
   *
   *    READ_COMMITTED:
   *        The transaction is required to take all locks.
   *        All locks are allowed in the GROWING state
   *        Only IS, S locks are allowed in the SHRINKING state
   *
   *    READ_UNCOMMITTED:
   *        The transaction is required to take only IX, X locks.
   *        X, IX locks are allowed in the GROWING state.
   *        S, IS, SIX locks are never allowed
```

这一部分实验需要考虑的其它细节问题，包括lock upgrade，不同隔离级别允许获得什么样的锁，允许在什么阶段获取锁，代码hints已经给出了详细的指导。
### Part 2 Deadlock detection


### Part 3 Concurrent Query Execution

基于不同的隔离级别，加锁的时候有些细节需要注意，尤其是RC与RR，bustub中似乎并没有专门的代码处理cascading abort，所以一个事务如果abort了，如果按照basic 2PL逻辑实现无法保证其它事务不读取abort事务修改的值，因此为了避免cascading abort的问题，所有的X锁应当在最后释放，也就是strict 2PL的做法。进一步的细节总结如下：

Read Uncommitted：
只考虑IX/X锁，不加其它的锁

Read Committed：
RC允许在shrinking阶段加 IS/S 锁，同时只有释放X锁才会进入shrinking阶段，其特点在于不保证可重复读，因为每次读完都释放读锁，这一点与RR是不同的

Repeatable Read：
本实验中为了确保可重复读，所有的锁应当在最后释放，原因是=>按照strict 2PL，假设item A只会被读，可以在其不再会被读后就释放，而非等待commit or abort。但是实际的系统实现里，很难预知该txn什么时候彻底结束对item A的所有操作（假设为只读），因此本实验RR按照教科书实现，也就是rigorous 2PL，复制文字如下：

A simple but widely used scheme automatically generates the appropriate lock and unlock instructions for a transaction, on the basis of read and write requests from the transaction: 
* When a transaction Ti issues a read(Q) operation, the system issues a lock-S(Q) instruction    followed by the read(Q) instruction. 
* When Ti issues a write(Q) operation, the system checks to see whether Ti already holds a shared lock on Q. If it does, then the system issues an upgrade(Q) instruction, followed by the write(Q) instruction. Otherwise, the system issues a lock-X(Q) instruction, followed by the write(Q) instruction. 
* All locks obtained by a transaction are unlocked after that transaction commits or aborts.

#### 修改SeqScan，Insert，Delete三个executor，加锁的顺序：
对于select操作，先对表加IS锁，再对tuple加S锁
对于insert，delete操作，先对表加IX锁，再对tuple加X锁