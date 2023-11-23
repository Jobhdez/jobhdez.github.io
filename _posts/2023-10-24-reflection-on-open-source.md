---
layout: post
title: Reflecting on my open source experience
author: Job Hernandez Lara
---

*version*: 1.0, 11/22/23

*version*: 0.9.0, draft, 10/24/23

### Introduction

I have been reading “Rebel Code”, a book about the early days of Linux and open source and I just find the stories in this book amazing. So, I am really excited about contributing to Coalton, a programming language written in Common Lisp that is embedded in Common Lisp. Coalton is a statically typed language, similar to Haskell, with type inference.

In [14 Grand Challenges for Engineering in the 21st Century](https://www.engineeringchallenges.org/challenges.aspx) some experts got together and determined 14 grand engineering challenges for the 21st century. Among these challenges was to build the tools for scientific discovery.

It happens that Coalton is a language that is used in production at a quantum computing company to build a quantum compiler. So, Coalton is a tool for advancing science. So, I think the peope at HRL labs are doing great work. So, I would like to contribute to Coalton.

### Lessons I learned from contributing to open source

Contributing to open source is like doing an internship. If the system is being used in production then by contributing you are essentially doing an internship because you get to work on a production system and with other engineers.

One of the things I learned from contributing to Coalton is how to independently investigate the codebase, identify the relevant code I need to understand to make my changes; moreover, I learned that often you can make a contribution by following the patterns of the surrounding code. In addition, working on a github issue is also just like working on some personal project. That is, you still need to research the problem and understand the problem.

One of the things I also learned was about the importance of code review. I learned that software is improved iteratively through code reviews. As the lead, Robert and very good software engineer, said:

> It [the PR] started as a humble initial piece of code with the basic structure, and now it's being "chiseled" and refined. It's turning into a higher and higher quality contribution with each pass. This iteration is the essence of good software.

In addition I also learned that PRs should be a series of sensible and well crafted commits. Or as Robert Smith says:

> Commits should cover an atomic amount of useful change, and be well described by the short and long commit message. Most ideal scenario: what I described above. Less ideal scenario: you squash everything into a single commit and describe your changes. Less^2 ideal scenario: I squash them all for you and write a message for you. Less^infinity ideal scenario: I merge something with whatever unpolished commit history.

Finally, I learned about the amazing feeling you get when you contribute code to a codebase that gets used. I felt excited when my first non trivial pr got merged because I felt I did something useful.

Now, I will say a little bit about my contributions to Coalton.

### My contributions

#### Dual numbers and automatic differentiation

I started fixing typos but after a conversation I had with Robert, he suggested that I work on a dual number library.

A dual number has the form $$ a + b\epsilon $$ where $$ a $$ and $$ b $$ are real numbers and $$ \epsilon $$ is a symbol that satisfies $$ \epsilon^2 = 0 $$  and $$ \epsilon \neq 0 $$. One application of dual numbers is automatic differentiation.

Consider the function $$ f(x) = 5x+3 $$ and you want to calculate $$ f(3) $$ and $$ f'(x) $$.

We can rewrite $$ f(x) $$ as a dual number as follows:

$$ f(3 + \epsilon) = 5(3 + \epsilon) + 3 = 18 + 5\epsilon $$.

Here $$ 18 $$ is $$ f(3) $$ and 5 is the derivative of $$ f’(3) $$.

So, this is what I did. I wrote this library in Coalton for Coalton’s standard library. The code I wrote supports **Number** such as addition, subtraction, and multiplication and division, **Radical**, **Exponentiable** and **Trigonometric** data types.


#### Signaling when an unsigned value underflows

Another contribution that I am currently working on, which has not been merged, but has been approved, is detecting underflow of unsigned integers and signaling an error.

This pull request has taught me a lot and taught me a good lesson. Namely, it is good to gain some practical experience and then read a textbook to learn some theory. Practice and theory go together.

In the textbook “Computer Systems, A Programmer's Perspective” the authors claim the following:

> By studying the actual number representations, we can understand the ranges of values that can be represented and the properties of the different arithmetic operations. This understanding is critical to writing programs that work correctly over the full range of numeric values and that are portable across different combinations of machine, operating system, and compiler

In response to reading the above quote, which was after I started working on this issue, I appreciated taking the time and researching what an unsigned value is and the arithmetic properties such as when it overflows and underflows.


So, as I mentioned above, I picked up an issue that involved signaling an underflow error for unsigned values. Unsigned values are numbers in the range $$ 0 $$ to $$ 2^{w} - 1 $$ where $$ w $$ is the the size of the unsigned number; for example, an 8 bit unsigned number. In other words, a bit vector $$ \vec{x} $$ of $$ w $$ bits can represent a decimal number in the range  $$ 0 $$ to $$ 2^{w} - 1 $$.

This is saying that the set of decimal numbers that the computer represents is contingent on the bit size of the number and whether it is an unsigned or signed number; for example, an 8 bit unsigned number can represent values ranging from $$ 0 $$ to $$ 255 $$, a 16 bit unsigned number can can represent values ranging from $$ 0 $$ to $$ 65535 $$, a 32 bit unsigned number can represent values ranging from $$ 0 $$ to $$ 4294967295 $$ and a 64 bit unsigned value can represent values ranging from $$ 0 $$ to $$ 18446744073709551615 $$. 

Recall that a $$ w $$-bit number is stored as consecutive addressable bytes. So, a $$ 8 $$-bit unsigned integer gets stored in one byte whose address could be 0x100. In contrast, a $$ 32 $$-bit unsigned number is stored in 4 bytes whose address locations may be 0x100, 0x101, 0x102, 0x103.

Given a bit vector $$ \vec{x} $$ one converts a bit vector into a decimal number as follows:

$$ B2U_{w} $$ $$ \vec{x} $$ $$ = $$ $$ \sum_{i=0}^{w-1} x_{i}2^i $$

So, I learned a lot thinking about this github issue.

### Conclusion

Above I have said a little bit about the lessons I have learned from contributing to Coalton. I learned how to investigate a codebase – e.g., identifying the relevant source code/files – and how code review leads to an iterative process for improving software. I also wrote about my contributions which consisted of building a dual number library, and on a current PR involving detecting unsigned number underflow.
