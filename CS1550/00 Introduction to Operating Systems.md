# What is an Operating System?

> At its basics, an operating system manages resources and abstracts details. 

This definition alone does not reveal much about what an OS is. Throughout this term, we will look at more definitions to supplement it and get a better understanding of what an OS is.

## What is a resource?
Resources are anything that we have a finite amount of. More specifically, they must be *reasonably constrained* since everything in the real world is technically finite.

### CPU Time
For instance, **CPU Time** is a common resource in a computer. Even the fastest processors can only do so much work per unit time. We cannot run any more processes without adding more of it (e.g., more cores).

### Memory
Another common resource in a computer is **Memory**. Recall the **Von Neumann architecture** in which both processes (code) and data lives in memory. In this architecture, the processor must fetch programs from memory and work on data that lives in memory/registers; the processor cannot execute code or modify data directly from the disk. Here, everything the processes does is an instruction drawn from memory/register.

Note that in the Von Neumann architecture, the code can also be considered to be data (as both lives on the RAM). As a consequence, it is *possible* to run a program which modifies itself (although it is highly discouraged). In comparison, the Harvard architecture has two memories (one for data and one for code).

*Technically* with modern systems, we can argue that they can change between the architectures depending on what the focus is (as generally modern systems have a split L1 cache)/

### I/O Devices
**Input/Output devices** are another type of a resource. All the data that is not stored in memory is stored in I/O devices (disks, file systems). I/O devices generally have information we want to consume, or receive information we produce.

### Security
Although not a typical resource, **security** focuses on the **management** of the other resources. Since the operating system manages the resources, it is a great place to implement security (E.g., access control). Often, this raises the question of *could you do X?* and *should you do X?*. and *security* vs *availability*.

## Abstracting details
In early systems, resources (e.g., RAM, CPU speed) were so limited that we gave **exclusive access** to programs. However, as resources grew to be more abundant, we wanted to be able to share our resources among various processes.

Our Problem: We have enough resources that sharing is possible, however not enough to make their management unbounded, or trivial. Thus, we need a way of managing the resources between different processes.

Sharing is hard because each process is greedy. The operating system is like the only 'adult' in a room full of children (processes).

Furthermore, we don't want programs worrying about other programs taking over resources. A developer designing a program should not worry about other programs running concurrently. To make the programmer's job easier, we can abstract a notion of exclusive access to the resources. Now, the program only needs to worry about the abundance of resources (whether we have it or not) instead of worrying about whether another process is using it. 

The work of sharing the resources is hidden from the program as a form of **abstraction.** This provides an interface that makes it seem as the program has exclusive access to the resource, for the sake of simplicity. As an added benefit, we don't need to worry about security as long as we don't do something catastrophic (E.g., another program cannot modify my data). Typically, these abstractions are done through virtualization of the resources.

An operating system exists because of the reasons we have described above. But, technically, it does not need to exist, if all our programs work in harmony. But, it's really easy to break the system (for everyone).

# Varieties of OS
All operating systems are fundamentally the same (as we have just described), but they are also different depending on the amount and type of resources they have avaiable.

| Lots of Resources | Limited Resources/Smaller Systems|
|-------------------|----------------------------------|
|Mainframe OS|Real-Time OS|
|Server OS (including Supercomputers which have a lot of resources| Embedded OS|
|Parallel Computer OS|Smart Card OS|
|Personal Computer OS| |

## Systems with lots of resources
When we think computers, we often think about systems with lots of resources (such as a Desktop PC, smartphone, etc.)

Mainframes are types of systems that have enough resources can virtualize the entire computer (E.g., VM - one of the oldest operating systems). With their amount of resources, sharing is almost trivial.

Servers and PCs on the other hand, have enough resources that sharing makes sense, but not enough that management is trivial. Our focus will be on these systems, and we will begin by looking at OSes with constraints such as a single core CPU.

## Limited Resources/Smaller Systems
Although we will primarily focus on Operating Systems with many resources (mostly Desktop OS), it is important to recognize that different systems have different needs, which means they need different operating systems.

In Operating Systems with very small resources, they do not have a lot of functionality and are built to do the bare minimum.

For instance, **embedded systems** are built as small and cheap as possible. They maximize the resources that they have, and only have the resources that are absolutely necessary. Often in these systems, we know the workload in advance and do not need to worry about users. It's also arguable that these systems do not have an OS, if the code runs on 'bare metal'. However, if an OS exists, it might simply be a library that glues and links programs together. 

A **real-time system**, is a system which has a set workload, and a deadline to complete them. Often, there are major consequences on how we manage the resources, such as CPU time. A major consideration in real-time systems is what will happen if a deadline is missed.

If missing the deadline is bad, we call it a **soft real-time system**. For instance, a video codec needs to decode video and figure out to display within a set time (e.g., 30ms). If the codec misses the deadline, the video gets laggy.

If missing the deadline is catastrophic, we call it a **hard real-time system**. Examples of hard real-time systems include: Autopilot, Nuclear Power plant control, Health care devices. 

---
# History of Operating Systems
For the sake of simplicity, we begin our discussion with computers with limited resources...the computers of the past. 
1. At first, calculation was done by human calculators, often female manual labor workers.
2. During World War II, mechanical computers were invented to break code (encryption)
3. Towards the end of the war, electronic computers were developed. These early, primitive computers were *single purpose* (computation) computers. They could do one thing, and were not 'programmable'.
4. After the war, the military took charge in developing electronic computers in national research labs. There, many academics saw the potential of computers in research, and built computers which could be programmed (by manipulating physical wires).
5. In the 40/50s, the **Von Neumann architecture** is proposed, and we are introduced to computers which can execute code and manipulate data. These computers were programmable in machine language.
6. As hardware became faster, programs were developed to encode a programming language into machine code. Later, **high-level programming languages** were developed.

## Primitive Operating Systems
In the early days (1960s), computers were extremely large and expensive; they costed more than the average salary of its users. Therefore, the computer's time was more important than the user's time. To minimize the costs, large institutions who owned these computers gave *usage time* to researchers (and users) and billed them for the time used (*time-sharing*). To maximize the profit, it was crucial for them to ensure that the computer was running as much as possible (any time the computer was not running, it was missing out on profit). This meant that users had to design their programs to run as efficiently as possible (to lower the bill) and they had to program on paper (since time spending doing input/output was time that was not spend computing/billable). Often, early programmers would write their programs on paper (using FORTRAN, COBOL, etc.), debug, and then transfer them to a stack of punch cards (each holding only 40 Bytes).
![](OS_History.png)
The programmers, once it was their turn, would use cheaper computers (a-b) to digitize the punch cards into magnetic tape; again, since the costs of using a computer was so expensive, everything besides the actual computation was not done on the main system. They would, then, feed the tape into the main system (c-d) which would run the desired program (along with accounting software) and print the output onto another tape. Finally, this output tape was fed into yet another cheaper machines (e-f) which would print the output (physically) onto paper.

In these early computers, *the user's time was less important than that of the computer*. To maximize the running time of the main system, the computers became extremely specialized while any auxiliary functions (such as I/O) were handled elsewhere.

Notice that our program code was converted and digitized onto tape, along with some rudimentary billing software (b). This is our first notion of a primitive operating system.

## Rise of Assembly Language(s)
During this period, many architectures and instruction sets were developed (and constantly changed) as researchers experimented to make computers even faster. Once technology progressed, developers began programming in assembly language (compilation for higher level languages wasted the time of the computer). However, the constantly changing instruction sets meant that the computers of these times were not compatible with each other.

## IBM standardizes the ISA
With main and auxiliary (I/O) computers taking up so much space and money, IBM realized that this created barriers for smaller institution from adopting computers (and IBM missing out on potential customers). In response, IBM created a set of 3 computers who shared the same fundamental architecture (but with varying performances). Now, smaller institutions could afford computers and scale up as they needed to. Furthermore, these new computers drove efforts at standardization/compatibility, and the adoption of computers. 

## Need for Multiprogramming and UNIX
With the invention of the transistors, there was the miniaturization, and performance boost computers. In the 1970s, computers eventually become fast enough that a single person running a single process at a time did not maximize the use of available resources (meaning that computers were not maximizing their profit). Thus, it became necessary to be able to run multiple processes at once.

Efforts to design a system to manage resources among multiple programs (an operating system) were first developed at Bell Labs by MIT researchers. However, this project (Multics) ultimately failed and was cancelled. Yet, few programmers at Bell Labs were persistent about this goal (a multiprogramming system) and create a smaller version of Multics called **Unix** (developed in **C** which was also created at Bell Labs).

### Widespread adoption of computers & a proto-internet proposed
In the late 70s, costs came down enough to allow for personal computers. Consumers could now buy parts and build a computer at home at a relatively affordable price.

However, these personal computers often lacked in performance, and high performance systems will still extremely expensive. Thus, some people proposed that institutions/government (with large enough funding) such as cities and towns should purchase a high performance machine and allow consumers to plug in a terminal at home for a fee (much like a utility).

This proposal however is largely not adopted as hardware for personal computers became faster, and a paradigm shift occurred leading to the miniaturization and mass production of computers.

## Why we study history
> Ontogeny Recapitulates Phylogeny. What's old is new again!

However, this reveals why we study history. The proposal for a network of shared computers and accessing them for a fee very much resembles Internet (which would be developed in the 90s). Studying history allows us to discover unrealized ideas that may be realized in the future. Often computers follow a similar cycle, (as was with original single-processing smartphones), and studying it allows us a better understanding of how they work.

---
# Multiprogramming

Eventually, computers become cheap enough that the user's time is worth more than that of the computers'. That means we want to implement multiprogramming as users wish to do many things at once.

As an aside, people often (mistakenly) want their resource usage to be lower (such as when we open the task manager). However, for this course, maximizing the usage of resources (CPU Time, Memory) is a good thing. It is what allows for multiprogramming (as long as the OS is efficient in management of the resources).

Also, running multiple programs implies the sharing of memory. Since all programs need to be read from memory (in a Von Neumann Architecture), sharing of CPU's time automatically implies the sharing of memory.

## Memory Management: Partitioning
However, how we share memory is a major concern for the OS. A primitive approach may be to partition memory and give it to each job/process.
![](Memory%20Partition.png)
However, one problem with this approach is that we need to ensure that programs cannot modify/read the code and data of other programs (**protection problem**), which means that we must do extra tasks for management to ensure that all programs are well-behaved.

### Setting Memory Bounds
To solve the protection problem, we may decide to remember the bounds of each process and ensure that new memory addresses are within the assigned bounds.
![](Image%20Protection.png)
This is a perfectly valid approach, but it is not what we do.

## Memory Management: Virtualization
Most modern operating systems take another approach: **virtualization**. Virtualization is extremely useful because it simulates exclusive access for each program (using **address spaces**). That is, the operating system makes it seem as if all processes have their own set of memory.
![Address Space](Address%20Space.png)
Note that the term **space** in address space is a technical term; It refers to the mathematical notion of an enumeration of everything that is possible. For instance, the space of all possible 4 character ASCII strings is $128^4$ (assuming each ASCII character is 7 bits).

For our case, an address is simply a number (which is stored as binary in modern computers). We will consider a CPU with a **native word size** of 32 bits. (The native word size refers to the most data that the computers can work with in one instruction; often, the native word size is also the address size). The address space of a 32-bit CPU is the enumeration of all possible addresses (`0000....000`, `0000....001`,`0000....011`,`1111....111`) which is approximately 4.2 billion addresses.

Most systems are byte-addressable (meaning we can only talk about addresses in terms of multiple of a byte). Thus, a 32-bit CPU has at most 4.2 billion bytes of addressable memory, or 4??GB of possible RAM. 

There is a caveat to address spaces, however. In a 32-bit CPU, we promise all processes each that the 4??GB of RAM is either the process's or not yet that process's. But, we know that this promise cannot be false. Discounting the amount taken up by the OS, a modern computer runs hundreds of processes in the background; and it's physically impossible to give 4??GB to all the processes.

Yet, since we promise each process the enumeration of the entire 4GBs, no process is able to generate an address that is the code/data of another program. This is what gives the illusion of exclusive access. In this scheme, the same (virtual) address can hold different values (based on which program requests it). The address provided to the program is not the actual address, but an abstraction provided by the OS. Thus, we have solved the protection problem.

As an aside, why does the stack grow down, and heap up? Because all the other ones create issues. If both grow in the same direction, we will need to shift data as they grow. If we split the memory into halves, our programs may terminate when they run out of heap/stack space when there is still space left in memory. By having them grow in the opposite directions, n space is wasted even when they collide.

