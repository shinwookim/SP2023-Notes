# Monolithic OS
## Optimal Implementation?
In this course, there is often no 'best' or 'optimal' ways.

In other courses, there are the best way because we can mathematically prove it. (although sometimes it depends on context -- e.g., sorting)
WTF is Tim sort?
![](Pasted%20image%2020230125102721.png)
There are no definitive ways to do it. It depends... but on what?

"One Big Rock"
## SCHEDULING
OS is responsible for building a schedule for what work is done when.
1. We need a data structure that represents all the work that needs to be done
	1. Table or Linked List (Linux)
	2. When new jobs are created (double-click on icon), we insert
	3. When a program exits, we remove
2. We need a algorithm that walks over the list, and selects what task to run next (scheduler running a scheduling algorithm; input: jobs; output: choice)--BASED ON SOME CRITERIA
3. We run the job/program
	1. Means we need to restore context
^ THIS IS SCHEDULING
ALL THIS WORK IS PART OF THE OS (IN THE KERNEL) IN A MONOLITHIC OS.

When a new job starts, we need to insert a node into a linked list. --> hence no need for privilege
When it ends, we remove from linked list --> again, no need for privilege
Searching a linked list --> again, no priveliege
^ Maintaing the list, and scheduling is not a priveliege task.

But after we make the choice, we need to restore context (privellege) and allow the program to run. < only priveleged instruction

Again, 99% of OS is not privelege. In a monolithic OS,the priveleged work and the non-priveleged work is bundled together.
-
Microkernel (+ exokernel)
.> making the kernel as small as possible

We could say the OS doesn't do very much...not particularly useful.
If we don't want to add contraints to the OS

We have minimized the code. (the non-privelleged code) is moved into the user space (in a exokernel) process server
The decision is made in user mode
The microkernel (in kernel mode) enacts that decision.

We push everything out the the user space

---

Both of these are valid approaches. Linux is monolithic; windows is microkernel (technically, hybrid)