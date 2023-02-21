# Types of Operating Systems
## No Optimal Solutions
In many other computer science courses, we often learned about algorithms and implementation that was the most optimal solution. However, we will discover that in this course, there are many situations where there does not exist a *best* or *optimal* way to complete a task.

Consider the problem of sorting an array. In data structures, we learned about the various sorting algorithms which outperformed one another depending on the starting state. Similarly, in operating systems, the methods we discuss will carry certain pros and cons which will be realized depending on context. Part of learning operating systems is learning that context by answering the following: What is needed for a certain implementation to outperform another and vice versa?

## Scheduling
Since our operating system is responsible for managing the resources of our system, it must build a schedule to determine what work is done when. To do so:
1. First, we need a data structure that can represent all the work that needs to be done. The schedule most commonly takes the form of a table or a linked list (on Linux) which holds the tasks. When a new job is created (for instance, when we launch a new program), we insert to our data structure; When the job is complete (for instance, when the program exits), we remove the task from the list
2. Next, we need to devise an algorithm which can walk over the schedule and select what task to run next based on some criteria. I.e., the scheduler must run some scheduling algorithm in which the input is the list of tasks, and the output is the decision.
3. Lastly, after we have made a decision, we need to actually run the program, which means we must switch out of the OS code into the user space. Thus, we need to conduct a context switch.
All of this work is called **scheduling**.

## Monolithic OS
> The one big rock approach

In a monolithic OS, this entire process of scheduling is part of the Kernel (OS). Everything from managing the task list (schedule), to determining what task to run next, to actually running the task is handled by the operating system.
![Monolithic OS](Monolithic%20OS.png)

## Microkernel (+ Exokernel)
But looking closely, we notice that a large part of scheduling does not require privileged instructions. Inserting and removing a node in a linked list (when a new process starts or exits) clearly does not require any privilege; and traversing a linked list and making a choice (scheduling) does not either. The only part of scheduling which requires privilege is when we need to restore context to allow user programs to run (after a choice has been made by the scheduler). In a Microkernel design, the goal is to make the kernel as small as possible. Thus, we push everything that does not need to be in the kernel (non-privileged code) out onto the server, which lives in the user space. Therefore, in a Microkernel, the management of the schedule, and scheduling is done in User mode, and only the context switch occurs in the kernel. 
![](Microkernel.png)

### Pros and cons of each design
So which design is better? That depends. Let us consider a few situations to see where a monolithic OS shines, and where the microkernel shines.

#### Considering performance
As mentioned before, we will be using the number of context switches required as a measure of performance. Consider scheduling. For a monolithic OS, scheduling requirest (at minimum) just 2 system calls. We need one to get into the kernel (from a user process), at which point all work related to scheduling is done inside kernel mode. Then, we need another to return to the user process. However, for a microkernel design, we need at least one context switch to move from the user process to the microkernel; another to move from the microkernel to the server handling the scheduler; one more to move back into the microkernel (once a decision has been made by the server); and lastly one final context switch to move from the microkernel to the new user process. As a consequence, microkernel designs are often slower than monolthic ones.

#### Modularity
However, a microkernel design shines when it comes to **modularity**. The microkernel's module based approach allows servers to be repaired, upgraded, or replaced rather easily. In fact, you'll find that monolithic OS often requires a reboot when upgrading a service (to restart the entire kernel) wheras this is not the case for microkernels. Furthermore. microkernel's modularity leads to smaller programs which means it is easier to debug and troubleshoot from a software engineering point of view. Smaller programs also means there are less chance of bugs or exploits in the code. 

But perhaps most importantly, the microkernel is more *secure* than a monolithic design. If an explot were to be injected into the OS, microkernel would most likely run that exploit in the user space whereas the monolithic OS would run the exploit code in kernel mode (this is assuming that the exploit doesn't lie on the microkernel itself). Furthermor, in considering reliability, we'll notice that if a server crashes, a monothlithic OS will crash the entire kernel (bringing down the entire system) whereas in a microkernel design, the server can simply be rebooted.

Indeed with both having their pros and cons, the two designs described here are still widely used in modern operating systems. Linux follows a monolithic design (although there was some controversy surrounding this choice of design — see Tanenbaum–Torvalds debate). On the other hand, Windows follows a hybrid microkernel design. Yet more interestingly, Windows follows a hybrid microkernel design in that it follows the general principles of a microkernel with some of the more commonly used servers (e.g., scheduling) pushed backed into the kernel for better performance.

# Virtual Machines
To study the various operating systems and their designs/implementations, this course will rely heavily on the use of virtual machines. Simply put, virtual machines are software that emulate actual hardware well enough that we could run an entire operating system on them. VMs vary from process level (e.g., QEmu/VMware), application level (e.g., JVM), to Hardware level (e.g., Virtual Machine Monitor, Hypervisor) which allows us to run multiple OS on a single machine.

Much like an operating system, a virtual machine must manage resources and abstract details. Hence, the study of the OS is essentially the study of VMs.

As an aside, the machine/OS that supports a VM is called the **host** (Linux in diagram) and the OS on the VM is called the **guest** (Windows NT). Furthermore, modern VMs are much faster than one might imagine due to the fact that many modern VMs allow fall through in which instructions in the same architecture (between host and guest) fall through, reducing time needed for emulation.

![](VM.png)


