---
layout: post
title: How do operating systems work?
author: Job Hernandez Lara
---

*version*: 0.9.0, initial version, draft

### Introduction

Last two years or so I have been on a journey to study computer science fundamentals. Over the course of the last two years I have studied algorithms, distributed systems and compilers among other topics but operating systems is one of my favorite topics.

In what follows I will explain how an operating system works based on my studies of the MIT [xv6[(https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf) and “Operating System Concepts”.

Obviously, I am not an expert. But I enjoy this topic a lot and wrote this to learn.

### What is an operating system?

The operating system provides resources to the apps that you use. These resources include memory, CPU time, files, and I/O. Since a process is an abstraction for a running program then we can say the process is the main character in an OS. The OS allocates CPU time, memory, files and I/O to processes.

The OS provides services to user programs through an interface. What is the Unix philosophy for designing interfaces? The Unix interface, i.e., system calls, is simple and narrow which allows ease of implementation. Unix has a simple and narrow interface but this set of system calls allow combinations to make more complex and general functionality.

xv6 and modern operating systems such as MacOS and Linux offers *time sharing* transparently switches the CPUs among the processes.

#### Processes and threads

A process needs resources -- namely, CPU time, memory, files, and I/O devices. These resources are allocated to the process while it is running. How is the process concept related to web programming? Suppose there is a process running on the web browser and it is trying to send a request to the server. The process will take an URL as an input and it will then carry out the appropriate instructions and system calls to display the response (web page).

A process is an abstraction for a program in execution; each process has its own address space and its memory is laid out as follows: text section, data section, heap and stack. The text section consists of the executable code, the data section consists of global variables, the stack consists of data storage associated with invocations of functions such as parameters, return address, and the heap is associated with dynamically memory storage such as when you allocate memory in C with malloc; for example, in an x86-64 if you were to loop over an array and print each element you would divide the program into \textit{.data} and \textit{.text} sections. In the \textit{.data} section you would declare and initialize the array and in the \textit{.text} section you would have the instructions. But to clarify, a program by itself is not a process; a program is a passive entity. In contrast a process is an active entity consisting of a program counter that points to the subsequent instruction and consisting of the contents of the registers. A thread is the execution control center of a given process. "A thread represents the basic unit of CPU utilization." There could be multiple threads per process. A thread has its own stack but shares the same address space with other threads. This makes sense because threads need to be aware of the same program. Ultimately, user level threads need to be mapped to kernel threads for the CPU to execute. There are several models of this, namely, many-to-one, one-to-one and many-to-many. As you might expect, when the model is many-to-one, many user threads get mapped to one kernel thread.

#### CPU Scheduling

How does the kernel provide memory resources to processes? Page tables are the mechanism through which the kernel provides memory resources to the processes. A process requests memory from the kernel and in turn the memory is allocated. How does it do this?

The kernel time shares processes - it switches the CPUs among the processes. What happens when a process is not executing? The kernel saves the process' CPU registers and restores them once the process runs again. Multiplexing means switching a CPU from one process to another. Context switching means multiplexing. Why is scheduling important? Scheduling is important because operating systems run with more processes than the computer has CPUs. How does scheduling happen? In response to system calls or interrupts the kernel switches from one process to another by the following way. First, there's a user-kernel transition to the old process' kernel thread, a context switch to the current CPU's scheduler thread, a context switch to a new (the one the kernel is switching to) process kernel thread and a trap return to the user level processes the kernel was switching to. Each CPU has its own scheduler thread which makes sense since when switching from user process A to user process B there is a transition to the process' A kernel thread via a system call, a context switch to the scheduler thread, and a context switch to process' B kernel thread. So, this means that scheduling involves alternating from thread to thread. And remember a thread is what executes the given process' instructions. Shares memory with other threads but has its own stack. What happens when a process gives up the CPU? The process' kernel thread calls switch to save its own context and return the scheduler context. I think this is why the kernel switches from the old user process kernel thread to the scheduler thread. Switching from one thread to a new one involves saving the registers on the old thread and restoring the registers of the new one. A scheduler is essentially a thread in the CPU. There is one scheduler thread per CPU. Each schedule thread has the scheduler function.

A timer interrupt drives context switching, i.e., the switch from one process to another process.

The kernel maintains a page table for each process. Page tables enable the kernel to isolate the different process' address space and multiplex them into a single memory system. When the kernel context switches it also switches page tables. The page table for each given process describes the user address space; moreover, the kernel provides a page table that describes the kernel address space.


The CPU scheduler is responsible for deciding which processes to allocate to the CPU. There are several algorithms for CPU scheduling; the simplest is a first come, first served algorithm in which the first process that requests the CPU gets allocated to the CPU; this is accomplished with the FIFO queue data structure; the first process that enters the queue it is the first out of the queue. What is tremendously amazing about scheduling is that the user does not notice when the CPU switches from one process to another process. So, it is interesting to think how one would build this. Why does the cpu scheduler exist? In most operating systems there will be more processes than there are CPUs so there needs to be a way for the CPUs to work on all of these programs.

#### Interprocess communication

Often processes need to communicate with other processes because they may need to access main memory. This can lead to race conditions. A race condition happens when two or more processes are sharing data (from main memory for instance) and the final result depends on the order of the processes. If one process runs before another process it may change the result because the order is off. How do we avoid race conditions? The answer is mutual exclusion; mutual exclusion dictates that when two or more processes share memory the processes that are not using the shared data are excluded from accessing the data. Mutual exclusion is achieved by semaphores, mutexes. A semaphore is initialized to the number of resources; when a process wants to use a resource it calls the wait() function thereby decrementing the count. When the process releases a resource it performs a signal() thereby incrementing the count. When the count is 0 processes are blocked until it increments again. When a process modifies a semaphore value another process cannot access the same semaphore value -- i.e., cannot access the same resource.

#### Memory Management

The job of the memory manager is to keep track of the parts of memory that are being used, to allocate memory to processes and to deallocate memory when the processes are done with it. How does the memory manager accomplish this? It does this through an abstraction mechanism known as virtual memory. As discussed above a process is an abstraction for a program; likewise, an address space is an abstraction of main memory -- it gives the illusion that each process has its own main memory. The set of addresses that a process can use to address memory is called the address space. A further extension of this is called virtual memory whereby a computer can work with programs that have more memory than what the computer is capable of; the basic idea behind main memory is that a process has its own virtual address space which is laid out as pages where a page is a contiguous range of memory addresses. The given processor's instructions manipulate virtual addresses; for example when you write, in assembly, `movq immediate, address1`, the operating system translates the virtual address `address1` into a physical address. The operating system maps virtual addresses to physical addresses using *page tables*.
> ... virtual memory is not a physical object, but refers to the collection of abstractions and mechanisms the kernel provides to manage physical memory and virtual addresses.

 And it is also important to note that the given operating system maintains one page table per process; and the root of the given page table is written to a register before the page table gets processed. Consider the *xv6* Unix kernel programmed by MIT. *xv6* is tailored to the Sv39 RISC-V processor - the top 39 bits of a 64 bit virtual address space are used for this; in the Sv39 processor a page table is logically an array of $$ 2^{27} $$ page table entries (PTEs). Each PTE has a 44 bit physical page number. The paging hardware translates the virtual address by using the top 27 bits of the 39 bits to index the page table to find the PTE. The paging hardware also makes 56 bit physical addresses whose top 44  bits come from the physical page number of the page table entry and whose bottom 12 bits come from the virtual address. The page table is stored in physical memory as a tree whose root node consists of a 4096-byte page table. In turn, this page contains 512 PTEs that contain the physical addresses of the page table pages of the next level of the tree and in turn these pages contain the addresses of the page table pages of the final level of the tree. The OS uses the top level 9 bits of the 27 bits to select a PTE from the root page table page and further, it uses the middle 9 bits to select the PTEs from the middle level of the tree and finally it uses the bottom 9 bits to select the PTEs from the final page table page in the final level of the tree. *How does the kernel allocate and free memory?*The kernel must allocate and free memory at run-time for page tables, user memory, kernel stacks, etc. It allocates and frees 4096 byte pages at a time. The kernel maintains a linked list of free pages. Allocation of a page consists of removing a page from the list and freeing a page consists of adding a page to the list.

#### Traps, interrupts, and drivers

What goes on with the CPU when a given process is running? The CPU executes the processor loop, namely, read an instruction, advance the program counter, execute the instruction, repeat. Often, however, control must transition from user programs to kernel instead of executing the next instruction. This happens when a user program does something illegal such as dividing by 0, trying to access a page table entry for which there is no virtual address.

An *exception* triggers an interrupt when there's an illegal program action. An \textbf{interrupt} is a signal that tells the OS that it needs to pay attention to it; for example, a clock chip may generate an interrupt every 100 msec to allow the kernel implement time sharing. The kernel is responsible for handling the interrupts. When designing an OS/kernel one needs to be aware of how the underlying processor handles system calls, interrupts and exceptions; for example, on the x86 a program triggers a system call by generating an interrupt using the *int* instruction. Exceptions generate an interrupt too.

How do interrupts work? As mentioned above an interrupt stops the processor loop; once the processor loop stops the interrupt executes a new sequence called an *interrupt handler*. And of course before starting this new sequence, the processor saves its registers so it can resume after the OS has returned from the interrupt. So, this means that the transition from the processor loop to the interrupt handler involves going from user mode and kernel mode.

Interrupts in UNIX terms are known as *traps*.

How does xv6 and thus Unix invoke a system call? To invoke a system call a program indexes the interrupt descriptor table (IDT) by invoking the *int*  instruction where n is the index of the IDT.

#### Locking

xv6 runs on multiprocessors; this means that multiple CPUs are executing instructions independently and since there is only one physical address space and shared data structures, xv6 needs a mechanism to keep them from interfering with one another. This is what *locking* accomplishes. A lock achieves mutual exclusion which makes sure that one CPU at a time can hold the lock. In turn, this allows careful treatment of the data structures.

Since there are multiple processors, for a given line of code, multiple processors can execute the line which will result in problems. Locking allows multiple processors to execute the line of code without introducing unwanted changes to the data structure.

The reason why we need locks is to prevent *race conditions*. Race conditions result from the ordering of execution of two or more processors that invalidates a data structure. A processor S may be operating on A but then processor T may work on A which will result in processor S working on a modified version of A, let's call it B.

#### Deadlock

Deadlocks involve the ordering of locks. Suppose a code path require to take out several locks. When this happens it is important that all code paths acquire the locks in the same order. Suppose there's to code paths and both paths need locks A and B. If code path 1 acquires locks in the order A, B and the other path acquires locks in the order B, A. This can result in a deadlock because when path 1 acquires lock A, path 2 will acquire B. As a result path 1 will wait for and path 2 will wait for A. Both path are waiting. In other words, a deadlock happens when two processors hold the lock that each of them needs.

#### File System
The file system exists to organize and store data; it is an abstraction of disk.

##### Buffer Cache

The buffer cache has two jobs: 1) ensures that only one copy of a disk block is in memory; 2) cache popular blocks so they do not have to be re-read from the slow disk. The buffer cache has an interface; in xv6 the interface is `bread` and `bwrite`. The former gets a buffer consisting of a block that can be read or modified and the latter writes a modified block to disk. In other words, there is a transition between memory and disk. A block can first be in memory and it can be read and modified in memory. `bread` gets a copy of such block. `bwrite` writes this modified block to disk. So, there seems to be a transition between memory and disk. How does the buffer cache synchronize access to each block? It only allows at most one kernel thread to reference to the block's buffer. While one kernel thread is referring to a block buffer other threads will have to wait until the thread releases the buffer block.

##### Logging

Logging solves the problem of crashes during file system operations.

##### Block Allocator
File content and directory content are stored in disk blocks which are allocated from a free pool. So, the blocks that the buffer cache synchronizes contain file and directory content. The block allocator consists of a bitmap on the disk. Each bit in this bitmap corresponds to one block. If the bit is 0 then the block is free; otherwise, the block has content. The block allocator has two functions: `balloc` and `bfree` which allocates a block and frees a block respectively.

### Conclusion
Above I have tried to explain what an operating system is. Operating systems are fascinating systems. It is amazing that anything at all works in operating systems. You should pick up a textbook and study this topic. Enjoy.
