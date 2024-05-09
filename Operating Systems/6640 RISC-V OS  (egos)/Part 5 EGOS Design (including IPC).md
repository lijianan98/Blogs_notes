
### Earth layer:
CPU and Device interfaces:
    [read library/egos.h]
### Grass layer: 
system services
     -- sys_proc  (GPID_PROC)
     -- sys_file  (GPID_FILE)
     -- sys_dir   (GPID_DIR)
     -- sys_shell (GPID_SHELL)
three root syscalls: sys_send, sys_recv, and sys_yield
     (grass/syscall.h)

### Boot process
CPU jmps to 0x20400000
   |
   +-> earth.S:_enter
       |
       +-> earth.c:main
           |
           +-> grass.S:_enter
               |
               +-> grass.c:main
                   |
                   +-> app.S:_enter
                       |
                       +-> sys_proc.c:main
                           |
                           ...
                           |
                           +-> sys_shell.c:main

Syscall / interrupt / exception workflow:
sys_send/recv (syscall.c)
    |                                                         ^
    +-> sys_invoke (syscall.c)                              [ret]
          |                              USER SPACE           |
  -----[trap]-------------------------------------------------|----
          |                               KERNEL                          |
          +-> trap_entry (cpu_intr.c)                       [mret]
              |                                                                 |
              +-> trap_handler (kernel.c)                   [ret]
                  [switch to kernel stack]          [switch to user stack]
                  ...                                                           |
                  |                                                             |
                  +-> proc_syscall (syscall.c)                 [ret]
                      |                                                         |
                      +-> proc_send/recv (syscall.c)        [ret]
                          |                                                     |
                          +-> sys_yield (scheduler.c)         [ret]
                              |                                                 |
                              +->[context switch]         [switch back]
                                    |                                            |
                                    +-->[other proc running]--+