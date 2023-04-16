---
layout: post
title: "SICP's important computing and programming lessons"
author: Job Hernandez
---
### Personal experience with SICP
The book “Structure and Interpretation of Computer Programs” was one of my favorite experiences I have had in the past 10 years. My copy of SICP is also very special because my mom gave it to me. The way it was written and how it uses mathematics made a huge impression on me. I got my mind blown a few times and I cried of joy while reading a few parts. It was such an amazing experience. SICP also made me love mathematics more and it helped me realize how deep computer science is. The way the authors of SICP described higher order procedures – e.g., how higher order procedures elevate the conceptual level of a given program – was a tremendously meaningful experience for me. So, in other words Scheme and SICP helped me appreciate computer science. In addition, my SICP studies helped me understand that interpreters/DSLs and compilers are the ultimate abstraction which is key for controlling complexity.

### SICP’s important lessons
#### Lesson 1: The Scheme Programming language
##### Scheme is a model of computation
I think the Scheme programming language is such a profound language; Scheme is an extension of the lambda calculus which essentially means that Scheme can be seen as a model of computation. The lambda calculus models all non imperative constructs of programming languages as applications of functions and specifies with axioms the semantics of these expressions. I think the key idea here is the substitution model of computation. Consider the following:
```Scheme
(define (multiply-squares n n2) (* (square n) (square n2)))
(define (square n) (* n n))

;; substitution model

(multiply-squares 3 4)
;; evaluates to
(* (square 3) (square 4))
;; evaluates to
(*  (* 3 3) (* 4 4))
;; evaluates to
(* 9 16)
;; evaluates to
144
```
In the above example we evaluated the expression `(multiply-squares 3 4)` - then each expression is replaced by an expression that is equivalent via an axiom of the lambda calculus. Another example of the substitution model is the the evaluation of a recursive function that generates a recursive process whereby a chain of expressions is built up and whereby the chain of expressions then contracts as expressions get evaluated. Here is the Scheme example:
```Scheme
(define (factorial n)
   (if (= n 0)
   	1
   	(* n (factorial (- n 1)))))
```
The process that factorial generates is the following:
```Scheme
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

##### Tail Recursion
One concept I learned from reading this book that not many programmers realize is tail recursion. If a language does not support tail recursion each recursive call has a different frame in the stack. The stack is memory that has a frame for each function call. Each frame consists of the function's return address, and addresses for variables. In contrast, in tail recursive language, the recursive call is not associated with its own frame. It only has to keep track of the state variables, i.e., the addresses of the variables. As a consequece, it is possible to write a recursive procedure that generates an iterative process. In most programming languages a recursive procedure generates a recursive process whose time complexity is O(n) and whose space complexity is O(n); an example of a recursive process is the `factorial` process above; but in Scheme, a recursive procedure may generate an iterative process – i.e., a process whose time complexity is also O(n) but whose space complexity is O(1). The real insight here is that for a given procedure that generates a recursive process one can also write a recursive procedure that generates an iterative process. In Scheme there are no looping constructs, for instance. Here is an example a recursive procedure that generates an iterative process (taken from SICP):
```Scheme
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

```Scheme
(factorial 6)
(factorial-iter 1 1 6)
(factorial-iter 1 2 6)
(factorial-iter 2 3 6)
(factorial-iter 6 4 6)
(factorial-iter 24 5 6)
(factorial-iter 120 6 6)
(factorial-iter 720 7 6)
```
#### Lesson 2: Functional Programming
One of the important lessons in computing and programming that you will learn from SICP is functional programming. Functional programming is functional in the mathematical sense; that is, an input to a function has exactly only one output. The programs in the first two chapters in the book are functional; moreover, functional programming is a type of programming that satisfies the substitution model of computation. Programs in functional style satisfy the substitution model because functional programs do not entail mutating variables. Variables in functional programming are definitions. Consider the following quote frome SICP:

>The trouble here is that substitution is based ultimately on the notion that the symbols in our language are essentially names for values. But as soon as we introduce set! and the idea that the value of a variable can change, a variable can no longer be simply a name.

Here are some examples  from SICP illustrating this idea. Suppose you have a function`make-simplified-withdraw` defined as:

```Scheme
(define (make-simplifed-withdraw balance)
  (lambda (amount)
    (set! balance (- balance amount))
    balance))
```

What happens when you apply the substitution model to this function?

```
((make-simplified-withdraw 25) 20)
;; evalutes to
((lambda (amount) (set! balance (- 25 amount)) 25) 20)
;; evaluates to
(set! balance (- 25 20)) 25
;; evaluates to
25
```
The above example sets balance to 5 and returns 25 which is the wrong answer; now compare this with a functional version that adheres to the substitution model:

```Scheme
(define (make-decrementer balance)
  (lambda (amount)
    (- balance amount)))
    
(define D (make-decrementer 25))

(D 20)
;; evaluates to
((make-decrementer 25) 20)
;; evaluates to
((lambda (amount) (- 25 amount)) 20)
;; evaluates to
(- 25 20)
;; evaluates to
5
```

This brings to another core concept in functional programming, namely, `referentially transparency`. Referentially transperency dictates that given two expressions these two expressions have the same computational behavior such that they can be substituted for one another; for example:

```Scheme
(define D1 (make-decrementer 25))
(define D2 (make-decrementer 25))
```

In the above example `D1` and `D2` behave the same because `D1` can be substituted for `D2` in any computation without changing its result but consider the function `make-simplified-withdraw`:

```Scheme
(define W1 (make-simplified-withdraw 25))
(define W2 (make-simplified-widthdraw 25))

(W1 20)
;; ->
5

(W1 20)
;; -> 
-15

(W2 20)
;; -> 
5
```

`W1` cannot be substituted for `W2` in any computation because `W1` is not a function in the mathematical sense -- i.e., as you can see in the example the input `20` corresponds to two different ouputs.

Another core concept in functional programming that SICP covers is `lexical scoping`; lexical scoping dicates that the body of a given function can refer to variables whose binding site is outside of the function as illustrated in the following example:

```Scheme
(define (sum a b x)
  (define (square-plus)
     (+ (* x x) a b))
     
  (square-plus))
```

Here, the body of `square-plus` refers to `x`, `a`, and `b` whose binding site is outside of the function and as a result you do not have to write:

```Scheme
(define (square-plus a b x) ...)
```
     
    


#### Lesson 3: Higher Order Procedures
Higher order procedures are procedures that consume other procedures or return other procedures. Higher order procedures increase the conceptual level of the given program; instead of thinking in terms of particular sums, higher order procedures allow you to abstract the summation pattern and apply this pattern to instances of summations. Modularity is the consequence of this because instead of repeating code with multiple concrete summations you get to have an abstraction applied multiple times. Consider the following to summations:

```Scheme
(define (sum-integers a b)
  (if (> a b)
  	0
  	(+ a (sum-integers (+ a 1) b))))

(define (sum-cubes a b)
  (if (> a b)
  	0
  	(+ (cube a) (sum-cubes (+ a 1) b))))
```

Here you have two instances of a pattern. With a higher order procedure you can do this instead:

```Scheme

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
#### Lesson 4: Data Abstraction
Data abstraction is a way to form compound data; data abstraction also increases modularity because data abstraction allows you to separate the use from the implementation. Here is how you would implement data abstraction in Scheme:

```Scheme
;; data abstraction for fractions
(define (make-fraction numer denom) (list numer denom))  ;; constructor
(define (numerator fraction) (car fraction))         ;; selector
(define (denominator fraction) (car (cdr fraction))) ;; selector
```
If you now use the constructor and the selectors to write your program then you now have the flexibility to change the representation of your data. If on the other hand you did not use data abstraction if you were to change the representation you would have to change more code.

In an object oriented language you would implement data abstraction with classes and encapsulation. In Python you would do something like this by the way:

```Python
class Fraction:
    def __init__(self, numer, denom):
        self.__numer = numer
        self.__denom = denom
        
    def get_numer(self):
        return self.__numer
        
    def set_numer(self, numer):
        self.__numer = numer
        
    def get_denom(self):
        return self.__denom
        
    def set_denom(self, denom):
        self.__denom = denom
     
     ....
```
#### Lesson 5: Data Directed Programming
Data directed programming is a technique that allows multiple programmers to work on different parts of a system at the same time; moreover, it is a type of additive programming where instead of making changes to add new functionality you instead add a module. For instance, suppose you are building a computer algebra system and you are implementing integers, fractions, polynomials, and matrices. If you were to use data directed programming you would define a package for each:

```Scheme
(define fraction-table (make-hash))
(define polynomial-table (make-hash))
;; more hash tables for the rest of the math objects

(define (put fn fn-name table)
  (hash-set! table fn-name fn))
;;....

(define (fraction)
  (define (addition frac1 frac2)
      ...)
      
   ;; more operations
   
   (put addtion 'addition 'fraction))
   
(define (polynomial)
  (define (addition p1 p2)
    ....)
    
   ;; more operations
   
   (put addition 'addition 'polynomial))

```

Once you have the above you can define generic functions:

```Scheme
(define (addition n b)
  (cond ((fraction? n) 
         (apply (hash-ref 'fraction 'addition) n b))
        ((polynomial? n) ...)))
 ```
 
The point of this is to allow multiple programmers to work on the given system without introducing name conflicts and in a modular fashion. You do not have to use this method exactly but the lesson here is that you need to program using additive programming and use techniques that allow many programmers to work on different parts of the system at the same time without introducing conflicts such as name conflicts; in the example above there could be many functions named `addition` but there will not be any conflicts because each math object is represented as a package that can be developed in isolation.
 
#### Lesson 6: Interpreters
I feel very lucky I studied this book because it introduces interpreters. By introducing interpreters, the authors teach a profound idea, namely, the theory of computation. Alan Turing introduced the idea of the universal computing machine - a Turing machine that uses what we now call programs stored in its memory can compute everything that is in principle computable. The SICP authors explain that programs can be thought of as descriptions of an abstract machine. Similarly, an interpreter can be seen as a machine that emulates other machines; that is, if an interpreter takes in a description of a factorial machine the interpreter will be able to compute factorials. So, since interpreters mimic other machines, interpreters are universal machines. Consequently, interpreters can mimic other interpreter machines. Now, this I find fascinating. This means that if your interpreter machine – written in Lisp – takes an interpreter for C machines it will mimic this C evaluator which in turn will mimic any other program described as a C machine. So, what is in principle computed is independent of the programming language. So, this is why I think learning Scheme and “Structure and Interpretation of Computer Programs” is a great thing to do. By studying this book one learns about profound ideas namely interpreters which capture the essence of computation. I think the theory of computation and its relationship with interpreters is a really deep connection. The above should make you marvel at the expressive power of Scheme. Despite how small the language is, one is able to express the notion of universal computation. In about 500 lines of code, you can build a Scheme interpreter in Scheme that captures this beautiful and profound idea. 
