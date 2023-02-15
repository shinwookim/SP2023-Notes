# Scheduling
> How to choose which of the *ready processes*/*threads* gets to *run* next

## CPU Bound vs. I/O Bound Processes
Nearly all processes alternate bursts of computing with I/O requests; the CPU runs for a while without stopping, then a system call is made to read or write to a file. When the system call completes, the CPU computes again until the next I/O request.

Suppose we measure a process by the amount of wall-clock time elapsed from start to termination (*elapsed real time*). If a process uses most of that time running instructions, we call it **CPU bound**. Conversely, if a process spends most of the time blocked (from blocking system calls), we call it **I/O bound**.
![](CPU-IO-Bound-Scheduling.png)
Now, if we wanted to run two processes, a naïve approach may be to run the processes sequentially. However, this approach will double our run-time. However, since I/O bound processes spend most of their time *blocked*, we might try to interweave the two processes to reduce run-time.

For instance, suppose a single I/O bound process requires 1 unit time to complete. During that time, the actual time spent doing computation might only be 0.3 units. Now, if we were to interweave an identical process, we might be able to complete both processes in 1 unit time if we run the computation for the second process while the orignal process is blocked (this is much faster than running them sequentially which would require 2 unit time). Note that there is overhead, however, as a result of the necessary context switches between the two processes. 

But what about for CPU bound processes? Again, by recognizing that different process use the same amount of time differently, we can attempt to interweave processes to gain performance benefits. However, for CPU bound processes, the time they spend blocked is minimal and thus our performance benefits will also be less (especially once we factor in overhead from context switches and scheduling). 

### Premption and its effect on pro




















But, processes can stop for other reasons.....preemption!
How long should preemption be?

Worst-case scenario: We preempt right before the blocking system call...when we return, we block again (almost immediately). (Now, 2 context switches)

context switches from IO is natural (we make it because we need it)

Preemption context switches is unnatural (overhead on top of OS-another overhead); As a process, we don't need it. It's there to maintain the illusion of parallelism (the users want. but the program do not.)

If the preemption timer is set so far in the future, that we block first, the preemption timer does not go off. (If we block before, timer, no need to switch)

If we give up the CPU time, the preemption time does not go off.

Hence, Usually I/O bound processes are unaffected by the preemption timer (we would set the preemption timer to be way longer)

CPU bound processes are however preempted. Hence, if we interleave two process for CPU bound, there are costs associated with the preemption context switches (enough to make the total performance worse than batch running two processes say 2.1 units)

---
For our goals, We hope that the vast majority of process are I/O bound. (and they are!)

We can make programs faster by using faster hardware (processes)
- With more transistors, we can put more resources on the CPU.
- With more transistors, we can have more cores (processing elements).
- However, Moore's Law is dead :(
^ Historical solution to making programs faster


If we have a CPU that runs twice the speed, the CPU bound process runs around half the time (about).
If we have a CPU that runs twice the speed, the I/O bound process does not in half the time (File I/O still take a long time).
- E.g., HDD can only spin so fast (7200RPM for decade)

With better CPUs, we can only scale improvements by CPU bound parts. If we can continuously improve a CPU-bound program (until CPU bound becomes faster and faster),  then the bottlenecks become I/O → Hence, CPU bound programs become I/O bound programs. (Happens because CPUs becomes faster much faster than other IO peripherals-linear vs. exponential);

Why are I/O bound programs so common? Human Interaction = I/O Bound!

Hence, we now beleive I/O bound programs are common --> Let's interweave them.


---


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
