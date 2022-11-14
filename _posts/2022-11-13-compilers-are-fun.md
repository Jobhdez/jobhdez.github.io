---
layout: post
title: "Compilers and why you should build one"
author: Job Hernandez
---

# Building compilers involve great engineering
The complexity of compilers is tremendous; for example, in one large program you can have unlimited variables but yet there is only a limited amount of registers. This is quite amazing. Or consider a massive program and think about all the work that goes into the compilation of this massive program – mostly every module of the compiler has to work correctly. The amount of engineering that goes into building a system such as GCC or LLVM is amazing. Compilers are beautiful systems. I find it remarkable how computer scientists figured all this out and the fact that a 15 million line compiler such as GCC works at all.

# My experience
Lately, I have been working on toy small compilers and I am having a lot of fun; for example, I have been working on a compiler that compiles core imperative features such as `while loops`, `if statements`, `assignments` and `variables` to x86-64; nevertheless, it is not quite done; for example, you cannot really run the assembly code, and the references to variables that are created after the compiler pass that replaces variables for registers and stack locations are off by a bit and off by a lot more more if the respective function is subsequently called. Despite these limitations mostly correct assembly is emitted from expressions such as 
`x = 30 + -2 print(x)` or `x = 40 + -20 print(x + 10)`. The largest program that gets compiled in my system is 

```
if 2==2:
   x = 40 + -3
   print(x)
else:
   y = 3 + -1
   print(y)
```

Although the furthest I have gotten is processing this program with the instruction selection module which takes the high level program and lowers it to x86-64 instructions but without replacing variables with registers and stack locations.

# Why you should learn compilers
But the point I want to make is that even if you write such a limited compiler you will gain better appreciation of how computers work; that is, you will gain experience how some of the language constructs you use as a programmer get mapped to a sequence of instructions and this will, in turn, give you an insight as to how computers work; for example, you will have concrete examples of how values get moved to registers. Making sense of assembly code felt like my studies of Logic — e.g., when studying `hypothetical syllogisms` which are forms of deductive arguments. In these logical forms everything flows logically. Assembly code feels this way to me. So, in a way you will also start appreciating studying discrete mathematics and doing proofs because this prepares you to think deeply about computer science. Not that I have much experience thinking deeply about computer science but I am realizing that computer science theory can help form you into a thinking individual.  Furthermore, I think learning and writing compilers, even if they are toys, but especially if they are more serious is analogous to the claim by Richard Feynman. He said something like:
>To appreciate nature you need mathematics. 

Likewise, learning and building compilers that emit assembly will help you appreciate computer science and appreciate how computers work. With my project I barely scratched the surface; for example, I still need to implement register allocation whereby all the variables in your program get stored in the limited number of registers there are. Implementing register allocation will help you appreciate computer science because the underlying algorithm is a graph problem. And algorithms are fundamental to computer science. In a way compilers have mathematical structure — the parse tree is a tree, intermediate languages can be either abstract syntax trees or graphs, and sub-systems such as the register allocator involve graph algorithms. Trees and graphs are mathematical objects.

Overall, I am glad I started working on toy compilers because working on them expanded my mind and if you work on compilers as well this experience will expand your mind as well.
