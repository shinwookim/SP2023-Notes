# What is an Operating System?

> At its basics, an operating system manages resources and abstracts details. 

This definition alone does not reveal much about what an OS is. Throughout this term, we will look at more definitions to supplement it and get a better understnading of what an OS is.

## What is a resource?
Recourses are anything that we have a finite amount of. More specifically, they must be *reasonably constrained* since everything in the real world is technicaly finite.

### CPU Time
For instance, **CPU Time** is a common resource in a computer. Even the fastest processors can only do so much work per unit time. We cannot run any more processes without adding more of it (e.g., more cores).

### Memory
Another common resouce in a computer is **Memory**. Recall the **Von Neumann architecture** in which both processes (code) and data lives in memory. In this architecture, the processor must fetch programs from memory and work on data that lives in memory/registers; the processor cannot execute code or modify data directly from the disk. Here, everything the processes does is an instruction drawn from memory/register.

Note that in the Von Neumann architecture, the code can also be considered to be data (as both lives on the RAM). As a consequence, it is *possible* to run a program which modifies itself (although it is highly discouraged). In comparision, the Harvard architecture has two memories (one for data and one for code).

*Technically* with modern systems, we can argue that they can change between the architectures depending on what the focus is (as generally modern systems have a split L1 cache)/

### I/O Devices
**Input/Output devices** are another type of a resource. All the data that is not stored in memory is stored in I/O devices (disks, file systems). I/O devices generally have information we want to consume, or receieve information we produce.

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
Although we will primarily focus on Operating Systems with many resources (mostly Desktop OS), it is important to recognize that different systems have different needs which means they need different operating systems.

In Operating Systems with very small resources, they do not have a lot of functionality and are built to do the bare minimum.

For instance, **embedded systems** are built as small and cheap as possible. They maximize the resources that they have, and only have the resources that are absolutely necessary. Often in these systems, we know the workload in advance and do not need to worry about users. It's also arguable that these systems do not have an OS, if the code runs on 'bare metal'. However, if an OS exists, it might simply be a library that glues and links programs together. 

A **real-time system**, is a system which has a set workload, and a deadline to complete them. Often, there are major consequences on how we manage the resources such as CPU time. A major consideration in real-time systems is what will happen if a deadline is missed.

If missing the deadline is bad, we call it a **soft real-time system**. For instance, a video codec needs to decode video and figure out to display within a set time (e.g., 30ms). If the codec misses the deadline, the video gets laggy.

If missing the deadline is catastrophic, we call it a **hard real-time system**. Examples of hard real-time systems include: Autopilot, Nuclear Power plant control, Health care devices. 

---
# History of Operating Systems

For sake of simplicity, we begin our discussion with computers with limited resources...the computers of the past. 
1. At first, calculation was done by human calculators, often female manual labor workers.
2. During World War II, mechanical computers were invented to break code (encryption)
3. Towards the end of the war, electronic computers were developed. These early, primitive computers were *single purpose* (computation) computers. They could do one thing, and were not 'programmable'.
4. After the war, the military took charge in developing electronic computers in national research labs. There, many academics saw the potential of computers in research, and built computers which could be programmed (by manipulating physical wires).
5. In the 40/50s, the **Von Neumann architecture** is proposed, and we are introduced to computers which can execute code and manipulate data. These computers were programmable in machine language.
6. As hardware became faster, programs were developed to encode a programming language into machine code. Later, **high-level programming languages** were developed.
1960s:
In the early days, computers were large and extremely expensive (more than the salaries of its users). Thus, the computer's time was more important than the user's time. Therefore, many large institution gave *time* to researchers and billed them for the time used (time-sharing). It was crucial to make sure that the computer was running constantly (to keep costs down). Thus, it was crucial to make sure our programs ran as fast as possible. Input/Output was unheard of; users would program/debug on paper (as it was extremely expensive to spend time programming directly on a computer) using FORTRAN, COBOL, etc.. Programs would then be transferred on a punchcards (each holding 40B).!
![](OS_History.png)
We then use cheaper computers to digitize our punch card into magnetic tape. Then we use the tape on our system, which produces an output tape. Our output tape is then brought to another computer and printed (physically) on paper.

Extreme specialization of computers. Big Idea: our time is less important than the computer. We need to maximize it running time.

System tape contains compiler and some rudimentary billing software. This is our first notion of an operating system.



During this time, architecture and instruction sets are constantly changing as researchers are figuring out to making computer faster. Developers are programming in assembly language (as compilers waste time). Thus, they were not compatible with each other.

The presence of the main and auxiliary computers (I/O) fills up space. IBM realizes that this is a barrier for small institution. They create a set of 3 computers that is fundamentally the same. The 3 machine have the same architecture (but with difference performace). Now, we have compatibility. This drives adoption of computers and standardization.


Later, we will see the minituraization of computers with the invention of transistors.
70s:
Eventually, computers become fast enough that one person running one program does not maximize the resource use.

MIT produces an opearting system which allows many users to submit their programs (using terminals/IO) --> Bell Labs. (Multics)

Development runs poorly --> Bell Labs quits.

A few programmers at the lab developed a smaller version and called it **Unix**. It's written in **C**, also developed at Bell Labs.

Late  70s: we have personal computers. we can buy the parts and build one at home. Computers are affordable (but still expensive).

Proposal: Cities and towns purchase computers, and users could plug in a terminal at home (much like a utility). Internet? --> this is why we study history (what's old is new again)
> Ontogeny Recapitulates Phylogeny

This proposal is thrown out with development of personal computers (paradigm shift): miniturization and mass production of computers.

---

# Multiprogramming

Eventually, the time of computer becomes cheaper (lower than that of users). However, we need to think that our OS will aid in the maximization of resources. If we have ram, we should use it. If we have, CPU time, we should use it. --> Running multiple programs. Von Neumann --> Sharing of CPU implies sharing of Memory.

We can partition memory and give it to each job/process.

One problem with his approach (**protection problem**) is that we need to ensure that programs that does not modify code/data of another program (whether intentionall or not). Now, we need to do extra management to ensure that programs are well behaved.
![](Memory%20Partition.png)
One solution: remember bounds and ensure new memory addresses are within bounds
![](Image%20Protection.png)
Perfectly valid approach, but not what we do. We use virtualization. 

Virtualization simulates exclusive access.

***Space***: Technical term. Mathematical notion of enumeration of all possible.
e.g., what is the space of all possible 4 character ascii strings? --> 7 bit --> $128^4$
![](Address%20Space.png)
ADDRRESS is just a number. And since all modern computers store numbers in binary,
native word size (e.g., Nintendo was an 8-bit system, 64 bit CPU)- most data that the computers can work with in one system/size of registers. native word size is also our address size.

Assuming 32 bit CPU: we have a 32 bit address.
Space: Enumeration of all possible address? `0000000....0000`, `000000...000001`,...,`1111...111111`. approx 4.2 billion addresses.

Most systems are byte-addressable. (We can only talk about addresses in terms of multiple of a byte). --> 4.2 Billin bytes of addressable memory --> 4 GB of RAM at max.
