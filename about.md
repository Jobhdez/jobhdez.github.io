---
layout: page
title: About
permalink: /about/
---

Hello, I am Job Hernandez and I am a programmer from Seattle, WA. Some of my interests include compilers, functional programming, and pure mathematics; in addition, I also enjoy thinking about operating systems and distributed systems. But I also enjoy making Django backends. Fortunately, I have learned that programming is fun irrespective of the what you're building or which programming language you are using. Personally, I get a kick out of solving problems and debugging -- i.e., making a program run after spending some time debugging it.

### Software Projects
I write software as a hobby but for me programming is a serious hobby. Some of my projects include:

#### Independent projects

Here is my favorite compiler I have written based on "Essentials of Compilation" by Dr. Siek. This compiler is loosely based on this book but I used a different but similar language.

- [zettapy](https://github.com/jobhdez/zettapy), a compiler that lowers the core of an imperative language to an x86 AST.

#### Open Source contributions

I have also contributed to open source in non trivial ways; for example, Coalton is used in production for the development of a quantum compiler. My biggest contribution to Coalton and in open source in general is a library for automatic differentiation via dual numbers. You can see the original PR [here](https://github.com/coalton-lang/coalton/pull/890). It was merged by a core developer [here](https://github.com/coalton-lang/coalton/pull/926).


- [coalton](https://github.com/coalton-lang/coalton), a statically typed programming language embedded in Common Lisp.

In addition I have also fixed some typos for TVM which is a compiler for deep learning. You can see my PRs [here](https://github.com/apache/tvm/commits?author=jobhdez). I chose to say something about these contributions despite these PRs being simple because the last typo I noticed and fixed was as a result of trying to figure out how a deep learning compiler such as TVM extracts the computational graph of a neural network model. What I learned was that for Pytorch models, TVM uses torchscript. From the torchscript representation I think you make the AST and continue from there.

### Some fascinating quotes about programming

I love programming and computer science and here are some of my favorite quotes:

**This is a quote taken from SICP, which is one of the best CS textbooks, about what programming entails**:

> I think that it’s extraordinarily important that we in computer science keep fun in computing. When it started out, it was an awful lot of fun. Of course, the paying customers got shafted every now and then, and after a while we began to take their complaints seriously. We began to feel as if we really were responsible for the successful, error-free perfect use of these machines. I don’t think we are. I think we’re responsible for stretching them, setting them off in new directions, and keeping fun in the house. I hope the field of computer science never loses its sense of fun. Above all, I hope we don’t become missionaries. Don’t feel as if you’re Bible salesmen. the world has too many of those already. What you know about computing other people will learn. Don’t feel as if the key to successful computing is only in your hands. What’s in your hands, I think and hope, is intelligence: the ability to see the machine as more than when you were first led up to it, that you can make it more. - Alan Perlis

**The following quote is taken from one of Paul Graham's essays about how Lisp is analogous to the quicksort algorithm; so the idea that Lisp is outdated or obsolete is wrong because Lisp is a piece of math like quicksort. This quote gets me enthusiastic about Lisp**.

>  Another way to show that Lisp was neater than Turing machines was to write a universal Lisp function and show that it is briefer and more comprehensible than the description of a universal Turing machine. This was the Lisp function eval..., which computes the value of a Lisp expression.... Writing eval required inventing a notation representing Lisp functions as Lisp data, and such a notation was devised for the purposes of the paper with no thought that it would be used to express Lisp programs in practice. - McCarthy


**The following quote is really meaningful because I also love math and because this quote taught me that you can see programming languages and programs as mathematical objects. It is taken from Dr. Pierce's Software Foundations course website**.

> This course introduces basic concepts and techniques in the foundational study of programming languages, as well as their formal logical underpinnings. The central theme is the view of individual programs and whole languages as mathematical objects about which precise claims may be made and proved.