### Project implementation: 
ULT is cooperative scheduling, not preemptive
pre-emptive implementation needs signal, which adds much more complexity

#### ULT details:
1. manages TCB explicitly
2. how to create stack for new ult,  how to switch stack => create stack on heap, user can specify size of stack; switch stack needs to save callee-saved registers on old stack, store "sp" to TCB of old ult, switch to new ult's stack, restore previously stored called-saved registers on stack
3. details:
	thread control block. Usually, TCB will at least contain:
- thread id: a unique identifier of a thread
- status: the thread’s current execution state
- a function pointer: the “main” function of a thread
- function arguments: argument(s) of the thread’s “main” function
- stack pointer: the top of the thread’s execution stack
- _program counter (*)_: the address of the next instruction to be executed
- _registers (*)_: general-purpose registers that store data used by the thread

4. heap is shared as kernel thread does, all user-level threads spawned of one process live in same address space
[scribble_w03a.pdf (naizhengtan.github.io)](https://naizhengtan.github.io/23fall/notes/scribble_w03a.pdf)

5. thread_init(), thread_create(), ctx_entry(), thread_yield(), thread_exit()
	2 core methods implemented in assembly: 
	ctx_switch(void \*\*old_ptr, void \*new_ptr)
	ctx_start(void \*\*old_ptr, void \*new_ptr)
	Note that directly return from a thread will cause exceptions since there's no previous "function call" to this thread, therefore we need to explicitly call ctx_switch() method and let next thread cleanup previous thread if it marked as terminated

#### caller-saved vs callee saved (ABI, alias is call-clobbered and call-preserved):
	=> ... store temp values and ... store long-lived values
In RISC-V, callee saved registers includes: s0-s11, ra and so on
[assembly - What are callee and caller saved registers? - Stack Overflow](https://stackoverflow.com/questions/9268586/what-are-callee-and-caller-saved-registers)


