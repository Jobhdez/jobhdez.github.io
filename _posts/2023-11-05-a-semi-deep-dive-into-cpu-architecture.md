---
layout: post
title: A semi deep dive into CPU architecture
author: Job Hernandez
---

### Introduction

My compiler journey has made me curious about computer architecture; for example, I would like to know the x86 processor better. A great way to get close to the machine is by writing compilers because you get to lower high level programs to assembly. Assembly exposes the CPU and as a consequence you get to know the CPU better. 

I am not an expert but in what follows I try to give you an idea of how the architecture of a CPU. I first try to expose the x86 architecture by demonstrating how core imperative constructs such as loops and arrays and if statements get represented in x86 assembly. Subsequently, I talk about the low level details.

Hopefully, you enjoy this!

### Exposing the x86 architecture

x86 assembly exposes the registers of the x86 architecture and the program counter. The program counter is the address of the next instruction to be executed. When using memory addresses instead of registers these memory addresses are actually part of the virtual memory address space.

The program counter is called `%rip` in x86-64. The register file of x86-64 consists of 16 named locations storing 64 bit values i.e., registers.

The register `%rsp` is the stack pointer, which indicates the end position of the run time stack. The top of the stack contains the lowest addresses. The `pushq` instruction pushes values to the stack and `popq` pops them. The stack follows a last in, first out operation; for example, suppose you have two variables each of which are a byte in size (i.e., quad word values). Pushing these values to the stack would consist of decrementing the stack pointer by 16:

```
 pushq %rbp
 movq %rsp, %rbp
 subq $16, %rsp
 ```

The operands of the instructions defined by the ISA consist of immediates, stack locations, and registers. Immediates are data such as numbers, stack locations are memory locations and registers are locations that can be accessed by the processor in a few cycles.


To get to know the x86-64 architecture a little bit more I will talk about how core imperative programming features get implemented in x86-64 assembly where core features include arrays, loops, and if statements.

First, lets expose the x86 architecture when its executing `if statements`.

Consider the following program:

```python
if 3=3:
  x = 30 + -5
  print(x)
else:
  y = 25 + -10
```

Lets expose immediates and stack locations and the stack pointer register `%rsp` by lowering the above program to x86 assembly. Remember some of the operands in x86 instructions are stack locations and immediates.

```asm
.globl main

main:
      pushq %rbp
      movq %rsp, %rbp
      subq $16, %rsp     // the stack grows by decrementing the stack
      movq $3, -8(%rbp)  // move 5 into stack location 1
      cmp  $3, -8(%rbp)  // compare: 3 is not equal to 5
      je block_1
      jmp block_2

block_1:
      movq $5, -8(%rbp)
      negq -8(%rbp)
      movq -8(%rbp), %rax
      movq $30, -16(%rbp)
      addq -16(%rbp), %rax
      movq %rax, %rdi
      callq print_int
      addq $16, %rsp
      popq %rbp
      retq

block_2:
      movq $10, -8(%rbp)
      negq -8(%rbp)
      movq -8(%rbp), %rax
      movq $25, -16(%rbp)
      addq -16(%rbp), %rax
      movq %rax, %rdi
      callq print_int
      addq $16, %rsp
      popq %rbp
      retq

```

The first three instructions in the `main` block sets up the stack. It prepares the stack for the two variables and corresponding two stack locations.

In the background the compiler lowers the `x = 30 + -5` into

```
tmp = -5
x = 30 + tmp
```

So it first needs to move 5 to a stack location, make it negative, move 30 to stack location 2 and then add them together and put the result in `%rax`.

It does the same thing for block 2.

Now, lets expose the registers and arrays and loops.

Suppose you have the following program:

```python
for i in array:
   print(array[i] + 2)
```

The x86 assembly for this code would look something like this:

```asm
.section .data

array:
	.quad 1,2,3,4,5
	.section .text
    
	.globl main
    
main:
	leaq array(%rip), %rbx 
	movq $0, %r15   	// initializes i to 0
	jmp test

body:
    
	movq (%rbx, %r15, 8), %rdi     
	addq $2, %rdi // this adds two to a[i]    
	callq print_int // it prints a[i] + 2
	incq %r15 // increments i by 1
	jmp test

test:
	cmpq $5, %r15 // if i > 5 (size of array) go to body; otherwise exit
	jne body
	jmp exit

exit:
	retq
	
```

The instruction `leaq array(%rip), %rbx` uses the stack pointer to get the address of the array and stores it in the register `rbx`.

On the other hand the instruction `movq (%rbx, %r15, 8), %rdi` uses the address of the array, the initialization of i which is in register `%r15` and the size of the data in bits. Each element of the array is a byte in size. With this instruction you are getting `array[i]`. This essentially is equivalent to `xA + L * i` where `xA` is a pointer to the starting location of the array, L is the size of data type L.

### How the processor executes instructions

Now, lets talk about how the cpu executes instructions.

The processor consists of the control unit, the arithmetic logic unit, and a set of registers; the control unit is responsible for fetching instructions from main memory, the ALU is responsible for doing arithmetic and the registers hold the ALU input. The registers feed into ALU input registers which hold the ALU input while the ALU is performing some computation. There are also the ALU output registers which hold the output of the ALU and whose data can be sent to registers again or written to memory. How does the cpu carry out the instructions? The most important registers are the program counter and instruction register. The program counter points to the next instruction and the instruction register holds the current instruction that is being executed. The $$ \textit{data path} $$ consists of the ALU and registers; the registers feed into two input registers that hold the ALU input while the ALU is carrying out some computation; in turn these input registers are connected to the ALU. After the ALU finishes the computation it yields the output that gets stored in an ALU output register which in turn can go back to a register or later on stored in memory; the layered structure is as follows:

```
registers -> ALU input registers -> ALU -> output register -> registers or memory
```
How do instructions get executed? $$ \textbf{Fetch-decode-execute} $$ cycle. Instructions get executed as follows:

```
1. Fetch the next instruction from memory into the instruction register.
2. Change the program counter to point to the following instruction.
3. Determine the type of instruction just fetched.
4. If the instruction uses a word in memory, determine where it is.
5. Fetch the word, if needed, into a CPU register.
6. Execute the instruction.
7. Go to step 1 to begin executing the following instruction
```

#### Clock cycles

A clock is a circuit that emits pulses with a precise pulse width and a time interval between consecutive pulses. The time interval between two consecutive pulses is called the clock cycle time; pulse frequencies range from 100Mhz and 4Ghz corresponding to 10 nsec to 250 psec. This pulse frequency falls in the range of 4ghz. A lot of events happen during a given clock cycle; for example, the program counter is  loaded with an instruction address every clock cycle, the registers get updated every clock cycle and memory locations get written when a `mov`, `push` and `call` instruction get executed. The control of  the memory and registers by the clock cycle is  what allows the instructions to get executed in a sequence.

But in practice there's instruction parallelism.

#### Instruction parallelism 

$$ \textbf{Instruction level parallelism} $$ is a process whereby the processor executes more instructions per second; it does this by implementing pipelining whereby more stuff gets done in less processor cycles.

Instruction level parallelism is exploited in compilers.

The main idea of instruction level parallelism is to pre-fetch instructions from memory and put them in a set of special registers called the pre-fetch buffer and as a result of this the processor would not have to read from memory which takes hundreds of cycles. In a pipeline, execution is divided into stages in which each stage is carried out by a different type of hardware in parallel.

Suppose you divide the execution into five stages.

Stage 1 fetches instructions from memory and are stored in the prefetch buffer. Stage 2  determines the type and determines which operands it needs. Stage 3 fetches the operands; stage 4 carries out the execution and stage 5 writes the output to a register. Since these stages can be carried out in parallel it is fast; for example during the first clock cycle  Stage 1 does its job; during the second clock cycle Stage 2 does its job but also Stage 1 does its job for the next instruction. And during the third clock cycle Stage 3 does its job for the first instruction, stage 2 does its job for the second instruction, and stage 1 does its job for the third instruction and so on. Now, suppose that each clock cycle takes 2ns to complete; so one instruction takes 10ns to get processed by each stage so there 500 million instructions per second get executed.


### Conclusion
As you can see there is a whole world as to how computers work. Getting to know architecture is a very interesting thing. Obviously, I have a lot to learn and what I wrote above is just the very basics. 

Well, I have briefly discussed the cpu architecture and have exposed the x86 architecture through some basic high level programs. Hopefully, you enjoyed it. 


