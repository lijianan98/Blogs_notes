[scribble_w04a.pdf (naizhengtan.github.io)](https://naizhengtan.github.io/23fall/notes/scribble_w04a.pdf)
PLIC      -- machine external interrupt  \
CLINT   -- machine software interrupt -- Core
			--   machine timer interrupt     /


a) mstatus Machine Status, holds the global interrupt enable, along with a plethora of other state. b) mie Machine Interrupt Enable, lists which interrupts the processor can take and which it must ignore 
c) mcause Machine Exception Cause, indicates which exception occurred 
d) mtvec Machine Trap Vector, holds the address the processor jumps to when an exception occurs 
e) mepc Machine Exception PC, points to the instruction where the exception occurred 
f) mtval Machine Trap Value, holds additional trap information: the faulting address for address exceptions, the instruction itself for illegal instruction exceptions, and zero for other exceptions 
g) mip Machine Interrupt Pending, lists the interrupts currently pending
## 3 steps

### Step 1 Register interrupt handler
mtvec register (32 bit) 
bit 31-2: BASE
bit 0&1:  
	0 => direct mode, all exceptions set pc = BASE;  
	1 => vectored mode, BASE + 4*cause(what is cause???)
``` c
    asm("csrw mtvec, %0" ::"r"(handler));
```
### Step 2 Set timer
CLINT(Core-local Interrupt) memory mapped:
1. mtime and mtimecmp, mtime > mtimecmp => trigger timer interrupt
2. msip (machine software interrupt)

### Step 3 Enable timer interrupt
mstatus register: 
	The mstatus register keeps track of and controls the hart’s current operating state
	Global interrupt-enable bits, MIE and SIE, are provided for M-mode and S-mode respectively
	MPP and MPIE bits indicate previous privilege and interrupt enable state, which will be used when mret is called
	An MRET or SRET instruction is used to return from a trap in M-mode or S-mode respectively. When executing an xRET instruction, supposing xPP bit holds the value y, xIE is set to xPIE; the privilege mode is changed to y; xPIE is set to 1; and xPP is set to the least-privileged supported mode (U if U-mode is implemented, else M). If xPP6=M, xRET also sets MPRV=0

mie and mip register: Machine Interrupt-Pending/Enable Register:
	An interrupt i will trap to M-mode (causing the privilege mode to change to M-mode) if all of the following are true: 
		(a) either the current privilege mode is M and the MIE bit in the mstatus register is set, or the current privilege mode has less privilege than M-mode; 
		(b) bit i is set in both mip and mie;
		(c) if register mideleg exists, bit i is not set in mideleg.
	Interrupt cause number i (as reported in CSR mcause, Section 3.1.15) corresponds with bit i in both mip and mie.
	
mcause register(MXLEN-bit length): 
	When a trap is taken into M-mode, mcause is written with a code indicating the event that caused the trap. Otherwise, mcause is never written by the implementation, though it may be explicitly written by software.
	bit MXLEN-1: 1 => interrupts,  0 => exceptions

code example:
``` c
int mstatus, mie;
    asm("csrr %0, mstatus" : "=r"(mstatus));
    asm("csrw mstatus, %0" ::"r"(mstatus | 0x8));
    asm("csrr %0, mie" : "=r"(mie));
    asm("csrw mie, %0" ::"r"(mie | 0x80));
```