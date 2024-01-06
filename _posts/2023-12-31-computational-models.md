---
layout: post
title: The computational model behind interpreters
author: Job Hernandez Lara
tags: lisp compilers computer-science
---

* Table of Contents
{:toc}

### Introduction

In this blog post I will explain what the substitution model for computation is which is associated with pure functional programming and then I explain how, when assignment and mutation are added,  the substitution model no longer holds. As a result, a different model for computation is needed, namely, the environment model for computation whereby expressions get evaluated in the context of an environment. I culminate the blog by sharing part of the scheme interpreter I built which is an implementation of the environment model. CPython also uses this environment model to carry out its computations; moreover, I also point out that a scheme interpreter built using a Python interpreter shows what it means to compute something in the sense of a universal machine.

### Background
In “Structure and Interpretation of Computer Programs” it talks about the deep meaning of what an interpreter is. An interpreter can be seen as a universal machine that emulates other machines. 

Interpreters compute anything that in principle can be computed, i.e. an interpreter is a universal machine; for example, if you feed a C interpreter to a Scheme interpreter the Scheme interpreter can emulate the C interpreter and thereby compute any C program.

> The deep idea here is that any evaluator can emulate any other. Thus, the notion of “what can in principle be computed” (ignoring practicalities of time and memory required) is independent of the language or the computer and instead reflects an underlying notion of computability.

To illustrate what this quote is saying, suppose we give the Python3 interpreter a Scheme interpreter. As a consequence, the Python3 interpreter will mimic a Scheme interpreter which in turn will compute any Scheme expression. So, computation is universal.

In what follows I will try to explain the environment model for  computation. The environment model for computation is the ground for interpreters. And we will conclude with a Scheme interpreter implemented in Python.

### Models for computation

In “Structure and Interpretation of Computer Programs” it talks about at least two computational models; one is associated with pure functional programming and one with programming with assignment/mutation. These computational models are the **substitution model** and **environment model**.

Both models are about how expressions get evaluated.

In the substitution model, each element of an expression is another expression including the operator. So, the way the substitution model works is by evaluating each expression and then applying the operator to the operands.

Suppose you have the following Scheme program:


```scheme
(define (sum-of-squares e e2)
  (+ (square e) (square e2)))

(define (square e)
  (* e e))
```

To evaluate this you will go through the following process:

```scheme
(sum-of-squares 3 4)
;; ->
(+ (square 3) (square 4))
;; ->
(+ (* 3 3) (* 4 4))
;; ->
(+ 9 16)
;; ->
25
```

A property of such model is **referential transparency**. A program is said to be referentially transparent if one function definition can be substituted for another one and still evaluate to the same value.

Consider the following:

```scheme
(define p1 sum-of-squares)
(define p2 sum-of-squares)

(p1 3 4)
;; ->
25

(p2 3 4)
;; ->
25
```

In the above example, the function `p1` can be substituted with `p2` at any time and get the same result.

But what happens when you introduce mutation? If you introduce mutation does referential transparency still hold?

Consider the following two programs taken from SICP:


```scheme
(define (make-simplified-withdraw balance)
   (lambda (amount)
     (set! balance (- balance amount))))

(define (make-decrementer balance)
   (lambda (amount)
     (- balance amount)))

(define w1 (make-simplified-withdraw 25))
(define d1 (make-decrementer 25))

(w1 10)
;; ->
15
(w1 10)
;; ->
5

;; the following adheres to the substitution model
(d1 10)
;; ->
15
(d1 10)
;; ->
15
```
For the first example, since the substitution model does not hold anymore then we must talk about the **environment model** of computation.

In the environment model for computation a variable is not just a name for a value; instead, a variable implies a container. This container is called an **environment**. In this model, to evaluate a program you must also evaluate the operator and operand but you need to do this within an environment. When an assignment is being evaluated the interpreter must look up the value for the variable in an environment. 

To evaluate procedures we must evaluate it in the context of the global environment and also within its local environment.

So, my point is that the deep computational idea behind interpreters is grounded on the environment model of computation. The environment model of computation is indeed the model that is used to implement interpreters. 

#### Interpreters are implementation of the environment model for computation

A Scheme interpreter or a Python interpreter or any other interpreter in general are implementations of the environment model for computation.

Evaluating or interpreting  programs is a process which consists of evaluating an expression in the context of an environment.

For example, here is an example of the environment for my Scheme interpreter:

```python
class Env(dict):
    def __init__(self, params=(), args=(), outer=None):
        self.update(zip(params, args))
        self.outer = outer

    def find(self, var):
        if var in self:
            return self
        elif self.outer is not None:
            return self.outer.find(var)
        else:
            raise NameError(f"Variable '{var}' is not defined.")
  ```


And as I said above, when you introduce mutation and assignment, the substitution model no longer holds, and as a consequence, a variable is no longer just a definition. Instead a variable implies an environment. As you can see here:

```python 
def interp(exp, env=global_env):
    match exp:
        #....
	case SetBang(var, e):
            env.find(var.var)[var.var] = interp(e, env)
            return None
```

Procedures are evaluated within an environment but a procedure needs to consider the global environment and an environment that is local to it. An example of this is the following code of this implementation:

```python
class Procedure(object):
    def __init__(self, params, body, env):
        self.params, self.body, self.env = params, body, env

    def __call__(self, *args):
        return interp(self.body, Env(self.params, args, self.env))
```

##### A Scheme interpreter implementation 

As an example take a look at the following [Scheme interpreter](https://github.com/Jobhdez/schemy) I built with Python. This example shows how a Python interpreter can emulate a Scheme interpreter which can compute any Scheme expression.

```python
class Env(dict):
    def __init__(self, params=(), args=(), outer=None):
        self.update(zip(params, args))
        self.outer = outer

    def find(self, var):
        if var in self:
            return self
        elif self.outer is not None:
            return self.outer.find(var)
        else:
            raise NameError(f"Variable '{var}' is not defined.")
    
class Procedure(object):
    def __init__(self, params, body, env):
        self.params, self.body, self.env = params, body, env

    def __call__(self, *args):
        return interp(self.body, Env(self.params, args, self.env))
	
def interp(exp, env=global_env):

    match exp:
        case Exps(exps):
            result = None
            for exp in exps:
                result = interp(exp, env)
            return result
        
        case Exp(e):
            return interp(e, env)
        
        case Bool(b):
            return b
        
        case If(cnd, thn, els):
            match interp(cnd, env):
                case "#t":
                    return interp(thn, env)
                case "#f":
                    return interp(els, env)
                
        case Prim(Op(oper), e, e2):
            match oper:
                case 'and':
                    match interp(e, env):
                        case '#t':
                            match interp(e2, env):
                                case '#t':
                                    return '#t'
                                case '#f':
                                    return '#f'
                        case '#f':
                            return '#f'
                        
                case 'or':
                    match interp(e, env):
                        case '#t':
                            return '#t'
                        case '#f':
                            match interp(e2, env):
                                case '#t':
                                    return '#t'
                                case '#f':
                                    return '#f'
                                
                case '+':
                    return interp(e, env) + interp(e2, env)
                
                case '-':
                    return interp(e, env) - interp(e2, env)

                case '*':
                    return interp(e, env) * interp(e2, env)

                case '=':
                    return '#t' if interp(e, env) == interp(e2, env) else '#f'
        case Int(n):
            return n
        
        case Var(e):
            return env.find(e)[e]
        
        case Let(Binding(Var(var), e), body_exp):
            proc = Procedure([var], body_exp, env)
            exps = [e]
            vals = [interp(e2, env) for e2 in exps]

            return proc(*vals)
        
        case SetBang(var, e):
            env.find(var.var)[var.var] = interp(e, env)
            return None
            
        case Begin(exps):
            flat_expressions = flatten_exps(exps)
            expressions = flat_expressions[:-1]
            for exp in expressions:
                interp(exp, env)

            return interp(flat_expressions[-1], env)

        case Define(Var(var), exp):
            env[var] = interp(exp, env)

        case Lambda(params, body):
            parameters = flatten_params(params)

            return Procedure(parameters, body, env)

        case Application(exps):
            exps = flatten_exps(exps)
            operator = interp(exps[0], env)
            exps = exps[1:]
            vals = [interp(e, env) for e in exps]

            return operator(*vals)
            
        case _:
            raise ValueError(f'Parse node {exp} is not valid node.')

```

### Conclusion

In conclusion, I have tried to illustrate two models for computation, one of which, the evaluation model, is the ground for the deep idea about computation underlying interpreters. I hope this was helpful. Thanks.

