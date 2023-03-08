---
layout: post
title: "Thoughts about computation"
author: Job Hernandez
---

### Intro
The theory of computation is a tremendously fascinating and deep theory. I have barely scratched  the surface but the theory of computation covers a lot of ground - from the Scheme programming language and interpreters to the human mind. What follows next are some thoughts I have about these ideas.

### Scheme as an extension of the lambda calculus

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

In the above example we evaluated the expression `(multiply-squares 3 4)` - then each expression is replaced by an expression that is equivalent via an axiom of the lambda calculus. Another example of the substitution model is the the evaluation of a recursive function that generates a recursive process whereby a chain of expressions is built up and whereby the chain of expressions then contracts as expressions get evaluated. Here is the Scheme example:

```
(define (factorial n) 
   (if (= n 0)
       1
       (* n (factorial (- n 1)))))
```

The process that `factorial` generates is the following:

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



### Computation and Interpreters: Scheme is wonderful
Alan Turing introduced the idea of the universal computing machine - a Turing machine that uses what we now call `programs` stored in its memory can compute everything that is in principle computable. One of the ideas you learn in SICP is a very profound one, namely, programs can be thought of as descriptions of an abstract machine. Similarly, an interpreter can be seen as a machine that emulates other machines; that is, if an interpreter takes in a description of a factorial machine the interpreter will be able to compute factorials. So, since interpreters mimic other machines, interpreters are universal machines. Consequently, interpreters can mimic other interpreter machines. Now, this I find fascinating. This means that if your interpreter machine – written in Lisp – takes an interpreter for C machines it will mimic this C evaluator which in turn will mimic any other program described as a C machine. So, what is in principle computed is independent of the programming language.
So, this is why I think learning Scheme and “Structure and Interpretation of Computer Programs” is a great thing to do. By studying this book one learns about profound ideas namely interpreters which capture the essence of computation. I think the theory of computation and its relationship with interpreters is a really deep connection. Consequently, I marvel at the power of the Scheme programming language; that is, despite how small the language is, one is able to express the notion of universal computation. In less than 500 lines of code, you can build a Scheme interpreter in Scheme that captures this beautiful and profound idea. But you can also build a Scheme interpreter in C or Java that captures the same idea but there is something about functional programming that I find beautiful. It is amazing that the human mind has the power to devise such amazing theoretical constructs.

### Is the mind a computer?
As much as I find the theory of computation fascinating, one has to ask whether the human mind is a computer. I think what Godel said was that a Turing machine can be thought of as a formal system. Godel defines a formal system as:
>“due to A. M. Turing’s work, [that] a precise and unquestionably adequate definition of the general concept of formal system can now be given ... [A] formal system can simply be defined to be any mechanical procedure for producing formulas, called provable formulas.”  

One of the things Godel proved was that in a consistent formal system there will exist a true statement that can't be proven. And his other theorem dictates that you can't prove the consistency of a formal system S from within S. Here, consistency just means that, for example  --  you can't both hold K and its contradiction at the same time. So, Godel claimed that turing machines are these such systems. But, Godel believes, the human mind isn't a formal system because it can determine things that a formal system cant determine about itself. Consider Godel’s quote:
>“My incompleteness theorem makes it likely that mind is not mechanical, or else mind cannot understand its own mechanism. If my result is taken together with the rationalistic attitude which Hilbert had and which was not refuted by my results, then [we can infer] the sharp result that mind is not mechanical. This is so, because, if the mind were a machine, there would, contrary to this rationalistic attitude, exist number-theoretic questions undecidable for the human mind.”

Penrose has argued that given Godel’s theorem, consciousness is not a computation. If this is the case the aim of AI people of creating machines as intelligent as humans will fail.   	               	 



