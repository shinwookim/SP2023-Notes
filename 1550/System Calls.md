# System Calls
Note that an operating system is simply a piece of software that manages resources, and abstract details. For a mere software to do this, the OS requires some measurable amount of resources. That is, the OS must use up some resources, which decreases the amount of available resources for the processes. I.e., the OS is a (necessary) overhead that lies as an intermediary between the program (resource requestor) and the resource (request).
```mermaid
graph TD; 
  A[App] 
  B[Operating System]
  C[Hardware] 
  A--System Call-->B
  B-->C
  C-->B
  B-->A
```
**System calls** are how programs communicate with the operating system (which in turn communicates with the actual hardware). Previously, we used system calls in MIPS for input/output, management of the processes (such as terminate), and system-level randomness (for cryptography). In a more broad view, a system call instruction is how a program asks an OS to perform something on its behalf. In essence, it is a control transfer (much like `jal`).

Almost all programming languages (besides assembly language) has a standard library. Thus, if a program makes calls to a function in that library, it must be linked during compilation, and loaded into memory during run-time (as a consequence of the Von Neumann architecture). Consider a simple "*Hello World!* Program written in C":
```C
#include <stdio.h>
int main() {
   printf("Hello, World!"); // <-- call to standard library function printf()
   return 0;
}
```
Notice that we use the standard library function `printf()` to print our string to the standard output. Thus, when we compile our program, the call to `printf()` will be handled by a `jal` instruction to the `printf()` function. Thus, we are transferring control from our code into the `printf()` function.

Now if we examine the `printf()` function, we'll see that the majority of work done by the `printf()` is the stringification and interpolation of the argument string (using appropriate format specifiers). Towards the end of `printf()`, we are left with a string which is not yet displayed on the standard output.

The task of actually displaying the string is very complex. To show this string on screen, we need to manipulate the hardware which controls the exact pixels on the screen.
1. First, we need to read the font (fonts are small programs which describe how each character needs to be drawn) to determine how to draw the string (what pixels to manipulate)
	* We may also need additional information such as font-size
2. Next, we must determine where the terminal is, and what line and column to display at.
	* We may also need to consider line-wraps and work-breaking 
3. Finally, once we have all this information, can we use an instruction to turn on/off the necessary pixels.
Clearly this involves a lot of details about the hardware and is best left abstracted for functions like `printf()`. I.e., it is the job of the operating system.
![System Call](Assets/System%20Call.png)
Thus, just before `printf()` returns, it makes a system call (with yet another control transfer) with arguments to declare the location of output, `stdout`, and the string. This is verifiable by examining the assembly-level code. However, at a higher level, the syscall is simply telling the OS to do some task (print to stdout) and then returning once the task is complete. The details of how that task is done is abstracted.

## Why syscall?
From the details above, we can conclude that a syscall is simply a control transfer with a return. But if it is simply a control transfer, why wouldn't we just use `jal` (used for library functions) and not `syscall` (used for OS tasks)?

Recall that `jal` is a *J-type* instruction (6 bit opcodes, 26 bit immediate); and that at run-time the address of the control transfer is loaded from the 26 bit immediate. 

However, a syscall did not take any operands. We put an integer in register `v0`, but that integer was not an address. Instead, they were an enumerated ordinal which could be looked up on a table.

Even though they look similar, `jal` and syscalls work in different ways. The work of `jal` is based on functions which are addressed, while the work of syscalls are indexed.

As an aside, since the presence of the `func` field allows for more R-type instructions, to not waste the limited J or I type instructions, syscalls are technically R-type instructions.

At its core, when an app makes a system call, the operating system does some conditional work to determine if the app has access to the requested resource. If the condition is met, the OS will do the work requested, and if the condition is not met (due to security policies, resource usage, etc.), the OS will decline to do the work.$$\verb|Operating System = if (condition) {do OS_task;}|$$
### Preventing apps from bypassing the OS
If the OS is just software (built out of the same instruction set), how do we prevent someone from programming an app to skip the middle man (OS) and do the work itself? I.e., how can we grant authority to the OS to control the resources which it has domain over?

Recall that the instruction set defines the things that a processor can do. However, if we examine closely, the instruction set is partitioned into at least (and most often exactly) two sections for the sake of giving the OS authority.

## Dual Mode
1. There is the **User mode instructions**/**Protected mode instructions** which are the instructions that our user program runs.
2. And there is the **Kernel mode instructions**/**Privileged mode instructions** which the operating system runs.
Note that some architectures may have more than 2 partitions (e.g., x86 has 4 partitions called ring 0, ... ring 3). However, for our purposes, we will consider the instruction set to be partitioned into 2 parts.

Although most instructions (User/Protected mode) can be run by all programs, some instructions (Kernel/Privileged mode) can only be run in kernel mode. Up until now, all the instructions we've written were user/protected mode instructions.
* (Note that the Kernel, here, refers to the core space of operating system instructions, and will be used synonymously with the OS for now.)
When a kernel mode instruction is called, the processor distinguished which mode we are in to determine if it is allowed to run that instruction. A single bit flag inside the machine status register stores which mode we are in. If we are in kernel mode, the processor can run both user mode and kernel mode instructions. However, if we are in user mode, the processor can only run user mode instructions.

But what if we are in user mode, but call a kernel mode instructions? Well, that depends on what instruction is called. Historically and least desirably, x86 simply ignored these calls (which caused problems when trying to virtualize x86). However, in modern architectures (such as MIPS, and many x86 instructions), a call to an unauthorized instruction will raise an **exception** (e.g., Integer division by zero, Page fault). The exception tells the operating system, which usually sends a signal to the process (which by default crashes the process).

But if the OS needs to run kernel mode instructions, how does it flip the mode? It uses syscall (which changes the mode)! Hence, is the reason why syscalls and `jal` are different. Syscall tells the processor that the next instruction is OS code (and allows us to run private mode instructions). Once the OS code is run, and we return, the mode bit is flipped again (as we return from the kernel space to the user space).
+ Note that an exit syscall is special in that it never returns!
+ The OS is also event driven, it acts when it's called upon!
+ Syscalls are often very simple. Instead of multiple print syscalls, there is often a single output syscall which can be specified using flags!

---

## Interrupts
We've seen how, when a user program makes a system call, the operating system stops the hardware from running user code and switches to the OS code to complete the requested task. Thus, our code is *interrupted* while the OS does the requested task, and when it's done, the hardware is returned to our program. This is the idea of an **interrupt**.

When the OS gets an interrupt, it must stop the current program, handle the interrupt, then come back (or crash the program), much like a system call. In fact, a system call is a type of interrupt called **trap**; however, an interrupt is much larger than a system call. Grouping by the source, there are:
1. Software (originated) interrupt (I.e., Trap, Exceptions)
	- comes about because a piece of software requested it
1. Hardware Interrupt
	- comes from hardware (to pause the code until the cause of interrupt is handled)
	- goes back to code when the interrupt is handled.

To handle an interrupt, regardless of origin, the processor must set the program counter to the OS code which can handle the interrupt. The address that is loaded into the program counter comes from a structure on the chip called the **interrupt vector,** which is indexed by the particular types of interrupt. At each index is an address which points to the OS code which can handle the interrupt.

For system calls, they are all mapped to the same entry in the interrupt vector (usually index 0, sometimes index 128 in x86). That is, the instructions which handle *all* system calls are the same. To handle the exact system call, the operating system has a second table (Linux's syscall table) which is indexed by the ordinal stored in register `v0`.

Thus, it is a 2-step process of dispatching a particular syscall:
1. Interrupt vectors determines that the interrupt is a syscall
	- Escalates privilege and sets program counter
2. Syscall table in the OS handles the particular syscall 
	- Once the instructions handling the syscall completes, the privilege level is dropped and the syscall returns

## Performance of System Calls
This process seems very complex... but it is too expensive?

Recall that some instructions are slower than others. For instance, `jal` and `j` instructions are much faster than any load instructions. For system calls, the `syscall` instruction is only minimally more expensive than a division instruction. 

However, the system calls themselves are slow. This is because of the work that must occur after the initial `syscall`.

Recall with function calls, we had to store and restore registers (following the calling convention). We often had to push the register entries onto the stack and pop them before we returned. This was a lot of (computational work); however, it was necessary because only one set of registers needed to be shared among multiple functions. Thus, as function implementers, it was crucial to follow the established calling conventions.

In 447, however, we did not worry about the state of registers before and after a `syscall`. How? Since the majority of the operating system code is built from the same user mode instructions, it should be modifying and using the same registers. Therefore, like functions, a system call must preserve the state of registers (to preserve the notion of exclusive access). 

### Context Switch
> Switching from one running process to another

Unlike functions, however, the OS must preserve the state of *all* general purpose registers. This state is called **context**. For functions, we preserved a subset of the context (depending on the *save registers* defined in the calling conventions). Thus, sometimes we could optimize our code by *shifting around* the values of registers to avoid the long memory write/load times. However, operating systems must preserve the entire context, which means we cannot shift around registers. The data stored in all registers must be stored in memory; thus, a system call is more expensive and slow.

### Address Protection
The `syscall` (interrupt) instruction itself does not perform the context switch. Instead, after this instruction, the program counter is loaded with the address of the OS code which does. But how do we make sure that the Interrupt Vector is not compromised by user programs?

The Interrupt Vector is protected because accessing it is a privileged instruction (only the OS can do it). When our computer boots, the OS is run first (in privileged mode) to set the interrupt vector and install itself as the event handler.

In the 80s/90s, it was common for viruses to infect the master boot record, which would allow it to install itself on the interrupt table at boot. Thus, the viruses acted as the OS and would infect any floppy drives that were inserted (and spreading). In modern systems, it is very difficult to modify boot sequence/boot record Thus, this is no longer a modern concern.

### OS is Event Driven

As an aside, the OS is not a typical program. Since user programs cannot run while the hardware runs OS code, the operating system cannot wait idle (as it takes away resources from user programs). I.e., it is not watching any programs for any errors.
$$\text{System Call}\in\text{Interrupt}\in\text{Event}$$
Instead, the operating system is an **event-driven** program. It runs, only when it needs to (when other software generate events). Other examples of event-driven programs may be a graphical user interface program. A GUI does not wait for the user to interact, but when a user clicks a button (and generates an event), it handles it accordingly.

Conversely, an example of a non-event driven program may be a prompt and wait program. Consider the code `Scanner.nextInt()` in Java. This line waits for the user to input an integer, then follows its code procedurally.

The OS and user programs runs simultaneously, but sequentially. Interrupt vector is the link between the vent and the OS code.

## Memory Hierarchy
Back to context switch...
There are no safe registers...so we must save to memory. But which memory?

We have registers because they are fast/// Our instruction can only run as fast as our hardware. If we can add two numbers fast, we need to be able to read/write those numbers equally fast, for the instruction to be actually fast. (Getting the numbers is slower than doing the math!)

Since the speed of light is finite, electrical signals can only move so fast. --> Physical distance matters! Memory that is closer to CPU is faster!

Registers are fast because they are as close as possible to the CPU (they are actually inside it).

Static RAM vs Dynamic RAM
Registers are fast because it's made of flip-flops, etc. but is more expensive.
Dynamic ram is like a water bucket (threshold) --> We need to flush it periodically.

If we must write 32 registers to RAM for context switch, it's slooooow!

Examine a program with many print statements (and remove them!) Notice the difference
![](Pasted%20image%2020230123104046.png)


---





