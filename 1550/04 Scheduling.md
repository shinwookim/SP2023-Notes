# Scheduling
> How to choose which of the *ready processes*/*threads* gets to *run* next

## CPU Bound vs. I/O Bound Processes
Nearly all processes alternate bursts of computing with I/O requests; the CPU runs for a while without stopping, then a system call is made to read or write to a file. When the system call completes, the CPU computes again until the next I/O request.

Suppose we measure a process by the amount of wall-clock time elapsed from start to termination (*elapsed real time*). If a process uses most of that time running instructions, we call it **CPU bound**. Conversely, if a process spends most of the time blocked (from blocking system calls), we call it **I/O bound**.
![](CPU-IO-Bound-Scheduling.png)
Now, if we wanted to run two processes, a naïve approach may be to run the processes sequentially. However, this approach will double our run-time. However, since I/O bound processes spend most of their time *blocked*, we might try to interweave the two processes to reduce run-time.

For instance, suppose a single I/O bound process requires 1 unit time to complete. During that time, the actual time spent doing computation might only be 0.3 units. Now, if we were to interweave an identical process, we might be able to complete both processes in 1 unit time if we run the computation for the second process while the orignal process is blocked (this is much faster than running them sequentially which would require 2 unit time). Note that there is overhead, however, as a result of the necessary context switches between the two processes. 

But what about for CPU bound processes? Again, by recognizing that different process use the same amount of time differently, we can attempt to interweave processes to gain performance benefits. However, for CPU bound processes, the time they spend blocked is minimal and thus our performance benefits will also be less (especially once we factor in overhead from context switches and scheduling). 

### Premption and its effect
We've discussed how processes may voluntarily *block* (via system calls like `read()`), but process can also stop non-voluntarily. Previous, we introduced the idea of preemption with the hopes of dealing with greedy processes. With preemption, a hardware timer sends a signal time-periodically, to *preempt* long-running processes. 

As a process, preemption is not desirable (it prevents the system from running the process's code). In fact, it exists to maintain the pseudo-parallelism for the user, but is not necessary from the process's point of view. For the process, a preemption context switch is unnatural (unlike I/O context switches which are incurred when the process calls for it). It's an additional overhead on top of the operating system, which itself is an overhead.

Indeed, in the worst case, we might preempt right before a blocking system call, which means when we return, we block again (almost immediately). This is undesirable as a system too, as we incurred 2 unncessary context switches.

Hence, setting an appropriate length for the preemption timer is important. If the timer length is too long, it won't be effective (as we would have already made a blocking system call before the preemption signal). Yet, if it is too short, we would inccur unnecessary context switches, slowing down our system performance. 

Assuming an appropriate length for the preemption timer is set, I/O bound processes are typically unaffected by the preemption timer (as they have short bursts of computation and many calls to blocking system calls). On the other hand, CPU bound process are frequently preempted. Thus, if we interweave two CPU bound processes, we now have to account for the costs associated with preemptive context switches. Empirically, this may slow the total performance enough to the point that interweaving is slower than the sequentially running the processes (batch system).

### With time, CPU bound processes become I/O bound
Thus, for our goals, we would prefer if the vast majority of the processes on our system are I/O bound (and they are!). Hence, the key question is: *Can we turn a CPU bound process into a I/O bound process?*

The traditional answer to this question was **better hardware**. With faster CPUS, we could improve the computational parts of a CPU bound process. (Note that I/O bound process typically do not reap the benefits of a better CPU because most of their time is spend waiting for I/O and I/O access speed is not proportional to CPU speed). Now, if we can continuously improve the computational performance of a CPU bound program, eventually we will reach a point where the I/O access time overtakes the computational time. And hence our CPU bound program becomes an I/O bound program and interweaving processes become reasonable.

Historically, with [Moore's Law](https://en.wikipedia.org/wiki/Moore's_law), it was reasonable to expect that CPU speeds to increase at an exponential rate. However, in the modern day, this expectation is no longer reasonable, and this approach is less accepted. Yet, still the general principle of faster hardware converting CPU bound processes to I/O bound process stil stand.

Also, as an aside, why can't we easily improve the performance of I/O bound processes? Unlike, CPUs, improvements in I/O devices have been largely stagnant. Although there certainly are some improvements (such as the development of solid state drives), a lot of I/O devices have largely stayed the same (Hard Disk Drive speeds have been the same for almost a decade!). Furthermore, many I/O in these programs represents *human interactions* (which is slow and unpredictable!).

## When to schedule
Now that we've justified the need for a scheduling, let's look at how scheduling act


When to schedule:
- Process Creation - syscall
- Process Exit - syscall
- Blocked - syscall
- I/O Interrupt - hardware interupt
- Clock Interrupt - preemption
(I.e., when are we scheduling? When we are in the OS...When are we in the OS?)

Three-Level Scheduling...where do we schedule?
![](three-level-schedule.png)
CPU schedule picks from the ready process to give the CPU time


Von Neumann architecture, we can implicitly schedule by denying process RAM. (since process needs to be RAM resident).

If we deny a process RAM, we deny that process an opportunity to run (we can't make the CPU choose it, but we can make sure it doesn't).
1. Admission Scheduler - When we are open a new program, the job goes into queue, and the OS allocates RAM. If our resources are full (e.g., RAM is full, CPU running), the admission scheduler will deny the task from running program (NOT in many modern systems ⇾ we defer that task to the user).
2. Memory Scheduler - Can we kick a process out of RAM (without killing it)--we need to preserve the RAM? on a disk... ('Swapping') The process cannot run since we can't run instructions from a disk in VN-arch
	1. If we never bring it back, we essentially killed the process (in an expensive way)
	2. Again in modern system, the memory scheduler is minimal.
Our focus will be on CPU scheduler (which is the most common in modern interactive systems)



---
Batch Scheduling
> Non-interactive jobs that can be run “overnight”
- No human interactions needed (usually)
- Run from creation to termination without blocks

Evaluate scheduling algorithm on 5 criteria:
2. Quantitative 
Throughput: Number of jobs completed per unit time. (usually fractional)
Turn around time: Time from job submission to job completion 
- In batch system, job has execution time (no preemption) + blocking is part of execution time (execution time = wall clock time)
- Turn around time includes the time not running (in ready state)3
→ Average turnaround time :average of all turnaround times for a set of all jobs
3. Computer Science Metrics
- Asymptotic Behavior (As we get more processes, how much more work does our scheduler do)
- How hard to implement? Complex algorithms with minimal improvements might not be 'worth it' (e.g, bugs?)
1. Qualitative
Fairness
> Comparable processes get comparable service
**comparable != equal**
^ This is the framework for determining fairness.

First non-preemptive batch scheduling algo:
# First come,  first served
![](Pasted%20image%2020230213103853.png)
Each process runs to completion

Analysis:
1. Throughput
$$\frac{\text{ \#jobs}}{\text{total time}}=\frac{4}{16}=0.25$$
2. ATT
$$\frac{\sum \text{turnaround}}{\text{\# jobs}}=\frac{40}{4}=10$$

Turnaround time:
A. 4 - 0 - 4
B. 7 - 0 = 7
C. 13 - 0 = 13
D.16 - 0 = 16

Asymptotic:
- Add to queue (constant time)
- Pop from front (constant time)
Hence, constant time!

Implementation:
- pretty simple

Fairness:
- Fair
	- All jobs (if you came here first, you get first service)
	- 


Fairness:
- Unfair
	- B,D
