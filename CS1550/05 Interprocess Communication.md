Threads: By sharing an address space → we make communicate easily

How do processes communicate with one another?

Race Condition
Rudimentary queue backed by an array

Since we are using parallelism → We want either 20 then 9 or 9 then 20 
If we want a specific order, we shouldn't be using parallelism
Scheduling is chaotic --> Hard to no race condition

*Race condition*: fast enough to get through code (insertion and increment)

Why did this occur?
- Shared data <-- Global state (e.g., tail)
	- `mmap()`
- Multiple (2 or more) processes and threads using that shared state
Work has:
	- reads the state, acts upon it - action: update, decision (does not require an actual change of state)
		- `if (tail > A.length()) throw exception`
	- Non-atomic : some block of code could be partially done
		- multiple machine instructions (`A[tail++]=20` does not solve it)
		- *Assumption*: a single machine instruction is atomic! (not truly true, but good enough!)
- between reading & acting, we were interrupted
- switch to something using the same state ==> changes to state, such that original decision/action is incorrect

solution ==> Block of code with potential race condition = Critical region/section
--> `enter_crit_region()` `<CRITICAL REGION>` `leave_crit_region()`

What is the implementation of enter and leaves (mutex or semaphore)?

Problem might be hard to see → if we are not preempted within the crit_region ;  if we don't schedule with another process using the same resource.

But it might suddenly manifest (new hardware, new OS) <-- even without the change in code.


```
enter_critical_region()
{
	disable_interrrupts
}
leave_critical_region()
{
	re-enable_interrupts
}
```
<-- this makes preemption opt-out enforcement != enforcement
- way too much power to applications (they will abuse)
<-- Also,..., we cannot run any other process no matter if they access the shared data (hurt pseudo-parallelism and interweaving)
	- As long as the processes don't access the same crit reg, it's fine
Hence, we don't prevent interrupt

Mutually exclusive access to critical region

Code differs from archi to archi
M
IN MIPS tail++
`lw $to ()`
`addi $to` <-- if we are interrupted here another race condition (note each thread has their own state)
`sw $t0, ()`

in x86 one instruction


---

We want to say conflicting interweaving of critical region is bad.
Analogy to a lock....locking of mutex

Locking a locked lock, we lock.

We've seen this ...in 449....pthread_mutex



Goals <-- whatever implementation of leave/exit critical region ,we choose must satisfy these goals////
No two processes can be inside their critical regions at the same time
No assumptions about CPU speed or number of CPUs
No process outside its critical region may block another process
No process should have to wait forever to enter its critical region
 - Not because the critical work is infinite, but due to the implementation

Strict Alternation
- Implementations can differ!

While process B is waiting, NOOP!
We are preempted when we use the entire quantum
- BUt during that time, we do useless!

Infinite loop....with no updates/...but not a bug

^ This works....for goal (1)

- But look at goal (3)
	- After executing the NC (short), can process B enter the critical region of process B of the iteration?
		- Process A is not in the critical region
		- Goal 3 fails
Strict altenation means that we need to do ABABABABA...
Why? because we explicitly say whose turn it is. (the **turn** variable)


what if instead....
``` c
int locked = 0;
enter_cr(){
	while (locked)
		; // <--- What if we are preempted here (1)
	locked = 1;
}

leave_cr(){
	locked =0;
}
```
....Now we don't explicty identify whose turn it is....just that the resource is in use.


***BUT>>>>*** what if we are preemped at (1)....and another process runs the same region
Process B runs CR....We come back and run CR in A---> now broken
/// Non atomic....shared global state <-- we can be interrupted between reading and action.... **race condition!** (inside the code handling the race codntion)

ADDING MORE CODE DOOESN'T HELP!


How do we fix this? (Read and set in one step...)

---> We need hardware support!

Why would a processor have this? To support locks! (at least in some architectures) `TSL`- test and a set lock

Hardware guarentees atomicity!

A lot of archi profvdies an atomic exchange instruction!

Busy waiting....Do we really need hardware support?
---> can we do with only software w/o race condition....(no soln for 30 years)---> Peterson's soln


