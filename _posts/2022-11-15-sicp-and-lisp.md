---
layout: post
title: "SICP and Lisp"
author: Job Hernandez
---

### Personal experience with SICP 
About 5 years ago I started learning Scheme, my first programming language, by reading `The Little Schemer` by Felleisen. But eventually I stumbled across `SICP` and it changed my life as a programmer. `SICP` ultimately taught me to program — I learned by imitating the fraction system and symbolic differentiator. 

Now, SICP is one of the best books I have studied. The way the authors described ideas such as higher order procedures and data abstraction gave me a tremendous amount of intellectual joy; I cried of intellectual joy a few times. `SICP’s` focus on interpreters and compilers shaped me as a programmer – it put me on a path for learning and building compilers; moreover, considering that I learned programming by studying `The Little Schemer` and `SICP` I learned how to express myself recursively. In addition I learned to build programs with data abstraction and decompose a problem into small procedures. 

### SICP’s important lessons
But more importantly, `SICP teaches you techniques for controlling complexity.` One of these techniques is *abstraction*. Although I do not have experience working on very large systems I bet that large systems are hard to maintain. So this book tells you explicitly how to control this complexity and emphasizes it which is not something I have seen in other computer science textbooks I have read. Another technique they teach for controlling complexity is building *interpreters*, *DSLs*, and *compilers*. As I will mention below Lisp is a great language for carrying out these techniques.


### Common Lisp programs grow organically
For a while I lost my enthusiasm for Lisp but, lately, I have been slowly exploring Lisp to write compiler software and to understand its benefits. I have not written an enormous amount of code but this is what I think so far. 

In `Software Design for Flexibility` the authors, one of which also wrote SICP, explains that one needs to build software that adapts and evolves as requirements change. They argue for a style of programming called `additive programming`. The authors say that one should be able to add to the system to add new functionally instead of modifying the existing code base. Small changes to the system should entail small changes to the software.

I am not too sure about the techniques in this book for accomplishing this but I do think that Lisp allows you to write software that adapts and evolves as new requirements arise. What I have noticed with Lisp is that it makes it easy for me to add functionality very easily. That is, I can write a version of my program that covers some functionality and if I want to add more functionality, I can just embed an expression without having to modify much of the existing program. So, since everything is an expression in Lisp it allows you to grow your program organically. So, it seems like the programs you write in Lisp are like biological systems that adapt and evolve. 

### Functional Lisp and the ultimate abstraction
In addition to this, you get the benefits of functional languages: first class procedures which increase the level of abstraction and as a result helps you control complexity and increase modularity. Some Lisps such as Racket have pattern matching built in but if this is not the case such as in Common Lisp (SBCL) you can use libraries. And it is very natural to express yourself recursively. These qualities bring me to another point. SICP also teaches that building interpreters and DSLs and compilers are the *ultimate abstraction*; for example, you may write a DSL to express yourself directly in the conceptual units of the given domain. Lisp’s functional features and its flexibility allow you to build DSLs or compilers very easily and as a consequence Lisp is a great language for controlling complexity of software systems: has great mechanisms for abstraction – e.g., first class procedures – and is a great language to build compilers and DSLs.

### Finally
So, with Lisp you have a language where the programs written in it grow organically and adapt and you also have a language that allows you to control complexity of software systems. As a result, Lisp is a great language for software engineering. 

Thanks.
