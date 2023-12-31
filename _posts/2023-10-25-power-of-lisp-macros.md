---
layout: post
title: Power of Lisp macros
author: Job Hernandez Lara
tags: lisp
---

*version*: 1.0, initial version

### Introduction

I find it hard to admit but I spent on average around 2 hours a day programming in Lisp for about 3 years and never touched macros. But fortunately by contributing to open source and seeing how really good programmers program I was able to pick up some macro skills.

As a result, I have more appreciation for Lisp and kind of see how in other languages you end up repeating yourself due to lack of syntactic abstraction; moreover, I have seen how macros enable you to build languages and extend the Lisp language itself.

This is all important because, as the SICP authors claimed, software engineering is about controlling complexity. Using macros allows you to control complexity because by using macros your systems are easier to maintain and by using macros for building DSLs and extending Lisp itself you end up increasing the expressiveness of Lisp.

In what follows I will explain how macros are indeed useful and lead to better systems. 

Definitely, I am not an expert but these are my observations from contributing to Coalton and conversing with the creator of Coalton.

### Macros allow you to get more done with less

One of the reasons why Lisp macros are powerful is because macros enable syntactic abstraction. Syntactic abstraction leads to shorter code and more modular code and  moreover it leads to code without repeating patterns

Consider building a vector library. Suppose you are trying to implement different types such as single float and double float and so on. 

In Common Lisp you can write the pattern of addition once and I apply it $$ n $$ times where $$ N $$ is the number of types you are implementing.

Here is how I would write it:

```lisp
(defmacro addition-handler (type fn-name)
  `(defun ,fn-name (vec vec2)
 	(progn (setq vec3 (make-array ,(length vec) :element-type ',type))
    	(dotimes (i ,(length vec))
      	(setf (aref vec3 i)
   	 	(+ (aref vec i)
   	    	    (aref vec2 i)))))))

(addition-handler single-float addition-vector-single-float)
(addition-handler double-float addition-vector-double-float)
```

You can continue this pattern and implement more vector operations and more types.

So, in Lisp you can get more done with less code. In turn, this leads to smaller systems due to the abstraction and since smaller systems are easier to make sense of the ease of maintenance increases.

### Macros allow you to build DSLs

In SICP you learn that DSLs are the ultimate abstraction. Software engineering is about controlling complexity. As we have seen previously macros allow you to control complexity because macros lead to more abstracted systems.

It turns out that the power of macros is about building languages. As Robert, the creator of Coalton and an state of the art quantum computing compiler Quilc said:

> The real power of macros is when they're in the hands of compiler writers and language designers.

The idea of building DSLs with macros is closely tied to the idea that software systems should be evolvable and adaptable; for example, in the book “Software Design for Flexibility”, Sussman and Hanson claimed that systems need to be evolvable. According to them, systems should be designed in such a way that they adapt and grow as requirements change.

> The best systems are evolvable: they can be adapted to new situations with only minor modifications.

When adding a new feature one should be able to add to the system instead of modifying the codebase.

Well Lisp can be seen as an evolvable system. With macros you can adapt the language to new situations and can extend the language with new constructs without modifying the Lisp compiler.

You can see Lisp as an assembly of tiny domain specific languages. 

Lisp has a tiny core and the rest of the language can be built by writing macros. As Alan Kay said:

> Yes, that was the big revelation to me when I was in graduate school—when I finally understood that the half page of code on the bottom of page 13 of the Lisp 1.5 manual was Lisp in itself. These were “Maxwell’s Equations of Software!” This is the whole world of programming in a few lines that I can put my hand over. 

So, as I said Lisp is an evolvable system because it has macros; for example, suppose your favorite language doesn’t have list comprehensions. How would you add this construct to your language? The only way you can do this is by modifying the compiler. As a result, your favorite language is not as evolvable as it could be.

In a conversation I had on Twitter with Robert Smith, he demonstrated the power of macros by showing me how a language for list comprehensions could be implemented.

What follows is a summary of his message.

Suppose you want to express this in your language: 

```
[(x, y) for x in range(5) for y = 2*x when isEven(x)] 
```

How would you add list comprehensions to your favorite language? You cannot.

But in Lisp you can write a macro that allows you to express this.

For example, a macro for list comprehensions is a tiny DSL.

You might say that Python can be used as this because of macros but as Robert Smith says:

> (1) Python macros don't really exist as a defined part of the language, (2) python has no concept of "macroexpansion time", (3) being able to manipulate ASTs and source code is not the same as having a macro facility built in to the language, even if you can accomplish similar tasks in theory. I would say that python allows metaprogramming (in a very bizarre and difficult way), but NOT that it supports macros. The  language is definitely not homoiconic, which is a key ingredient in making macros facile.

I was able to see the power of macros thanks to Robert Smith. He explained this to me by the following implementation of the the loop comprehension DSL:

```lisp
;;; A list comprehension will be defined as:
;;;
;;;    (comp &lt;expr&gt; &lt;clause&gt;*)
;;;
;;; where &lt;expr&gt; is any Lisp expression, and &lt;clause&gt; can be:
;;;
;;;    &lt;clause&gt; := (for &lt;var&gt; in &lt;expr&gt;)
;;;              | (for &lt;var&gt; = &lt;expr&gt;)
;;;              | (for &lt;var&gt; from &lt;expr&gt; to &lt;expr&gt;)
;;;              | (when &lt;expr&gt;)
;;;
;;; For example,
;;;
;;;    (comp (list x y) (for x from 1 to 100) (for y = (* x x)) (when (evenp y)))
;;;
;;; would generate a list of numbers and their squares, only when the
;;; square is even.

CL-USER> (compile-comp '(comp (list x y) (for x from 1 to 100) (for y = (* x x)) (when (evenp y))))

 (LET ((#:RESULT1639 NIL)) (LOOP :FOR X :FROM 1 :TO 100 :DO (LET ((Y (* X X))) (WHEN (EVENP Y) (PUSH (LIST X Y) #:RESULT1639)))) (NREVERSE #:RESULT1639))

CL-USER> (compile-comp '(comp x (for x from 1 to 10) (for y from 1 to 10)))

(LET ((#:RESULT1640 NIL))
  (LOOP :FOR X :FROM 1 :TO 10
    	:DO (LOOP :FOR Y :FROM 1 :TO 10
              	:DO (PUSH X #:RESULT1640)))
  (NREVERSE #:RESULT1640))


(defun for-in-clause? (form)
  (and (= 4 (length form))
   	(string= "FOR" (first form))
   	(string= "IN" (third form))))

(defun for-=-clause? (form)
  (and (= 4 (length form))
   	(string= "FOR" (first form))
   	(string= "=" (third form))))

(defun for-range-clause? (form)
  (and (= 6 (length form))
   	(string= "FOR" (first form))
   	(string= "FROM" (third form))
   	(string= "TO" (fifth form))))

(defun when-clause? (form)
  (and (= 2 (length form))
   	(string= "WHEN" (first form))))

(defun list-comprehension-form? (form)
  (and (= 2 (length form))
   	(string= "COMP" (first form))))


(defun compile-clause (build-expr clause)
  (cond
	((for-in-clause? clause)
 	`(loop :for ,(second clause) :in ,(fourth clause)
        	:do ,build-expr))
	((for-=-clause? clause)
 	`(let ((,(second clause) ,(fourth clause)))
    	,build-expr))
	((for-range-clause? clause)
 	`(loop :for ,(second clause) :from ,(fourth clause) :to ,(sixth clause)
        	:do ,build-expr))
	((when-clause? clause)
 	`(when ,(second clause)
    	,build-expr))
	(t
 	(error "Invalid clause: ~S" clause))))

(defun compile-clauses (build-expr clauses)
  (if (null clauses)
  	build-expr
  	(compile-clauses
   	(compile-clause build-expr (first clauses))
   	(rest clauses))))

(defun compile-comp (form)
  (assert (list-comprehension-form? form))
  (let* ((clauses	(nthcdr 2 form))
     	(result 	(gensym "RESULT"))
     	(build-expr `(push ,(second form) ,result)))
	`(let ((,result nil))
   	,(compile-clauses build-expr (reverse clauses))
   	(nreverse ,result))))
```

Here is the interaction in the REPL:

```lisp
* (defmacro comp (expr &rest clauses) (compile-comp `(comp ,expr ,@clauses))) 

* (comp (* x x) (for x from 1 to 10)) 
  (1 4 9 16 25 36 49 64 81 100) 

* (comp (* x x) (for y from 1 to 10) (for x in (comp (sin y) (when (evenp y))))) 
(0.82682174 0.57275003 0.07807302 0.97882974 0.295959) 

* (comp (list x y) (for x from 1 to 10) (for y from 1 to 10) (when (< x y))) 
((1 2) (1 3) (1 4) (1 5) (1 6) (1 7) (1 8) (1 9) (1 10) (2 3) (2 4) (2 5) (2 6) (2 7) (2 8) (2 9) (2 10) (3 4) (3 5) (3 6) (3 7) (3 8) (3 9) (3 10) (4 5) (4 6) (4 7) (4 8) (4 9) (4 10) (5 6) (5 7) (5 8) (5 9) (5 10) (6 7) (6 8) (6 9) (6 10) (7 8) (7 9) (7 10) (8 9) (8 10) (9 10))
```

As you can see you can extend Lisp, which does not support list comprehensions, with the ability to express comprehensions. This is indeed the power of macros.

### Conclusion

So, there you go. Based on my observations and conversations I have had I have tried to show the power of macros. Firstly, I showed how you can do more with less code by using macros which in turn leads to systems without repeating patterns. If you consider that Lisp also has higher order functions you can see how Lisp is a great language to build modular code. In addition, I shared how macros can help the programmer extend the Lisp language itself which is the ultimate way to conquer complexity.


