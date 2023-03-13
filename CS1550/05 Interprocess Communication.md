# Inter-process Communication
We have seen so far how we can achieve parallelism by using threads and process. In our discussion of threads, we mentioned that multiple threads can communicate easily with one another through a shared address space, which allows all threads to read and write to the same variables.

However, processes may also need to communicate with one another. In fact, this is a rather common desire for processes. For instance, in a shell pipeline, the output of the first process may need to be passed to the second process, and so on. Here, we will see how processes can communicate with one another.

A few key questions arises as we begin the study of inter-process communication.
1. How exactly can we pass information between multiple processes?
2. How do we make sure that a process does not get in the way of another?
3. How can we properly sequence processes (in a specified order)?
Notice that the first question applies only to processes and has a rather simple solution. Just like threads, we can share data among multiple process via `mmap()`. However, the last two questions, which applies to both processes and threads, require a much more elegant solution.

## Race condition
Suppose we have a rudimentary queue which is backed by an array. To use parallelism, this queue will be *shared* among two processes *A* and *B*.
``` C
/* Shared Data*/
int tail = 4;
A[] = {1, 8, 5 ,6, ' ', ' ', ...};

/* Process A*/
A[tail] = 20;
// What if we process switch here?
tail++;

/* Process B*/
A[tail] = 9;
// What if we process switch here?
tail++;
```
Notice that process A enqueues 20 and increments the `tail` counter, while process B enqueues 9 and increments the `tail` counter.

Hence, we would expect that after the termination of both processes, our queue would be `{1, 8, 5, 6, 20, 9}` or `{1, 8, 5, 6, 9, 20}`. Note that we do not care about the order of 20 and 9 since if we wished to specify an order, we would not be using parallelism anyway.

Indeed, in many runs, the output does match our expectations. However, since scheduling is *chaotic*, we may find ourselves in the situation where we *process switch* between when we add to the array and increment the tail. Assuming process A runs first, we would add 20 to the queue to get `{1, 8, 5, 6, 20,' ', ...}`.  Then, as mentioned, the process is interrupted and switched to process B. Since the `tail` counter has not been updated yet, `tail == 4` and we overwrite the 20 that is already in the queue to get `{1, 8, 5, 6, 9,' ', ...}`. We then increment the `tail` and process B terminates. Lastly, the scheduler chooses process A, and we continue from where we were interrupted by incrementing the `tail` counter. Thus, we are left with a queue `{1, 8, 5, 6, 9,' ', ...}` which has an unmatching tail counter `tail == 6`.

This is clearly an undesired behavior; not only is the data in the queue wrong, subsequent enqueues will leave an empty space in the underlying array (which may cause issues when we dequeue). So why did this happen?

Simply, our processor was not fast enough to get through the entire code (insertion and increment) before the process was preempted. Situations like these in which multiple process reads and writes to some shared data and the final result depends on who runs precisely when is called a **race condition**.

But more precisely, the problem had to do with the fact that the work had multiple stages. That is, we first needed to read the shared state and then act upon it. Note that the *action* may or may not involve a change of state. For instance, we may *update*  a shared variable, or we may make a *decision* from the shared state and execution unrelated code. E.g., `if (tail > A.length()) throw exception`. However, since the work is done in multiple steps, we might be interrupted (preempted) between when we *read* and when we *act*.

Code such as this in which some blocks of code could be partially done before we are preempted is called **non-atomic**. Note that just because a code is a single-line, it does not mean that the code is **atomic**. For instance, `A[tail++]=20` will not solve the issue above, as this line of code is run as multiple machine instructions (which we can be preempted in). Typically, we will assume that a single machine instruction to be atomic (although this may not necessarily be true depending on architectures).

Problems from race conditions are hard to spot, as it occurs only when the process is preempted between reading and acting by a process that uses the same shared state. In fact, if the scheduler does not ever choose another process that uses the same shared state, the issue may lie dormant forever. Yet, it is important to identify this issue, as it may suddenly manifest with new hardware, new operating system, or by random chance even without any change in the code.

## Critical region
We call the region of code that accesses shared memory (code that contains a potential race condition), the **critical region**. If we can ensure that no two process are simultaneously in the critical region, we can solve the problem of race condition.

Hence, we will create handlers `enter_critical_region()` and `leave_critical_region()` to be called when we enter and exit the critical region. The goal of these handlers will be to ensure that a process cannot enter the critical region if one is in the region already.
```
enter_critical_region()
<Critical region code>
leave_critical_region()
```

### Disabling interrupts in the critical region
Notice that the race condition manifests only when we are interrupted inside the critical region. Thus, a naïve implementation might be to enable and disable interrupts inside the critical region:
``` c
enter_critical_region()
{
	disable_interrrupts();
}
leave_critical_region()
{
	enable_interrupts();
}
```
However, there is a big problem with this approach. First, this makes preemption opt-out. That is, the operating system cannot enforce policies and manage resources (as the processor cannot switch to the OS code without an interrupt). Hence, we are giving too much power to applications (which they will abuse). For instance, a malicious application might call `enter_critical_region` but never leave, causing no process other than itself to run forever.

Furthermore, this breaks the notion of pseudo-parallelism and interweaving processes since we will not be able to run any other processes while a single process is in the critical region regardless of whether the other processes access the shared data or not. As long as the other process isn't trying to access the same critical region, it should be allowed to run.

### Mutual exclusion using lock variables
Since we can't prevent interrupts (nor do we want to), what if instead we use a *lock* (often called a **mutex** for mutual exclusion) to lock entry into the critical region?
``` c
/*Process A*/
lock(&mutex); // Lock the mutex before entering the CR
A[tail] = 20;
tail++;
unlock(&mutex); // Unlock the mutex when we leave the CR

/*Process B*/
lock(&mutex); // If the mutex is already locked, we block
A[tail] = 9;
tail++;
unlock(&mutex);
```
Now, just before we enter the critical region, we lock the mutex. Then, if we are preempted inside a critical region and another process tries to access it, they will attempt to lock the mutex for themselves. However, upon discovering that the mutex is already locked, they will block.

As an implementation, we often use the `p_thread_mutex_t` provided by the pthread library:
```c
#include <stdio.h>
#include <pthread.h>

int tail = 0;
int A[20];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void enqueue(int value)
{
	pthread_mutex_lock(&mutex);
	A[tail] = value;	
	tail++;
	pthread_mutex_unlock(&mutex);
}
```
Yet, this approach still contains the same flaw from the initial example. Inside the `pthread_mutex_lock()` function call, if we are interrupted before we are able to update the mutex, we may still end up with multiple processes inside the critical region simultaneously. The race condition was simply transported from the `enqueue()` function to inside the `lock()` function.

## Finding a solution to critical regions
So if these approaches do not work, what does? To find a solution to the problem of race condition, we should first establish the goals of our solution. Whatever our solution (or implementation) may be, it must satisfy these goals:
1. No two process can be inside their critical regions at the same time
2. Solution should not depend on assumptions about CPU speed or number of CPUs
3. No process outside the critical region may block another process
4. No process should have to wait forever to enter its critical region
	- That is, no process should have to wait indefinitely due to the implementation of our solution (not because the work inside the critical region is infinite).

### Strict alternation
``` c
/* Process A*/
while (TRUE) {
	while (turn != 0)
		; /* loop */
	critical_region ();
	turn = 1;
	noncritical_region ();
}
/* Process B*/
while (TRUE) {
	while (turn != 1)
		; /* loop */
	critical_region ();
	turn = 0;
	noncritical_region ();
}
```
Although there are various implementations, this is the basic idea of strict alternation. The integer variable `turn` keeps track of whose turn it is to enter the critical region and examine or update the shared memory.

Notice that while it is not each process's turn, they simply wait idle (no-op). That means that if process A is preempted inside the critical region, and process B is scheduled, it will wait idle for the entire quantum. This type of waiting in which we continuously test a variable is called **busy waiting**. This should generally be avoided since it only wastes the CPU time.

Notice that strict alternation accomplishes our first goal. No two process can be inside the critical region simultaneously.

However, strict alternation explicitly declares whose turn it is. That means, process A enters the critical region, then process B, then process A, and so on. But what if process A runs through one iteration (completes critical region work and non-critical work)? Now, process A tries for the next iteration and attempts to enter the critical region, but is blocked even though there are no processes inside the critical region. Hence, our third goal (no process outside the critical region can block another) fails.

Then, what if instead of explicitly declaring turns, we implicitly identify which resource is in use?
``` c
int locked = 0;
enter_critical_region(){
	while (locked)
		; // <--- What if we are preempted here
	locked = 1;
}

leave_critical_region(){
	locked =0;
}
```
We can still be preempted just before we update the lock with another process running the same code. Thus, the second process will run the critical code, be preempted, and when we return, the initial process will still try to run the critical code, breaking our mutual exclusion lock.

Again, the code that reads (the lock) and acts on it is *non-atomic*. Hence, the code that   is supposed to handle the race condition, now has a race condition.

### Hardware-supported 
It appears the adding more code doesn't really help. So what if, instead, we try adding hardware support? We have said that the problem comes from the fact that reading and acting is a non-atomic instruction. So, what if we make it atomic?

Some architectures support just this in an instruction called **Test and Set Lock** (`tsl`). In this case, the hardware guarantees atomicity and the issue is solved. Often, most modern architecture has a more general *atomic exchange* instruction which reads and writes values in a single instruction.

### Peterson's solution
However, do we really need to incorporate hardware support for busy waiting? After all, that violates the second goal (no assumption about hardware).

In fact, there was no software solution to this problem for 30 years. But, in 1981 Gary L. Peterson found an algorithm which achieves busy waiting called **Peterson's algorithm**.
``` C
#define FALSE 0
#define TRUE1
#define N 2// # of processes
int interested[N]; // Set to 1 if process j is interested
int last_request; // Who requested entry last?

void enter_region(int process){
	intother = 1-process; // # of the other process
	interested[process] = TRUE; // show interest
	last_request = process; // Set it to my turn
	while(interested[other]==TRUE && last_request == process)
		; // Wait while the other process runs
}
void leave_region (intprocess) {
	interested[process] = FALSE; // I’m no longer interested
} 
```

## Producer-Consumer Problem
In 1965, Edsger W. Dijkstra described a classical computing problem which demonstrates concurrency, called the **producer-consumer problem** (also called **bounded buffer problem**).
``` c
/* Shared variables */
#define N 10;
int buffer[N];
int i = 0, out = 0, counter = 0;

/* Producer produces X*/
while (1) {
	if (counter == N) sleep();
	buffer[in] = ...; // Production of items into the buffer
	in = (in+1) % N;
	counter++;
	if (counter==1) wakeup(consumer);
}

/* Consumer consumes X*/
while(1) {
	if (counter == 0) sleep();
	... = buffer[out]; // Consumption of items from the buffer
	out = (out+1) % N;
	counter--;
	if (counter == N-1) wakeup(producer);
}
```
In this problem, there is a producer who *produces* something and a *consumer* who consumes it. Notice that the natural way for this problem would be to run the producer and consumer sequentially. In one iteration, a producer would produce one instance, and then a consumer would consume it, before re-running the iteration.

But what if there was a disparity in the production and consumption speeds? If production was much faster than consumption, we may want multiple instances of the consumers. Similarly, if consumption was much faster, we may want multiple instances of the producers. Hence, this problem would benefit from introducing parallelism.

Indeed, this is the whole point of existence for **caching**. In our simple demonstration, this shared cache is called a **buffer**. For simplicity, we will back this **buffer** with an array (recall, complex data structures like a queue had the risk of causing race conditions) which means our buffer has a fixed maximum size (hence the *bounded buffer*). Thus, it follows that we need to check to make sure neither the producer nor consumer over-run or under-run the buffer.

Hence, we provide if statements in the producer to guard over-runs, and in the consumer to guard under-runs. In either case, if the condition is true, we will invoke the `sleep()` system call, which puts the current process in the blocked state. This is in contrast to `yield()` which similarly context switched but kept the process in the ready state.

With this `sleep()` system call, the current process (either the producer or consumer) is now blocked at the scheduler level. But, notice that if we put ourselves to sleep, we cannot wake ourselves up. Before, when we put ourselves into the blocked state (often to wait for I/O), we informed the OS that we were waiting for some condition to be met (often an interrupt) to be woken up. Here, we call `sleep()` without any condition. Thus, we must rely on another process to wake us up. Hence, the opposite process must make the corresponding system call `wakeup(process)`.

### Race conditions
Again, there are multiple race conditions in our code. Similar to what we saw before, we share the `counter`, `in`, and `out` variables which can cause race conditions from preemption. 

But there is more. Suppose the scheduler chooses to run the consumer first. Since the buffer is empty, the consumer will attempt to call `sleep()`. Now, just before we sleep, suppose we are preempted, and the producer is chosen. The producer, since the buffer is not full, will produce and update the counter to 1. Hence, it will attempt to wake up the consumer (which is not asleep yet).

### Race condition handling with mutexes
So how do we take care of this?
One approach would be to do nothing (a no-op). But in this case, we might end up with the producer, asleep, waiting for the consumer and the consumer, asleep, waiting for the producer; a **deadlock**.

A deadlock occurs because of a race condition. Hence, we return to the problem of solving race conditions: how do we make condition check and sleep into an atomic unit?

Once again, we can try to add handlers when we enter and leave the critical region.
``` c
/* Consumer*/
enter_crit()
if (counter == 0) sleep();
exit_crit()

/* Producer */
enter_crit()
if (counter == N) sleep();
exit_crit()
```
However, as we have seen before, we will result in yet another deadlock (although this time, it is guaranteed). Suppose that the consumer runs first and enters the critical region before being preempted. The producer would then run, attempt to enter the critical region, fail and be blocked. We come back to the producer which invokes `sleep()` which blocks the consumer. Now, both processes are blocked, waiting for each other to wake them up. Hence, we are left with the same problem as before.

Notice that moving the `sleep()` outside the critical region is not viable, as that brings us back to the case before. And we can't sleep inside the critical region, since that creates another deadlock (and we will never exit the critical region).

Hence, we need our `sleep()` system call to: (1) sleep and (2) leave the critical region, but also (3) re-enter the critical region when we are awakened (to continue executing the `exit_crit()`).

This can be done through a **condition variable** which provides a linkage between process who are sleeping and process who are waking up. As an implementation, the pthread library provides us with `pthread_cond_wait(condition, mutex)`. Notice that this function is parameterized by the mutex (which locks the critical region) and the `condition` which provides the linkage (who we are waiting for).

Therefore, it appears that we can solve the producer-consumer problem (deadlock-free) as such:
``` C
#define N 10
int buffer[N];
int counter = 0, in = 0, out = 0, total = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t prod_cond = PTHREAD_COND_INITIALIZER;
pthread_cond_t cons_cond = PTHREAD_COND_INITIALIZER;


void *producer(void *junk) {
	while(1) {
		pthread_mutex_lock(&mutex);
		if( counter == N )
			pthread_cond_wait(&prod_cond, &mutex);
		buffer[in] = total++;
		printf("Produced: %d\n", buffer[in]);
		in = (in + 1) % N;
		counter++;
		if( counter == 1 )
			pthread_cond_signal(&cons_cond);
		pthread_mutex_unlock(&mutex);
	}
}

void *consumer(void *junk) {
while(1) {
	pthread_mutex_lock(&mutex);
	if( counter == 0 )
		pthread_cond_wait(&cons_cond, &mutex);
	printf("Consumed: %d\n",buffer[out]);
	out = (out + 1) % N;
	counter--;
	if( counter == (N-1) )
		pthread_cond_signal(&prod_cond);
	pthread_mutex_unlock(&mutex);
	}
}
```
Note that since it is generally bad practice to constantly lock and unlock (as they incur undesired performance costs), we've made the critical region one iteration of production/consumption.

### Semaphore-based solution
Previously, we did a no-op when a process tried to wake an already-awake process. What if instead, if we wake-up a non-sleeping process, we make sure that they do not sleep? That is, what if we counted the number of wake-ups and sleeps to determine whether a process should sleep or not?

We can do just this with the **semaphore**.
``` c++
class Semaphore {
	int value; // Accouting of sleeps and wakeups; positive more wakeups; negative means more sleeps; zero means balance
	processList p1;
	
	void down(){
		value -= 1;
		int (value < 0){ // value was 0 or fewer...we didn't miss the wakeup
			p1.enqueue(currentProcess); // to allow us to wake one of many sleeping processes
			sleep(); // Hence we sleep
		}
	}
	
	void up(){
		value += 1;
		if (value <= 0){ // value was negative before (note that this negative value is encoded in P.length)
			P = p1.dequeue() // which means someone was asleep
			wakeup(P); // we wakeit up
		}
	} 
}
```
Note that the nomenclature for `up()` and `down()` varies depending on the implementer. In his original paper discussing semaphore, Dijkstra used `P()` and `V()` (from Dutch) to indicate this; In Unix, the operations are called `wait()` and `post()`.

In this case, we also need a constructor which initializes the `value` to the starting number of wake-ups. Typically, we set this number to be zero or positive, as setting a negative value would indicate that there are already existing sleeping process (which cannot be true).

Let's take a look at who we can use these semaphores in the producer-consumer problem:
``` C
/* Shared Data*/
const int n;
Semaphore empty(n), full(0), mutex(1);
Item buffer[n];

/* Producer */
int in = 0;
Item pitem;
while (1) {
	// produce an item into pitem
	empty.down();
	mutex.down();
	buffer[in] = pitem;
	in = (in+1) % n;
	mutex.up();
	full.up()
}
/* Consumer */
int out = 0;
Item citem;
while (1) {
	full.down();
	mutex.down();
	citem = buffer[out];
	out = (out+1) % n;
	mutex.up();
	empty.up();
	// consume item from citem
}
```
Notice that we are using several semaphores to make clear the multiple race conditions present in the code. 

First, we need a semaphore (which we have called a mutex) for the purposes of mutual exclusion. When either the producer or consumer first executes, the mutex, which is initialized to be 1, is lowered, and we consume the *missed wake-up* and execute. Since we are starting with $1$ as our initial mutex value (we assume there is 1 missed wake-up at start), our `mutex.down()` essentially becomes `enter_critical_region()` and the `mutex.up()` becomes the `exit_critical_region()`. This, in essence, gives us our desired mutual exclusion.

Next, we need a semaphore to manage the shared resources. Consider a printer spool in a shared environment. If there are five printers but six print request, one request must wait. Similarly, if there are $n$ slots in the buffer, but more than $n$ producers, the additional producers must wait. Conversely, if there are more consumers than the products, the consumers must wait. In the case of our implementation above, this is achieved through the `empty(n)` and `full(0)` semaphores.

As an implementation, this would look something like:
``` C++
#include <semaphore.h>
#define N 10
int buffer[N];
int counter = 0, in = 0, out = 0, total = 0;
sem_t semmutex; // sem_init(&semmutex, 0, 1); in main()
sem_t semfull; // sem_init(&semfull, 0, 0); in main()
sem_t semempty; // sem_init(&semempty, 0, N); in main()

void *producer(void *junk) {
	while(1) {
		sem_wait(&semempty);
		sem_wait(&semmutex);
		buffer[in] = total++;
		printf("Produced: %d\n", buffer[in]);
		in = (in + 1) % N;
		counter++;
		sem_post(&semmutex);
		sem_post(&semfull);
	}
}
void *consumer(void *junk) {
	while(1) {
		sem_wait(&semfull);
		sem_wait(&semmutex);
		printf("Consumed: %d\n", buffer[out]);
		out = (out + 1) % N;
		counter--;
		sem_post(&semmutex);
		sem_post(&semempty);
	}
}
```
Here, we are using the wait and post syntax provided by the Unix semaphores. But, notice that we raise the empty or full semaphore before raising the mutex; similarly, we lower the mutex before lowering the full or empty semaphores.

If we were to do so the other way (raise mutex before full or empty), a preemption inside the critical region will cause a deadlock as the post to empty or full from the other process (which is inside the critical region) can never execute if we sleep inside the critical region.

However, notice that the empty and full semaphores as a matched pair between the producer and the consumer. Indeed, this is merely the same counter variable from before, bundled into an object called a semaphore. Hence, once again, we are met with a race condition (with the same reasoning as before). Whether it is the pthread library or semaphores, the instructions for checking the condition and acting upon it is still non-atomic.