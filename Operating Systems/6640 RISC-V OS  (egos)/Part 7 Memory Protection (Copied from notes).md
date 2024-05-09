
1. Memory protection, the problem statement
2. Segmentaion (x86-32)
3. PMP (RISC-V)
4. Paging (brief)
5. Meltdown & its consequences
6. Other possible solutions
---

0. last time

  * privilege levels: U/S/M-mode
  * differences between levels
    - instructions
    - registers
    - memory regions
  * by privilege levels, we can build trap-and-emulate hypervisor


1. Problem statement

  Q: Motivation? why do we need memory protection?
    * avoid a process access kernel's memory
    * avoid a process access another process's memory
    * avoid a process accidentally modifying its own memory (bugs)
    * what else?

  - the fundamental security problem: access control

      subj --[access]--> obj

    for us,
      CPU --[op]--> memory

    Concretely,
        subj: instructions
              + context (privilege levels and others)
        p: r/w/x
        obj: memory address

    Q: is this the only way to formulate memory protection?

    Q: Why "op" only has r/w/x?

      This is defined by memory abstraction:
      -- data cells, identified by addresses
      -- ops: read/write
      -- instructions are part of the memory

   Now, memory protection is a yes/no question:
       allow [instruction+ctx] --[r/w/x]-->[a set of memory addresses]?

   Q: what are invalid accesses?
    -- invalid subj (e.g., user program reading kernel memory)
    -- invalid op (e.g., execute 0xdeadbeef)
    -- invalid obj (e.g., writing to a readonly memory)

    a straightforward solution:
      associate a ACL to obj
      => for each memory obj, attach a ACL of which subj can do what op

    Multiple questions:
    Q1: what are memory objs? (memory granularity)
    Q2: who is the subj? (how to define subj)
    Q3: where to store the ACL?


2. x86 segmentation

  - segfault example:

      In x86:
      """
      int main() {
          int *ptr = (int *)0xdeadbeef;
          *ptr = 1;
      }
      """

      You will see something like:
      "zsh: segmentation fault  ./segfault"

    Q: Why it is called segmentation fault?

  - x86 memory segmentation, a brief history

    * Segmentation was introduced on the Intel 8086 in 1978 as a way to allow
      programs to address more than 64KB of memory.
      // real mode
      // 16bit => 20bit
      // address = (base << 4) | offset

    * The Intel 80286 introduced a second version of segmentation in 1982 that
      added support for memory protection.

    * x86-32 (starting from 80386, 32-bit, 1985)

      How it works:
        * registers---CS, DS, SS, ES (FS and GS)---contain "selectors"
        * selectors point to GDT/LDT (global/local descriptor table)
            which has "descriptors"
        * descriptor contains base address, limit, access right
        * finally, the CPU will combine the requested
           address with the base address

      memory protection (checks):
        - check access right
        - check privilege levels:

        max(CPL, RPL) <= DPL
        // in x86, lower is more privileged

        segment selector (RPL)
        segment descriptor (DPL)
        current privilege level (CPL)

        [show how Linux uses segment]

    * The x86-64 architecture, introduced in 2003, has largely dropped support
      for segmentation in 64-bit mode.


  - How does segmentation answer the previous questions?

    Q1: what is the granularity of memory obj?
    A: defined by segment's base and limit, a dynamic region of memory.

    Q2: how to define the subj?
    A: defined by instruction

    Q3: where to store the ACL?
    A: registers (selectors), memory (descriptors in LDT/GDT)

3. PMP

    * PMP configuration registers (pmpcfg0–pmpcfg15), 16 of them
    * PMP address register (pmpaddr0–pmpaddr63), 64 of them

    Mapping:
      * each pmpaddr is associated with one pmpcfg;
      * one pmpcfg maps to four pmpaddr reigsters

  - How does PMP answer the previous questions?

    Q1: what is the granularity of memory obj?
    A: defined by PMP address registers; dynamically defined

    Q2: how to define the subj?
    A: defined by instruction

    Q3: where to store the ACL?
    A: PMP registers (pmpcfg0–pmpcfg15, pmpaddr0–pmpaddr63)


4. paging (brief)

   - super brief history

     * designed for swapping in/out memory (1962)
       [T Kilburn, R B Payne, D J Howarth, The Atlas Supervisor
       http://www.chilton-computing.org.uk/acl/technology/atlas/p019.htm]

    * later, adding memory protections

   - Paging is the fundamental source of isolation
     for almost all today's systems

  - How does paging answer the previous questions?

    Q1: what is the granularity of memory obj?
    A: a page (4KB), a fixed size

    Q2: how to define the subj?
    A: defined by (1) instruction, (2) privileged levels,
       and (3) root of the page table (cr3 in x86; satp in RISC-V)

    Q3: where to store the ACL?
    A: memory (page table entries)


5. Meltdown & its consequences

  * Meltdown
     allows malicious user code to read kernel memory, despite page protections.
     surprising and disturbing
     one of a number of recent "micro-architectural" attacks
       exploiting hidden implementation details of CPUs
     fixable, but people fear an open-ended supply of related surprises

  * will skip the technical details

  * consequences:

    what OS used to do: put itself in the address space of user processes'
    but isolate itself by using privilege levels (set "user-cannot-read" bit on
    page table entries)

    now, a bummer: user can violate isolation and access kernel memory

    a fix (simplified): run OS in its own memory
      [something deep here;]

    consequences: performance

    [see paper, An Analysis of Performance Evolution of Linux's Core Operations, SOSP'19
        https://dl.acm.org/doi/10.1145/3341301.3359640]


6. Capability-based processor (a wild idea)

  Another way to solve access control problem (hence, memory protection) is capability.

  High-level idea:
    a subj has a set of capabilities.
    the subj can only access the obj when it has the corresponding capability.


  How to build such a capability-based processor?
  Here is a proposal:
    - the capabilities are crytographic keys
    - the processor will have hardware encryption/decryption
    - has two registers that store read and write capabilities
    - memory are encrypted by the write capability (private key) of the owner process
    - a process want to reads a memory need the owner's read capability (public key)