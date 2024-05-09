Tinykv和etcd实现很像，先从整体架构上有个了解：
[Etcd Raft库的工程化实现 - codedump的网络日志](https://www.codedump.info/post/20210515-raft/#%E6%A6%82%E8%BF%B0)

![[Pasted image 20231111233109.png]]

### 2ac part: Implement the raw node interface

`raft.RawNode` in `raft/rawnode.go` is the interface we interact with the upper application, `raft.RawNode` contains `raft.Raft` and provide some wrapper functions like 
`RawNode.Tick()`
`RawNode.Step()`. 
Also it provides `RawNode.Propose()` 
to let the upper application propose new raft logs.

### struct `Ready` 

When handling messages or advancing the logical clock, the `raft.Raft` may need to interact with the upper application, like:
- send messages to other peers
- save log entries to stable storage
- save hard state like the term, commit index, and vote to stable storage
- apply committed log entries to the state machine
- etc

But these interactions do not happen immediately, instead, they are encapsulated in `Ready` and returned by `RawNode.Ready()` to the upper application. It is up to the upper application when to call `RawNode.Ready()` and handle it.  After handling the returned `Ready`, the upper application also needs to call some functions like `RawNode.Advance()` to update `raft.Raft`'s internal state like the applied index, stabled log index, etc.
