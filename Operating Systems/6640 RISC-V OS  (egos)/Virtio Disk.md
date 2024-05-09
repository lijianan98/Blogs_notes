Interrupt comes:
=> mtvec = trap_handler (store and then restore relevant registers, which is ensured by compiler)
=> ctx_entry()
=> trap_entry() or excp_entry()
=> maybe proc_yield() choose next process
=> manage_userstack(0) (restore stack)
... cp stk
=> now problem is what if choose same process???

Below is where bug happened:
(on stack) address of b and b->disk:
0x84001710, 0x84001718

Root cause, interrupt handler modified on-stack flag while in context switching, this value got overwritten. 

Solution: do not use on-stack variable as important flag

