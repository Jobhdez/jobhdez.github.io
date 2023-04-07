---
layout: post
title: "SICP's important computing and programming lessons"
author: Job Hernandez
---
# Personal experience with SICP
The book “Structure and Interpretation of Computer Programs” was one of my favorite experiences I have had in the past 10 years. My copy of SICP is also very special because my mom gave it to me. The way it was written and how it uses mathematics made a huge impression on me. I got my mind blown a few times and I cried of joy while reading a few parts. It was such an amazing experience. SICP also made me love mathematics more and it helped me realize how deep computer science is. The way the authors of SICP described higher order procedures – e.g., how higher order procedures elevate the conceptual level of a given program – was a tremendously meaningful experience for me. So, in other words Scheme and SICP helped me appreciate computer science. In addition, my SICP studies helped me understand that interpreters/DSLs and compilers are the ultimate abstraction which is key for controlling complexity.

# SICP’s important lessons
## Lesson 1: The Scheme Programming language
### Scheme is a model of computation
I think the Scheme programming language is such a profound language; Scheme is an extension of the lambda calculus which essentially means that Scheme can be seen as a model of computation. The lambda calculus models all non imperative constructs of programming languages as applications of functions and specifies with axioms the semantics of these expressions. I think the key idea here is the substitution model of computation. Consider the following:
```
(define (multiply-squares n n2) (* (square n) (square n2)))
(define (square n) (* n n))

;; substitution model ->

(multiply-squares 3 4)
     	|
     	V
(* (square 3) (square 4))
     	|
     	V
(*  (* 3 3) (* 4 4))
     	|
     	V
 	(* 9 16)
     	|
     	V
    	144
```
In the above example we evaluated the expression (multiply-squares 3 4) - then each expression is replaced by an expression that is equivalent via an axiom of the lambda calculus. Another example of the substitution model is the the evaluation of a recursive function that generates a recursive process whereby a chain of expressions is built up and whereby the chain of expressions then contracts as expressions get evaluated. Here is the Scheme example:
```
(define (factorial n)
   (if (= n 0)
   	1
   	(* n (factorial (- n 1)))))
```
The process that factorial generates is the following:
```
(factorial 3)
(* 3 (factorial 2))
(* 3 (* 2 (factorial 1)))
(* 3 (* 2 (* 1 (factorial 0))))
(* 3 (* 2 (* 1 1)))
(* 3 (* 2 1))
(* 3 2)
6
```
As you can see in the above example a chain of operations is built up and when it reaches the base case the expressions start getting evaluated. At each point the expressions are replaced with an equivalent expression according to an axiom of the lambda calculus.

### Tail Recursion
One concept I learned from reading this book that not many programmers realize is tail recursion. That is, it is possible to write a recursive procedure that generates an iterative process. In most programming languages a recursive procedure generates a recursive process whose time complexity is O(n) and whose space complexity is O(n); an example of a recursive process is the `factorial` process above; but in Scheme, a recursive procedure may generate an iterative process – i.e., a process whose time complexity is also O(n) but whose space complexity is O(1). The real insight here is that for a given procedure that generates a recursive process one can also write a procedure that generates an iterative process. In Scheme there are no looping constructs for instance. Here is an example a recursive procedure that generates an iterative process (taken from SICP):
```
(define (factorial n)
  (factorial-iter 1 1 n))

(define (factorial-iter product counter max_count)
  (if (> counter max_count)
  	product
  	(factorial-iter (* counter product)
                  	(+ counter 1)
                  	max-count)))
```
The process that the above generates for `(factorial 6)` is as follows:

```
(factorial 6)
(factorial-iter 1 1 6)
(factorial-iter 1 2 6)
(factorial-iter 2 3 6)
(factorial-iter 6 4 6)
(factorial-iter 24 5 6)
(factorial-iter 120 6 6)
(factorial-iter 720 7 6)
```
## Lesson 2: Functional Programming
One of the important lessons in computing and programming that you will learn from SICP is functional programming. Functional programming is functional in the mathematical sense; that is, an input to a function has exactly only one output. The programs in the first two chapters in the book are functional; moreover, functional programming is a type of programming that satisfies the substitution model of computation. Programs in functional style satisfy the substitution model because functional programs do not entail mutating variables. Variables in functional programming are definitions.

## Lesson 3: Higher Order Procedures
Higher order procedures are procedures that consume other procedures or return other procedures. Higher order procedures increase the conceptual level of the given program; instead of thinking in terms of particular sums, higher order procedures allow you to abstract the summation pattern and apply this pattern to instances of summations. Modularity is the consequence of this because instead of repeating code with multiple concrete summations you get to have an abstraction applied multiple times. Consider the following to summations:

```
(define (sum-integers a b)
  (if (> a b)
  	0
  	(+ a (sum-integers (+ a 1) b))))

(define (sum-cubes a b)
  (if (> a b)
  	0
  	(+ (cube a) (sum-cubes (+ a 1) b))))
```

Here you have two instances of a pattern. WIth a higher order procedure you can do this instead:

```

(define (sum term  a next b)
  (if (> a b)
  	0
  	(+ (term a)
     	(sum term (next a) next b))))

(define (sum-squaress a b)
  (sun square a inc b))
(define (square n) (* n n))


(define (sum-integers a b)
  (sum identity a inc b))
(define (identity x) x)
```
## Lesson 4: Data Abstraction
Data abstraction is a way to form compound data; data abstraction also increases modularity because data abstraction allows you to separate the use from the implementation.

```
;; data abstraction for fractions
(define (numerator fraction) (car fraction))
(define (denominator fraction) (car (cdr fraction)))
```

## Lesson 5: Data Directed Programming
Data directed programming is a technique that allows multiple programmers to work on different parts of a system; moreover, it is a type of additive programming where instead of making changes to add new functionality you instead add a module.

## Lesson 6: Interpreters
I feel very lucky I studied this book because it introduces interpreters. By introducing interpreters, the authors teach a profound idea, namely, the theory of computation. Alan Turing introduced the idea of the universal computing machine - a Turing machine that uses what we now call programs stored in its memory can compute everything that is in principle computable. The SICP authors explain that programs can be thought of as descriptions of an abstract machine. Similarly, an interpreter can be seen as a machine that emulates other machines; that is, if an interpreter takes in a description of a factorial machine the interpreter will be able to compute factorials. So, since interpreters mimic other machines, interpreters are universal machines. Consequently, interpreters can mimic other interpreter machines. Now, this I find fascinating. This means that if your interpreter machine – written in Lisp – takes an interpreter for C machines it will mimic this C evaluator which in turn will mimic any other program described as a C machine. So, what is in principle computed is independent of the programming language. So, this is why I think learning Scheme and “Structure and Interpretation of Computer Programs” is a great thing to do. By studying this book one learns about profound ideas namely interpreters which capture the essence of computation. I think the theory of computation and its relationship with interpreters is a really deep connection. The above should make you marvel at the expressive power of Scheme. Despite how small the language is, one is able to express the notion of universal computation. In about 500 lines of code, you can build a Scheme interpreter in Scheme that captures this beautiful and profound idea. 
