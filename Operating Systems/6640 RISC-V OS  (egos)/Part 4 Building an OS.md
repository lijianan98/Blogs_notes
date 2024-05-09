Step #3: understand computer architecture 
	• memory layout => Read RISC-V and SiFive Documents Manual 
	• running a program 
	• calling convention 
Step #2: understand interrupt and exception 
	• void handler()  __attribute__((interrupt)); (tells compiler to store all registers at the beginning of handler instead of just callee-saved)
	• use mret and mepc instead of ret and ra
Step #1: understand context-switch

### monolithic vs micro vs exo vs multi kernel

monolithic: high in coupling, a single piece of code serving all requests

micro: kernel only responsible for IPCs, scheduling. services running in user-level as separate processes, IPC can be bottleneck

exo: kernel only handles multiplexing resources, extra performance due to having hardware primitives, seldom use in  practice

multi: target heterogenous hw and many-core machines, shared-memory => shared-nothing(msg passing) model, treat OS as distributed system