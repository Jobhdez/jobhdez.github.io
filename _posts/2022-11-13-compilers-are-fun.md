---
layout: post
title: "Why you should build a compiler :)"
author: Job Hernandez
---

*version 1.1*: add better support for my claim

*version 1*: initial version

## My background
Lately, I have been working on toy small compilers and I am having a lot of fun; for example, I have been working on a compiler that compiles core imperative features such as `while loops`, `if statements`, `assignments` and `variables`, `tuples` and `functions` to x86-64; nevertheless, it is not quite done; for example, as of `7/3/23` it lowers the source language to an x86 AST. The x86 AST is mostly correct so I just now have to create the x86 file and finish up tuples which require a 64 bit tag.

So, as you can see I am not an expert compiler engineer but I have done some work and as a result I have learned a lot. So, the following statements are based on my observations from implementing my compiler and studying compiler textbooks.



## Why you should build a compiler
In the book “Selected Papers on Computer Science” Donald Knuth wrote:
>Perhaps I can summarize all of these remarks by saying that methods are more important than facts. The educational value of [toy programs] a problem given to a student depends mostly on how often the thought processes that are invoked to solve it will be helpful in later situations. It has little to do with how useful the answer to the problem may be.

This above quote resonated with me because it helped me realize that compilers, even if you build toys, are valuable because of the methods and knowledge you gain from these systems that can be useful in later situations.

### Learn compilers to gain a better understanding of computer science
Dr. Knuth claimed in his book `Selected Papers on Computer Science` that algorithms are the main object of study in computer science. And I agree. Everytime you write a program you are writing an algorithm. As a consequence, considering that algorithms are the main object of study in CS and considering that graph theory is essential in compiler engineering then by studying and implementing a compiler one will get a great education in computer science.

It is remarkable to think how compilers compile large systems such as the linux kernel. Imagine how the mathematical structures – i.e., graphs – enable the efficiency of the generated code. This will help you appreciate how important graph algorithms are and how important algorithms are in general. 

In addition, you  will gain a better understanding of a big chunk of the system stack; for example, implementing a toy x86 compiler will teach you how memory is laid out in processes, it will give you a better idea of how a high level program is represented as assembly which in turn will give you a better understanding of how a processor executes a program; moreover, you will learn how garbage collection works; for example, when implementing tuples or arrays you will learn how these are heap objects and how garbage collection manages heap memory for these; furthermore, implementing a compiler will also give a much better appreciation of the memory hierarchy; for example, you will learn why register allocation improves the performance of the generated code. Register allocation is the process by which registers are used instead of stack locations. Register allocation gives light into the memory hierarchy because the cpu takes 0 cycles while fetching from main memory takes hundreds of cycles. As a result of implementing register allocation you will have a concrete example regarding the memory hierarchy. If you dig deeper you will learn that  caches take a few cycles, a few cycles more than register but considerably less cycles then fetching from main memory. An interesting thing I learned from studying how systems work is how the disk within a client-server architecture is part of this memory system. The memory hierarchy consists of:

```
Registers
   |
   V
Caches 
   |
   V
Main memory
   |
   V
Disk
   |
   V
Disk in remote network server
```
As you go down the memory hierarchy, from registers to disk in a remote server, the number of cpu cycles needed to fetch from memory increases. As a programmer it is important to visualize the consequences of the programs one writes.

You may be asking what use can this level of understanding have. I have read engineering blogs from facebook such as this [article](https://engineering.fb.com/2015/06/26/security/fighting-spam-with-haskell/) where the engineers were able to improve the performance of the system by improving the garbage collector of the Haskell’s GHC compiler; moreover, this same FB blog post also talks about how they analyzed how their system used cpu and memory resources. So, I think understanding compilers and hence systems can pay off when you are working on network applications. 


But the point I want to make is that even if you write such a limited compiler you will gain better appreciation of how computers work; that is, you will gain experience how some of the language constructs you use as a programmer get mapped to a sequence of instructions and this will, in turn, give you an insight as to how computers work; for example, you will have concrete examples of how values get moved to registers. Making sense of assembly code felt like my studies of Logic — e.g., when studying `hypothetical syllogisms` which are forms of deductive arguments. In these logical forms everything flows logically. Assembly code feels this way to me. So, in a way you will also start appreciating studying discrete mathematics and doing proofs because this prepares you to think deeply about computer science. Not that I have much experience thinking deeply about computer science but I am realizing that computer science theory can help form you into a thinking individual.  Furthermore, I think learning and writing compilers, even if they are toys, but especially if they are more serious is analogous to the claim by Richard Feynman. He said something like:
>To appreciate nature you need mathematics.

Furthermore, by building a compiler you will understand modern programming languages better. People say that to understand something you need to teach it but to really understand something you need to implement it. So, by implementing a programming language you will understand constructs such as lexical scoped functions, efficient tail calls, dynamic and statically typed languages. And you will also gain an understanding of how tuples and arrays are lowered to assembly and also understand how these are heap objects.

### Learning about and implementing a compiler will empower you to make a big impact
According to this [post](https://www.embecosm.com/2018/02/26/how-much-does-a-compiler-cost/) GCC is about 5 million and LLVM is about 1.6 million. So, the complexity of compilers is tremendous; it is also amazing to think that computer scientists figured out how to build compilers and it is also amazing that such large and complex systems such as compilers work at all.

Once you have implemented a compiler you will have a good understanding to contribute to GCC or LLVM. Now, think about the tremendous impact you will have if you contribute to LLVM. LLVM is used to compile Android and Apple apps, and it is crucial for deep learning. A lot of the infrastructure depends on LLVM so by contributing to LLVM you will have an enormous impact. 
