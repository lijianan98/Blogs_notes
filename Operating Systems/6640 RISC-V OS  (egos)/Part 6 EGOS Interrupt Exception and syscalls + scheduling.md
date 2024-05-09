
## CLINT
[SiFive FE310-G002 Manual: v19p04 (naizhengtan.github.io)](https://naizhengtan.github.io/23fall/docs/sifive-fe310-v19p04.pdf)
![[1701926765289.png]]

## Grass Layer
### syscall.c
1. syscall flow diagram
2. syscall_invoke() method, which is Syscall first half , running in user-space: asynchronous implementation is using MSIP register, see above. synchronous implementation is using ecall instruction
3. jumps to kernel.c file trap_entry() method, whose address is stores in mtvec register
4. proc_syscall() method, which is running in 

### kernel.c
1. trap_entry() method, checks nested trap, save mcause in trap_cause for distinguishing interrupt and exceptions and calls ctx_start(assembly implementation) to switch to GRASS_STACK_TOP which is earth/grass(kernel) stack.
2. ctx_start the calls ctx_entry() method in kernel.c
3. ctx_entry() calls manage_userstack() method to store user process stack temporarily on a data structure(current implementation) then calls intr_entry() or excp_entry() entry points by checking MSB of mcause register(previously stored in trap_cause in step 1 above)
4. Preemptive scheduling, then in intr_entry or excp_entry() the process switches
5. intr_entry() and excp_entry() both calls proc_syscall() method for system call, both kills the process by setting mepc to APPS_ENTRY + 0xC which is exit() == 0x0820_0000 + 0xC
	However, intr_entry() also handles like timer interrupt by calling proc_yield()
6. Now consider calling proc_syscall() case

### scheduler.c

