![[1699151843860.png]]
### 背景：KASLR（Kernel Address Space Layout randomization）
[Address space layout randomization - Wikipedia](https://en.wikipedia.org/wiki/Address_space_layout_randomization#KASLR)
#### 大致作用
In order to prevent an attacker from reliably jumping to, for example, a particular exploited function in memory, ASLR randomly arranges the [address space](https://en.wikipedia.org/wiki/Address_space "Address space") positions of key data areas of a [process](https://en.wikipedia.org/wiki/Process_(computer_science) "Process (computer science)"), including the base of the [executable](https://en.wikipedia.org/wiki/Executable "Executable") and the positions of the [stack](https://en.wikipedia.org/wiki/Stack-based_memory_allocation "Stack-based memory allocation"), [heap](https://en.wikipedia.org/wiki/Dynamic_memory_allocation "Dynamic memory allocation") and [libraries](https://en.wikipedia.org/wiki/Library_(computer_science) "Library (computer science)")

Kernel address space layout randomization (KASLR) enables address space randomization for the Linux kernel image by randomizing where the kernel code is placed at boot time.[[21]](https://en.wikipedia.org/wiki/Address_space_layout_randomization#cite_note-21)

### 那么为什么要KPTI（解决了什么问题）？
Despite prohibiting access to these kernel mappings, it turns out that there are several [side-channel attacks](https://en.wikipedia.org/wiki/Side-channel_attack "Side-channel attack") in modern processors that can leak the location of this memory, making it possible to work around KASLR.

Side channel attacks 比较出名的有：
cache attack, timing attack，看一看如下的meltdown例子，由于乱序执行，攻击者可以偷到unauthorized address的data，因为会被缓存在cache里，尽管用户代码并不能真的读写这些地址。
https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability)#:~:text=Meltdown%20exploits%20a,a%20readable%20result
有哪些side-channel attack
https://en.wikipedia.org/wiki/Side-channel_attack#:~:text=General%20classes%20of,see%20examples%20below).

### 如何解决：
KPTI fixes these leaks by separating user-space and kernel-space page tables entirely. One set of page tables includes both kernel-space and user-space addresses same as before, but it is only used when the system is running in kernel mode. The second set of page tables for use in user mode contains a copy of user-space and a minimal set of kernel-space mappings that provides the information needed to enter or exit system calls, interrupts and exceptions

