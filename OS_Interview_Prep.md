# Operating Systems — Interview Preparation

## 1. What is an Operating System?
System software that acts as an intermediary between users/applications and computer hardware, managing resources (CPU, memory, I/O devices, files) and providing services like process management, memory management, and security.

## 2. Functions of an OS
Process management, memory management, file management, I/O device management, security and access control, and providing a user interface (CLI/GUI).

## 3. Process vs Program vs Thread
- **Program**: a passive set of instructions stored on disk (static).
- **Process**: an active instance of a program in execution, with its own memory space (code, data, stack, heap), Process Control Block (PCB).
- **Thread**: the smallest unit of execution within a process; threads within the same process share memory/resources but have their own stack and registers, enabling concurrent execution (lightweight compared to processes).

## 4. Process Control Block (PCB)
A data structure maintained by the OS for each process, containing process ID, process state, program counter, CPU registers, memory management info, and I/O status.

## 5. Process States
New → Ready → Running → Waiting/Blocked → Terminated. A process moves between Ready and Running via scheduling, and to Waiting when it needs I/O.

## 6. Process Scheduling Algorithms
- **FCFS (First Come First Serve)**: processes executed in arrival order; simple but can cause the convoy effect.
- **SJF (Shortest Job First)**: picks the process with the smallest burst time; minimizes average waiting time but can cause starvation of long processes.
- **Priority Scheduling**: process with highest priority runs first; can cause starvation (solved via aging).
- **Round Robin**: each process gets a fixed time quantum in a cyclic order; fair, good for time-sharing systems, performance depends on quantum size.
- **Multilevel Queue Scheduling**: processes divided into queues based on priority/type, each queue can have its own algorithm.
- **Multilevel Feedback Queue**: like multilevel queue but allows processes to move between queues based on behavior/aging.

## 7. Context Switching
The process of saving the state (PCB) of a currently running process and loading the state of another process so the CPU can switch between them — enables multitasking but incurs overhead (pure switching time, no useful work done).

## 8. Threads: User-level vs Kernel-level
- **User-level threads**: managed by a user-level library without kernel awareness; fast context switching but the whole process blocks if one thread blocks (no true parallelism on multicore).
- **Kernel-level threads**: managed directly by the OS; slower context switching but allows true parallelism and independent scheduling.

## 9. Multithreading Models
Many-to-One, One-to-One, Many-to-Many — describe how user threads map to kernel threads.

## 10. Concurrency Concepts
- **Race Condition**: outcome depends on the timing/interleaving of concurrent processes/threads accessing shared data.
- **Critical Section**: the part of code where shared resources are accessed; must be protected to prevent race conditions.
- **Mutual Exclusion**: ensuring only one process/thread accesses the critical section at a time.

## 11. Critical Section Problem — Requirements
A correct solution must satisfy: **Mutual Exclusion**, **Progress** (no indefinite postponement when the resource is free), and **Bounded Waiting** (limit on how long a process waits).

## 12. Synchronization Tools
- **Mutex (Lock)**: binary flag ensuring only one thread accesses a resource at a time.
- **Semaphore**: an integer variable used for signaling between processes; **Binary Semaphore** (0/1, like a mutex) and **Counting Semaphore** (controls access to a resource pool with multiple instances). Operations: wait (P/decrement) and signal (V/increment).
- **Monitors**: a higher-level synchronization construct that encapsulates shared data and the procedures that operate on it, ensuring only one process executes inside at a time (used in Java's `synchronized`).

## 13. Classical Synchronization Problems
- **Producer-Consumer Problem**: producers add to a shared buffer, consumers remove from it; needs synchronization to avoid overflow/underflow and race conditions.
- **Readers-Writers Problem**: multiple readers can read simultaneously, but writers need exclusive access.
- **Dining Philosophers Problem**: illustrates deadlock and resource-sharing issues among concurrent processes.

## 14. Deadlock
A situation where a set of processes are blocked because each is waiting for a resource held by another in the set, forming a cycle.

### 14.1 Necessary Conditions for Deadlock (Coffman conditions)
Mutual Exclusion, Hold and Wait, No Preemption, Circular Wait — all four must hold simultaneously for deadlock to occur.

### 14.2 Deadlock Handling Strategies
- **Prevention**: negate one of the four necessary conditions.
- **Avoidance**: use algorithms like Banker's Algorithm to allocate resources only if the system remains in a safe state.
- **Detection and Recovery**: allow deadlock to occur, detect it (via resource allocation graph/wait-for graph), then recover by killing/preempting processes.
- **Ignorance (Ostrich Algorithm)**: assume deadlocks are rare and don't handle them (used by many general-purpose OSs).

## 15. Banker's Algorithm
A deadlock-avoidance algorithm that simulates resource allocation to determine if granting a request would leave the system in a "safe state" (there exists some sequence of process completions without deadlock).

## 16. Memory Management
- **Contiguous Allocation**: each process is allocated a single contiguous block of memory (fixed or variable partitioning).
- **Paging**: memory divided into fixed-size frames and processes into pages; eliminates external fragmentation but can cause internal fragmentation. Page table maps logical to physical addresses.
- **Segmentation**: memory divided into variable-sized logical segments (code, data, stack); mirrors the logical view of a program but can cause external fragmentation.

## 17. Fragmentation
- **Internal Fragmentation**: wasted space within an allocated block because the process doesn't fully use it (common in fixed-size paging).
- **External Fragmentation**: free memory is split into small non-contiguous blocks, so even if total free memory is enough, no single block is large enough for a request (common in variable partitioning/segmentation).

## 18. Virtual Memory
A technique that allows execution of processes that may not be completely in physical memory, using disk space to simulate additional RAM. Uses paging/segmentation with a page table and handles page faults (bringing the needed page into memory when it's not present).

## 19. Page Replacement Algorithms
- **FIFO**: replaces the oldest loaded page; can suffer from Belady's Anomaly (more frames leading to more page faults).
- **LRU (Least Recently Used)**: replaces the page not used for the longest time; approximates optimal but has overhead.
- **Optimal**: replaces the page that won't be used for the longest time in the future; theoretical best, not implementable in practice (needs future knowledge).
- **LFU (Least Frequently Used)**: replaces the least frequently accessed page.

## 20. Thrashing
A condition where the system spends more time swapping pages in/out than executing actual processes, due to over-committed memory and high degree of multiprogramming — causes severe performance degradation.

## 21. File System
Manages how data is stored, organized, named, and retrieved on secondary storage. Includes concepts like file allocation methods (contiguous, linked, indexed), directory structures, and file attributes/permissions.

## 22. Disk Scheduling Algorithms
- **FCFS**: serves requests in arrival order.
- **SSTF (Shortest Seek Time First)**: serves the closest request first; can cause starvation.
- **SCAN (Elevator)**: disk arm moves in one direction, servicing requests, then reverses.
- **C-SCAN**: like SCAN but only services requests in one direction, then jumps back to the start.
- **LOOK/C-LOOK**: like SCAN/C-SCAN but only goes as far as the last request, not the disk end.

## 23. Belady's Anomaly
A phenomenon where increasing the number of page frames can actually increase the number of page faults (can occur with FIFO, not with LRU/Optimal).

## 24. Interrupts and System Calls
- **Interrupt**: a signal from hardware/software that temporarily halts CPU execution to handle an urgent event.
- **System Call**: a programmatic way for a user process to request a service from the OS kernel (e.g., file operations, process creation) — transitions from user mode to kernel mode.

## 25. Kernel vs User Mode
**Kernel mode** has unrestricted access to hardware/system resources; **User mode** is restricted and must use system calls to request OS services — this separation protects system stability and security.

## 26. Monolithic vs Microkernel
- **Monolithic Kernel**: entire OS (device drivers, file system, etc.) runs in kernel space as a single program — fast but less modular/stable (a bug can crash the system).
- **Microkernel**: only essential services (IPC, basic scheduling, memory management) run in kernel space; other services run in user space — more stable/modular but slower due to increased context switching/IPC.

## 27. Spooling vs Buffering
- **Buffering**: temporary storage area to handle speed mismatch between producer and consumer of data.
- **Spooling**: uses disk as a large buffer to hold data for devices like printers, allowing jobs to be queued (Simultaneous Peripheral Operations OnLine).

## 28. Semaphore Deadlock/Starvation
Improper use of semaphores (e.g., wrong ordering of wait/signal) can itself cause deadlock or starvation, so care is needed in design.

## 29. Cache Memory
A small, fast memory located close to the CPU that stores frequently accessed data to reduce average access time, based on the principle of locality of reference (temporal and spatial).

## 30. Bootstrapping (Booting Process)
The process of loading the operating system into memory when a computer starts, initiated by firmware (BIOS/UEFI) executing the bootloader.

---

## 31. Paging in Detail
- Logical memory is split into fixed-size **pages**; physical memory into equally sized **frames** (commonly 4 KB).
- A **page table** per process maps page number → frame number. A logical address is split into (page number, offset).
- **Translation**: page number indexes the page table to get the frame number; the offset is appended unchanged.
- **TLB (Translation Lookaside Buffer)** is a small associative cache of recent page-table entries. A TLB hit avoids a memory access. Effective access time = hit ratio × (TLB time + memory time) + miss ratio × (TLB time + 2 × memory time).
- **Multi-level paging** avoids storing one huge flat table — a 32-bit address space with 4 KB pages needs 2^20 entries, so the table itself is paged.
- **Inverted page table** stores one entry per physical frame instead of per page, saving space at the cost of search time.
- Page table entries carry **valid/invalid, protection (read/write/execute), dirty, and reference bits**.

## 32. Segmentation vs Paging
| Paging | Segmentation |
|---|---|
| Fixed-size blocks | Variable-size, logically meaningful blocks (code, data, stack) |
| Invisible to the programmer | Visible to the programmer/compiler |
| Causes internal fragmentation | Causes external fragmentation |
| One page table per process | One segment table per process |
| Address = (page, offset) | Address = (segment, offset) |
Modern systems use **segmentation with paging**: segments are logically meaningful, and each segment is paged for physical allocation.

## 33. Demand Paging and Page Faults
Pages are loaded only when first referenced. On a reference to a page marked invalid, the hardware raises a **page fault trap**:
1. OS checks whether the reference is legal; if not, terminate the process.
2. Find a free frame (or evict one using a replacement algorithm; if the victim is dirty, write it back).
3. Schedule a disk read of the required page into that frame.
4. Update the page table and TLB.
5. Restart the instruction that faulted.

**Effective access time** = (1 − p) × memory access + p × page-fault service time, where p is the fault rate. Because a disk access is ~10^5 times slower than memory, even a tiny p dominates — which is why locality of reference matters so much.

## 34. Copy-on-Write and fork()
`fork()` creates a child process. Instead of copying the entire address space, the pages are marked read-only and shared; the first write by either process triggers a fault, and only that page is copied. This makes `fork()` followed by `exec()` cheap. `exec()` replaces the process image; `wait()` lets the parent collect the child's exit status.
- **Zombie process** — has terminated but the parent has not called `wait()`, so its PCB entry lingers.
- **Orphan process** — the parent died first; it is re-parented to `init`/`systemd`, which reaps it.

## 35. Inter-Process Communication (IPC)
- **Shared memory** — fastest; a region is mapped into both address spaces, but the processes must synchronise access themselves.
- **Message passing** — the kernel copies messages; slower but safe and works across machines.
- **Pipes** — unidirectional byte streams between related processes (`ls | grep`); named pipes (FIFOs) work between unrelated processes.
- **Sockets** — bidirectional, work over a network as well as locally (Unix domain sockets).
- **Signals** — asynchronous notifications (SIGTERM, SIGKILL, SIGINT, SIGSEGV).
- **Message queues, semaphores, shared memory** — the System V / POSIX IPC trio.

## 36. Producer-Consumer with a Semaphore (know the code shape)
```
semaphore mutex = 1, empty = N, full = 0;

Producer:                     Consumer:
  wait(empty);                  wait(full);
  wait(mutex);                  wait(mutex);
  add item to buffer;           remove item from buffer;
  signal(mutex);                signal(mutex);
  signal(full);                 signal(empty);
```
The order matters: taking `mutex` before `empty`/`full` deadlocks.

## 37. Scheduling Metrics and a Worked Example
- **Arrival time** — when the process enters the ready queue.
- **Burst time** — CPU time it needs.
- **Completion time** — when it finishes.
- **Turnaround time** = completion − arrival.
- **Waiting time** = turnaround − burst.
- **Response time** = first time it runs − arrival.

**Example** — processes P1 (burst 5), P2 (burst 3), P3 (burst 8), all arriving at t=0.

FCFS in order P1, P2, P3:
| Process | Completion | Turnaround | Waiting |
|---|---|---|---|
| P1 | 5 | 5 | 0 |
| P2 | 8 | 8 | 5 |
| P3 | 16 | 16 | 8 |
Average waiting time = (0 + 5 + 8) / 3 = 4.33.

SJF (P2, P1, P3):
| Process | Completion | Turnaround | Waiting |
|---|---|---|---|
| P2 | 3 | 3 | 0 |
| P1 | 8 | 8 | 3 |
| P3 | 16 | 16 | 8 |
Average waiting time = (0 + 3 + 8) / 3 = 3.67 — SJF is provably optimal for average waiting time.

Round Robin with quantum 2 gives a worse average waiting time but far better response time. That is the whole trade-off: throughput versus responsiveness.

## 38. Preemptive vs Non-Preemptive Scheduling
Non-preemptive: once a process gets the CPU it keeps it until it blocks or finishes (FCFS, plain SJF). Simple, no race conditions from a forced switch, but a long job blocks everyone (convoy effect). Preemptive: the OS can take the CPU back on a timer interrupt or when a higher-priority process arrives (Round Robin, SRTF, preemptive priority). Better responsiveness and required for time-sharing and real-time systems, at the cost of context-switch overhead and the need for synchronisation.

## 39. Real-World Linux Details Worth Knowing
- **Process states in Linux**: R (running/runnable), S (interruptible sleep), D (uninterruptible sleep, usually I/O), Z (zombie), T (stopped).
- **nice value** ranges −20 (highest priority) to +19 (lowest). Only root or a process with `CAP_SYS_NICE` can lower it below 0. My **ModeOS** project uses exactly this to boost or de-prioritise applications without root.
- **Completely Fair Scheduler (CFS)** is the default Linux scheduler; it tracks virtual runtime per task and always runs the one with the least, giving proportional fairness rather than fixed time slices.
- **`/proc`** is a virtual filesystem exposing kernel and process state as files — `/proc/<pid>/status`, `/proc/meminfo`, `/proc/cpuinfo`.
- **Signals**: SIGTERM (15) asks a process to exit and can be caught or ignored; SIGKILL (9) cannot be caught and is delivered by the kernel. ModeOS deliberately sends SIGTERM first, waits a 2-second grace period, and only escalates to SIGKILL — that is the standard, safe two-stage shutdown.
- **OOM killer** — when memory is exhausted, the kernel picks a victim by score and kills it.

## 40. Virtualisation and Containers (commonly asked follow-up)
- A **virtual machine** runs a full guest OS on a hypervisor (Type 1 runs on bare metal, Type 2 on a host OS). Strong isolation, heavy.
- A **container** shares the host kernel and isolates processes using **namespaces** (PID, network, mount, user, UTS, IPC) and limits resources using **cgroups**. Much lighter, starts in milliseconds, but weaker isolation because the kernel is shared.
- Docker images are layered and immutable; a container is a writable layer on top.

---

# Practice Questions with Answers (Operating Systems)

### Conceptual / Definition-based

**1. What is an operating system and what are its main functions?**
System software that sits between hardware and applications. It is a **resource manager** (allocates CPU, memory, I/O, and storage) and an **extended machine** (hides hardware detail behind clean abstractions like processes, files, and sockets). Core functions: process management, memory management, file system management, I/O device management, security and protection, and providing an interface (shell or GUI).

**2. Explain the difference between a process and a thread.**
A process is an executing program with its own address space, file descriptors, and PCB — creating one is expensive and processes are isolated from each other. A thread is a unit of execution inside a process; threads share code, data, heap, and open files but each has its own stack, registers, and program counter. Threads are cheap to create and switch between, and communication between them is trivial because memory is shared — which is exactly why they need synchronisation.

**3. What is a PCB and what does it contain?**
The Process Control Block is the kernel's record of a process: process id, state, program counter, CPU registers, scheduling information (priority, pointers to queues), memory management information (page tables, base/limit registers), accounting information, and I/O status including open files. On a context switch the OS saves the running process's registers into its PCB and loads the next process's PCB.

**4. Explain process states with a diagram description.**
New (being created) → Ready (waiting for the CPU) → Running (executing) → either Terminated (finished), Waiting/Blocked (waiting for I/O or an event, then back to Ready), or back to Ready if preempted by the scheduler. The key transitions to name: Ready→Running is dispatch, Running→Ready is preemption, Running→Waiting is an I/O request, Waiting→Ready is I/O completion.

**5. What is context switching and why is it expensive?**
Saving the state of the current process into its PCB and restoring another's so the CPU can run it. It is pure overhead — no useful work happens during it. Costs: saving/restoring registers, switching the page table and flushing or tagging the TLB, and losing cache locality so the new process starts with cold caches. That last part usually dominates.

**6. What is a system call?**
The controlled entry point from user mode into kernel mode. A user program cannot touch hardware directly, so it puts a system call number in a register and executes a trap instruction; the kernel validates the arguments, performs the privileged operation, and returns. Categories: process control (`fork`, `exec`, `exit`, `wait`), file management (`open`, `read`, `write`, `close`), device management, information maintenance (`getpid`), and communication (`socket`, `pipe`).

**7. What is virtual memory?**
An abstraction that gives every process a large, private, contiguous address space regardless of how much physical RAM exists, by keeping only the actively used pages in memory and the rest on disk. Benefits: programs larger than RAM can run, more processes fit in memory so CPU utilisation rises, processes are isolated from each other, and loading is faster because only needed pages are brought in.

### Comparison-based

**8. Multiprogramming vs multitasking vs multiprocessing.**
Multiprogramming: several programs are resident in memory and the CPU switches to another whenever the current one blocks on I/O — the goal is CPU utilisation. Multitasking (time-sharing): the CPU switches on a timer as well, so users get interactive response — the goal is responsiveness. Multiprocessing: more than one physical CPU or core executes simultaneously — the goal is real parallelism.

**9. Paging vs segmentation.**
See section 32. One line: paging is a physical-memory technique with fixed blocks and internal fragmentation; segmentation is a logical technique with variable blocks and external fragmentation.

**10. Internal vs external fragmentation.**
Internal fragmentation is unused space *inside* an allocated block — a 4 KB page holding 3 KB of data wastes 1 KB. External fragmentation is unused space *between* allocated blocks — enough total free memory exists but no single contiguous chunk is big enough. Paging eliminates external fragmentation; compaction fixes it in contiguous allocation.

**11. Deadlock vs starvation.**
Deadlock: a set of processes are permanently blocked because each holds a resource the next one needs — nobody can ever proceed. Starvation: a process is runnable but keeps being passed over indefinitely, usually by priority scheduling — it *could* proceed but never gets picked. Deadlock is a circular dependency; starvation is unfairness. Aging fixes starvation; deadlock needs prevention, avoidance, or detection.

**12. Kernel mode vs user mode.**
A CPU mode bit determines which instructions are allowed. In user mode, privileged instructions (I/O, changing page tables, halting) are forbidden and attempting them traps. In kernel mode everything is allowed. Applications run in user mode and cross into kernel mode only via a system call, exception, or interrupt. Without this split, a buggy program could corrupt the entire machine.

**13. Monolithic kernel vs microkernel.**
Monolithic: all OS services (file system, drivers, networking, memory management) run in kernel space in one address space — fast because calls are function calls, but a driver bug can crash the whole system, and it is large. Microkernel: only the bare minimum (IPC, scheduling, basic memory) is in the kernel; the rest run as user-space servers — more reliable, more secure, easier to extend, but slower because of the IPC and mode-switch overhead. Linux is monolithic (with loadable modules); MINIX and QNX are microkernels; Windows and macOS are hybrids.

**14. Preemptive vs non-preemptive scheduling.**
See section 38.

**15. Thread vs process, and when to choose each.**
Use threads when tasks share a lot of data and you want low creation and communication cost — for example handling many connections in one server. Use processes when you need isolation and fault containment — a crash in one should not take down others, and a compromise in one should not read another's memory. Chrome uses a process per tab for exactly that reason. Judge0, which my ClassRoom Code project uses, runs each student submission in an isolated sandboxed process for the same reason.

**16. Spooling vs buffering.**
Buffering overlaps I/O with computation for *one* job by using a memory area between producer and consumer. Spooling (Simultaneous Peripheral Operation On-Line) queues complete jobs on disk so a slow device like a printer can be shared by many jobs, and the submitting process does not wait at all.

### Scheduling-based

**17. Compare FCFS, SJF, Priority, and Round Robin.**
FCFS: simple, fair in arrival order, non-preemptive, suffers the **convoy effect** where one long job delays everything. SJF: optimal average waiting time, but requires knowing burst times in advance (estimated by exponential averaging) and starves long jobs. Priority: important work first, but starves low-priority jobs unless **aging** gradually raises their priority. Round Robin: preemptive with a fixed quantum, excellent response time and no starvation, but higher context-switch overhead — too small a quantum wastes CPU on switching, too large a quantum degenerates to FCFS.

**18. What is the convoy effect?**
Under FCFS, one CPU-bound process at the head of the queue makes all the short, I/O-bound processes behind it wait. Those I/O devices sit idle. Then when the long process finally blocks, everyone runs briefly and queues up again. Overall device and CPU utilisation collapses. Round Robin or SJF avoids it.

**19. How do you choose the time quantum in Round Robin?**
It should be large compared with the context-switch time (a rule of thumb is that 80% of bursts should be shorter than the quantum) but small enough that response time stays acceptable. Typical values are 10–100 ms against a context switch of tens of microseconds.

**20. What is a multilevel feedback queue?**
Several ready queues with different priorities and quanta. A new process enters the top queue; if it uses its whole quantum it is demoted to a lower-priority queue with a longer quantum; if it blocks on I/O it stays or is promoted. This automatically separates interactive jobs (short bursts, stay high priority) from CPU-bound jobs, without being told which is which. Aging promotes starved processes back up.

### Synchronization / Deadlock-based

**21. What is the critical section problem and what must a solution guarantee?**
A critical section is code that accesses shared data; concurrent execution of critical sections causes race conditions. Any solution must guarantee: **mutual exclusion** (at most one process inside at a time), **progress** (if no process is inside, the choice of who enters next cannot be postponed indefinitely by processes not wanting to enter), and **bounded waiting** (a limit on how many times others can enter before a waiting process gets its turn).

**22. Explain semaphores, and the difference between binary and counting.**
A semaphore is an integer with two atomic operations: `wait()`/`P` decrements and blocks if the value would go negative, `signal()`/`V` increments and wakes a waiter. A **binary semaphore** takes values 0 and 1 and acts as a lock. A **counting semaphore** takes any non-negative value and controls access to N identical resources — for example 5 permits for 5 printers. Crucially, semaphores do not have ownership, so any thread can signal one.

**23. Difference between a mutex and a semaphore.**
A mutex is a locking mechanism with **ownership** — only the thread that locked it may unlock it — and is used purely for mutual exclusion. A binary semaphore is a signalling mechanism with no ownership, so one thread can wait and a different thread can signal, which is what makes it usable for producer-consumer ordering. Mutexes typically support priority inheritance to avoid priority inversion; semaphores do not.

**24. What is a race condition? Give an example.**
When the result depends on the unpredictable interleaving of concurrent operations. Classic example: two threads both execute `counter++`, which is really load, increment, store. If both load 5 before either stores, both store 6 and one increment is lost. The fix is to make the sequence atomic with a lock, or use an atomic instruction.

**25. What is a deadlock and what are the four necessary conditions?**
All four must hold simultaneously: **mutual exclusion** (a resource cannot be shared), **hold and wait** (a process holds one resource while requesting another), **no preemption** (a resource cannot be forcibly taken), and **circular wait** (a cycle of processes each waiting on the next). Breaking any one prevents deadlock.

**26. How do you handle deadlock?**
Four approaches. **Prevention** — structurally deny one of the four conditions, for example impose a global ordering on resource acquisition to break circular wait, or require all resources up front to break hold-and-wait. **Avoidance** — grant a request only if the resulting state is safe (Banker's algorithm), which needs advance knowledge of maximum needs. **Detection and recovery** — let it happen, run a wait-for-graph cycle detection periodically, then kill a process or roll back. **Ignore it** — the ostrich algorithm, which is what general-purpose operating systems actually do, because deadlock is rare and the cost of prevention is high.

**27. Explain Banker's algorithm.**
A deadlock-avoidance algorithm. Each process declares its maximum need in advance. The system tracks Allocation, Max, and Need = Max − Allocation, plus Available. When a request arrives it is granted only if the resulting state is **safe** — meaning there exists an ordering of all processes such that each can obtain its remaining need from what is available plus what earlier processes release. The safety check simulates that sequence. If no such order exists, the requester waits. It is rarely used in practice because processes seldom know their maximum needs.

**28. Explain the dining philosophers problem and a solution.**
Five philosophers alternate thinking and eating; each needs the two forks beside them. If all pick up their left fork at once, all deadlock. Solutions: allow at most four to sit at the table simultaneously; require picking up both forks atomically; or break the symmetry by having odd-numbered philosophers take the left fork first and even-numbered ones the right — this breaks circular wait.

**29. What is priority inversion?**
A high-priority task is blocked waiting on a lock held by a low-priority task, and a medium-priority task preempts the low-priority one — so the high-priority task waits behind a medium-priority task indefinitely. This famously nearly ended the Mars Pathfinder mission. Fixes: **priority inheritance** (the lock holder temporarily inherits the highest waiting priority) or **priority ceiling** (a lock carries a fixed priority that any holder gets).

### Memory Management-based

**30. Explain page replacement algorithms and compare them.**
**FIFO**: evict the oldest page — simple, but ignores usage and suffers Belady's anomaly. **Optimal (OPT/MIN)**: evict the page that will be used furthest in the future — provably best, but requires knowing the future, so it is only a benchmark. **LRU**: evict the least recently used page — a good approximation of OPT based on locality, but exact LRU needs a counter or stack per access, which is expensive, so real systems approximate it with a **reference bit and the second-chance/clock algorithm**. **LFU**: evict the least frequently used, which mishandles pages that were hot once and are now dead.

**31. What is Belady's anomaly?**
Increasing the number of frames increases the number of page faults — counterintuitive but real for FIFO. It happens because FIFO's victim choice does not respect the "stack property": the set of pages held with n frames is not necessarily a subset of the set held with n+1 frames. LRU and OPT are stack algorithms and are therefore immune.

**32. What is thrashing, and how do you fix it?**
A process spends more time paging than executing because it does not have enough frames to hold its working set. Every instruction faults, CPU utilisation drops, the scheduler assumes it needs more processes and admits more, which makes it worse — a positive feedback loop. Fixes: the **working set model** (give each process enough frames for the pages it referenced in the last Δ references, and suspend processes if the total demand exceeds memory) or **page fault frequency control** (if a process's fault rate is above an upper bound give it more frames, below a lower bound take some away).

**33. What is locality of reference?**
Programs do not access memory randomly. **Temporal locality**: a recently used address is likely to be used again soon (loop counters). **Spatial locality**: addresses near a recently used one are likely to be used soon (array traversal, sequential instructions). Every cache, TLB, and page-replacement heuristic exists because of this.

**34. Explain the memory allocation strategies for contiguous allocation.**
**First fit** — allocate the first hole big enough; fastest. **Best fit** — allocate the smallest hole that fits; wastes least immediately but leaves many tiny unusable holes and needs a full scan. **Worst fit** — allocate the largest hole; leaves a usefully large remainder but performs worst in practice. First fit and best fit both beat worst fit; first fit is generally fastest.

### Scenario/Application-based

**35. A system is slow. How do you diagnose whether it is CPU, memory, disk, or I/O bound?**
Start with `top`/`htop`: high `%us` means user CPU bound, high `%sy` means kernel/syscall heavy, high `%wa` means waiting on I/O, high `%si`/`%hi` means interrupt load. Then `free -h` and `vmstat` for memory pressure and swap activity (heavy `si`/`so` means thrashing). `iostat`/`iotop` for disk throughput and utilisation. `ss -s` and `netstat` for network. Check load average against the core count. If a single process dominates, look at its state — D state means it is blocked in uninterruptible I/O.

**36. Why can a multithreaded program be slower than a single-threaded one?**
Lock contention serialises the work and adds context switching; false sharing makes cores fight over the same cache line; thread creation and synchronisation have overhead; and if the task is memory-bandwidth-bound rather than CPU-bound, more threads do not help. Amdahl's law puts a ceiling on it: if a fraction s of the work is inherently serial, the maximum speedup is 1/s no matter how many cores you add.

**37. How does an OS protect one process from another?**
Hardware-enforced address translation means a process can only name addresses in its own page table. The user/kernel mode bit prevents privileged instructions. System calls validate arguments at the boundary. File permissions and user ids enforce access control. Modern additions: ASLR randomises layout, DEP/NX marks data pages non-executable, and stack canaries detect overflow.

### Miscellaneous

**38. Explain the booting process.**
Power on → **BIOS/UEFI** firmware runs POST (power-on self test) and initialises hardware → it locates a bootable device and loads the **bootloader** (GRUB on Linux) from the MBR or the EFI system partition → the bootloader loads the **kernel** and an initial RAM disk into memory and transfers control → the kernel initialises subsystems, mounts the root filesystem, and starts the first user-space process (**init/systemd**, PID 1) → systemd starts the remaining services and the login manager.

**39. What are interrupts, and how do they differ from traps?**
An **interrupt** is asynchronous and hardware-generated (a key press, a timer tick, a disk finishing). A **trap/exception** is synchronous and software-generated by the currently executing instruction (divide by zero, page fault, a deliberate system call). Both transfer control through the interrupt vector table to a handler in kernel mode, and both save the current context first. Interrupt handling is split into a fast top half and a deferred bottom half so the CPU is not blocked for long.

**40. Explain disk scheduling algorithms.**
The goal is minimising total head movement (seek time). **FCFS** — fair, but the head wanders. **SSTF** — nearest request first; good throughput but starves distant requests. **SCAN (elevator)** — head sweeps to one end servicing requests, then reverses; bounded waiting. **C-SCAN** — sweeps one way only, then jumps back without servicing, giving more uniform wait times. **LOOK / C-LOOK** — the same but reverses at the last request rather than the physical end of the disk, which is what real systems use. On SSDs seek time is irrelevant, so these matter far less and the OS optimises for write amplification and parallelism instead.

**41. What is a file system, and what does an inode contain?**
The file system organises data on storage into files and directories, and manages free space, metadata, and access control. A Unix **inode** holds file metadata: type, permissions, owner, group, size, timestamps, link count, and pointers to data blocks (direct, single/double/triple indirect). Notably it does **not** hold the file name — names live in directory entries that map a name to an inode number, which is what makes hard links possible.

**42. Explain RAID levels briefly.**
RAID 0 — striping, fast, no redundancy. RAID 1 — mirroring, full redundancy, 50% capacity. RAID 5 — striping with distributed parity, survives one disk failure, needs at least 3 disks. RAID 6 — double parity, survives two failures. RAID 10 — mirrored stripes, best performance plus redundancy, expensive.

---

# Rapid-Fire One-Liners

- **Throughput** — processes completed per unit time. **Turnaround** — submission to completion. **Response time** — submission to first response.
- **Dispatcher** — the module that actually gives the CPU to the process the scheduler chose; **dispatch latency** is how long that takes.
- **Long-term scheduler** controls the degree of multiprogramming (which jobs enter memory); **short-term** picks who runs next; **medium-term** swaps processes out to relieve memory pressure.
- **Swapping** — moving a whole process out to disk; **paging** — moving individual pages.
- **Dirty bit** — a page has been modified so it must be written back before eviction.
- **Thread pool** — reuses a fixed set of worker threads instead of creating one per task, bounding resource use.
- **Kernel-level threads** are scheduled by the OS and can run truly in parallel; **user-level threads** are cheaper but one blocking call stalls all of them.
- **Reentrant code** — can be safely executed by multiple threads simultaneously because it keeps no mutable global state.
- **Busy waiting / spinlock** — looping until a lock is free; wastes CPU but avoids context-switch cost, so it is only sensible on multiprocessors for very short critical sections.
- **Atomic instruction** — test-and-set, compare-and-swap; the hardware primitive every lock is built on.
- **Real-time OS** — correctness depends on meeting deadlines; **hard** real-time never misses one, **soft** real-time degrades gracefully.

---

# Linking Operating Systems to My Projects

- **ModeOS** is an OS project end to end: it enumerates running processes with `psutil` (reading `/proc`), changes scheduling priority with **nice values**, terminates processes with a **SIGTERM → 2-second grace → SIGKILL** two-stage sequence, protects system daemons with a whitelist so it never kills a shell or the compositor, writes to the Linux **sysfs backlight** interface, follows the **XDG base directory specification** for config, state, and cache, and persists a snapshot to disk so it can revert to the exact prior state. It also ships a Docker sandbox — containers, namespaces, and cgroups.
- **NetSpecter** requires **root privileges** because raw sockets are a privileged operation; it checks `os.geteuid() != 0` and refuses otherwise. It uses libpcap/BPF, which is kernel-level packet filtering, and keeps per-flow buffers with timeouts — essentially a small memory-management problem.
- **ClassRoom Code** runs student code in **isolated processes** with wall-clock timeouts, output caps, and process-group kills on timeout — and the README states plainly that this local runner is *not* a sandbox, which is why Judge0 (which uses `isolate`, a cgroup- and namespace-based sandbox) is the production path. That is a strong answer to "why does untrusted code need OS-level isolation?".
- **ModelAuth** uses a **thread pool** (`concurrent.futures.ThreadPoolExecutor`) to issue probes concurrently — a direct example of choosing threads over processes because the work is I/O bound.
