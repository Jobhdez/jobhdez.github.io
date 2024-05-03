---
layout: post
title: "A semi deep dive into Python"
author: Job Hernandez
description: "a brief description of the Python language"
tags: python compilers internals
---

## Introduction
I am on a journey to understand programming deeply or as deeply as I can and as a result I want to understand the programming languages I use. In this article I will explore the Python programming language; specifically, I will briefly explain the execution model of Python and explore some features that experienced engineers may argue are conducive to building large systems.

After writing this blog post I have realized that Python is indeed a powerful language. Python3 is strongly typed with dynamic and static guarantees (with mypy). This is similar to Lisp which is what made it so much better back in the day; moreover, Python has first class functions so you can build powerful abstractions using higher order functions. It allows incremental development, perhaps not as interactive as Common Lisp + SLIME but it does offer a REPL nonetheless which enables you to experiment with your programming ideas, get feedback and develop programs that have few bugs (since you have already tested your ideas on the REPL). Python also has a great community and has a great library ecosystem.

Consider what Peter Norvig [said](https://norvig.com/python-lisp.html) about Python. He considers Python to be a dialect of Lisp:

>Basically, Python can be seen as a dialect of Lisp with "traditional" syntax (what Lisp people call "infix" or "m-lisp" syntax). One message on comp.lang.python said "I never understood why LISP was a good idea until I started playing with python." Python supports all of Lisp's essential features except macros, and you don't miss macros all that much because it does have eval, and operator overloading, and regular expression parsing, so some--but not all--of the use cases for macros are covered.

## A quick dive into Python's execution model

According to the Python [CPython internal docs](https://devguide.python.org/internals/compiler/) Python3 consists of a compiler and stack-based virtual machine. So, as any other compiler architecture it consists of a frontend and backend. The frontend consists of a parser that generates the abstract syntax tree which in turn gets converted into a control flow graph which in turn gets compiled into bytecode, but unlike a native compiler, this bytecode gets executed by a stack-based virtual machine. Java also consists of a similar architecture including a stack-based virtual machine.

To illustrate the above points I will provide some concrete examples. I  have been working on a compiler for a language that has Python syntax. Although this compiler generates x86 assembly it consists of a parser that generates an abstract syntax tree. 

An abstract syntax tree (AST) is a tree that represents a given program. You can represent nodes in a tree with objects; for example, the `if` statement node can be represented by the following Python object:

```
Class IfStmt:
    def __init__(self, condition, then, else):
        self.condition = condition
        self.then = then
        self.else = else

     def __repr__(self):
         f’(IF {self.condition} {self.then} {self.else})’
```
Once you have your node objects you recursively go through the parse tree and, for example, when you hit an `if` statement you construct the AST node using the `IfStmt` object.

Consider the following program:

```
if 2==2: 
    x = 30 + -10 
    print(x + 10)
else: 
    y = 34 + -2 
    print(y)
```

The parser will generate the an abstract sysntax tree that looks *similar* but not exact to the following:

```
#S(PY-MODULE
   :STATEMENTS (#S(PY-IF
                   :EXP #S(PY-CMP
                           :LEXP #S(PY-CONSTANT :NUM 2)
                           :CMP :==
                           :REXP #S(PY-CONSTANT :NUM 2))
                   :IF-STATEMENT (#S(PY-ASSIGNMENT
                                     :NAME ZETTA-VAR::X
                                     :EXP #S(PY-SUM
                                             :LEXP #S(PY-CONSTANT :NUM 30)
                                             :REXP #S(PY-NEG-NUM
                                                      :NUM #S(PY-CONSTANT
                                                              :NUM 10))))
                                  #S(PY-PRINT
                                     :EXP #S(PY-SUM
                                             :LEXP #S(PY-VAR
                                                      :NAME ZETTA-VAR::X)
                                             :REXP #S(PY-CONSTANT :NUM 10))))
                   :ELSE-STATEMENT (#S(PY-ASSIGNMENT
                                       :NAME ZETTA-VAR::Y
                                       :EXP #S(PY-SUM
                                               :LEXP #S(PY-CONSTANT :NUM 34)
                                               :REXP #S(PY-NEG-NUM
                                                        :NUM #S(PY-CONSTANT
                                                                :NUM 2))))
                                    #S(PY-PRINT
                                       :EXP #S(PY-VAR :NAME ZETTA-VAR::Y))))))
```

The above example was taken from my compiler project but it should give you a concrete idea of what an AST is.
                                      
After the AST is generated the Python system creates a control flow graph. A control flow graph is a directed graph. It takes the above Python AST and creates a graph similar to the following.
```
if 2==2:
   goto block1
   goto block2
block1:
    x = 30 + -10 
    print(x + 10)
block2:
    y = 34 + -2 
    print(y)
```
                                   	
It is not straightforward as that because you need to convert the block:

```
x = 30 + -10
print(x+10)
```

into an intermediate language that is closer to the bytecode but the CPython internals article does not talk about this; moreover, it is also not as straightforward because expressions such as `if expressions` generate more blocks and to generate the blocks in the most efficient way possible you also need to use graphs.

After the control flow graph is generated then the Python3 system lowers this control flow graph to bytecode. Since the Python's virtual machine is stack-based it pushes constructs to the stack and pops them; for example, suppose you are adding two numbers: `2+2`. Then `2`, `2`, and the addition operator `+` will be pushed to the stack; to execute the virtual machine will pop those and push `4` to the stack.

## A quick dive into some of Python's programming constructs

I don’t have experience with building large systems as I have not started my career in software engineering so I don’t have first hand experience with this topic; nevertheless, I would like to explore this topic because I want to improve my understanding of the Python3 programming language. As a result I have decided to use as a guide a few points that Robert Smith made in one of his [writings](https://github.com/stylewarning/deprecated-coalton-prototype/blob/master/thoughts.md) where he explains what it takes to build large systems. Robert is very experienced – he has built state of the art compilers and linear algebra systems, and a programming language (and many more things) that are used in production at a couple quantum computing companies. So, yes his metric is worth considering. Anyways, these are the points he makes:

```
- Does the language manage names well? Are there ways of creating new namespaces that won't likely collide with others'?
- Does the languages' implementation offer any static or dynamic guarantees of correctness? Do we know that certain classes of errors are not possible?
- Does the language intrinsically, or the implementation explicitly, permit one to incrementally change programs?
- Does the language offer or allow to be expressed re-usable abstractions? What kinds?
```

I will briefly explain a little bit about the above points and try to use what I have learned from my computer science studies to evaluate. I am mostly doing this as a reflection.

### Namespaces

In Python3 a namespace is a mapping from names to objects. They are implemented as dictionaries. According to the Python3 [docs](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces) there are three types of namespaces: built in namespaces, module (global names) namespaces and local namespaces. There are no relationship between the names in different modules so you can have a function named `square` in two modules and there will be no name conflicts; nevertheless, each respective `square` name has to be prefixed by the name of the module. The namespace consisting of the built in names is created when the interpreter starts up. On the other hand, the namespace consisting of global names of a given module is created when you import the module. And finally the namespace consisting of local names is created when a given function is called and deleted when the function returns.

From an elementary point of view  based on the classic introductory textbook “Structure and Interpretation of Computer Programs  (SICP)”, Python's organization of namespaces, especially the module namespace, is conducive to building large software systems. In SICP the authors explain that by isolating different parts of the system as packages, multiple developers can work independently on different parts of the system without introducing name conflicts. And it makes a system modular. So, namespaces in Python are conducive for building large systems or at least it is at a basic level. But as I said, I do not have first hand experience building large systems (in Python) so take my evaluation with a grain of salt.

### Does Python3 offer any static or dynamic guarantees of correctness?

That is, as Robert points out, does Python offer us any certainty that a whole class or errors will not happen? Well as you know, Python3 is a dynamically typed language so the Python3 implementation does not have a type checker; it consists of tags instead. Dr. Siek, author of a very good compiler textbook called “Essentials of Compilation” explains a dynamically typed language to be:

> In dynamically typed languages, …, a particular expression may produce a value of a different type each time it is executed.

Consider the following example:

```
d = lambda x: 3 + x if x == 3 else False
d(3) ## -> 6
d(4) ## -> False
```

The function `d` yields an `Int` or `Bool` depending on the input. This is not possible in a statically typed language. This means that the function `d` is polymorphic which is not to be confused with subtype polymorphism or parametric polymorphism. 

Now that I have explained what dynamic typing is, what guarantees does Python offer? Python is *strongly typed* and dynamically typed. In a strongly typed language the expression `2 + “hello”` will yield an error. The Python interpreter is smart enough to know that an integer and a string cannot be added together. In contrast, for example in Javascript the expression `2 + “hello”` evaluates to `2hello`. Python respects the semantics of the language but Javascript does not. Robert Smith puts it like this:

>The strong-typedness of Common Lisp ensures that a program which has a type error cannot descend into a state that no longer respects the predictable semantics of the language.

Python also offers static guarantees if you use `mypy` and given that Python has a garbage collector it guarantees that there will not be a certain class of memory error.

Given that Python offers dynamic and static guarantees and memory guarantees but especially because it offers static guarantees with mypy I think Python can be used to build large systems. As Robert said:

> Programs can get large. Along with that comes challenges in comprehension, modification, and maintenance.

Or as I learned in the mitocw Software Construction course, statically typed languages are more conducive to programs that are easier to understand, safe from bugs, and ready for change. As a result I think Python is a good language to build large software system but of course my answer to this question is theoretical and based on Robert’s thinking.

### Does Python allow the expression of re-usable abstractions? What kinds? And does it allow an incremental approach to development?

I think the answer here is that yes it does. Since functions are first class objects in Python then you can build higher order functions which lead to a more modular system since you can reuse the higher order function multiple times without repeating yourself. Python also offers an object oriented system which is really good too.

Python also allows the programmer to incrementally develop a program because it has a repl.

## Conclusion

Well, there you have it. We can still go deeper. If we want to understand Python deeper then we can build compiler and implement features such as tuples, arrays, lexical scoped functions, first class functions, object system. In addition we can implement a type checker, efficient tail calls, generics to get a much better understanding of programming in general.