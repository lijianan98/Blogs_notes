(EGOS initial policy) RR: Round-Robin
(EGOS new policy) MLFQ: Multi-level Feedback Queue

in egos design, loop &; ls and we will see ls responds after a long time, which is because current RR design lets loop & process and other system processes and short-lived process ls live in the same queue, and every time loop & sleeps for at least one quantum(timer interrupt setting) when it is scheduled.
