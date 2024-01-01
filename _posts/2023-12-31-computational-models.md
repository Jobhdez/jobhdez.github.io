---
layout: post
title: The computational model behind interpreters
author: Job Hernandez Lara
tags: lisp compilers computer-science
---

### Introduction

In “Structure and Interpretation of Computer Programs” it talks about the deep meaning of what an interpreter is. An interpreter can be seen as a machine that emulates other machines. 

Interpreters compute anything that in principle can be computed, i.e. an interpreter is a universal machine; for example, if you feed a C interpreter to a Scheme interpreter the Scheme interpreter can emulate the C interpreter and thereby compute any C program.

> The deep idea here is that any evaluator can emulate any other. Thus, the notion of “what can in principle be computed” (ignoring practicalities of time and memory required) is independent of the language or the computer and instead reflects an underlying notion of computability.

To illustrate what this quote is saying, suppose we give the python3 interpreter a Scheme interpreter. As a consequence, the python3 interpreter will mimic a scheme interpreter which in turn will compute any scheme expression.

In what follows I will try to explain the environment model of computation that is the ground for interpreters. And we will conclude with a Scheme interpreter implemented in Python.
### Models of computation

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

In the environment model of computation a variable is not just a name for a value; instead, a variable implies a container. This container is called an **environment**. In this model, to evaluate a program you must also evaluate the operator and operand but you need to include an environment. When an assignment is being evaluated the interpreter must look in an environment to look up the value for the variable. 

To evaluate procedures we must evaluate it in the context of the global environment and also within its local environment.

So, my point is that the deep computational idea behind interpreters is grounded on the environment model of computation. The environment model of computation is indeed the model that is used to implement interpreters. 

### A Scheme interpreter implemented in Python

As an example take a look at the following [Scheme interpreter](https://github.com/Jobhdez/schemy) I built with Python. This example shows how a Python interpreter can emulate a Scheme interpreter which can compute any Scheme expression.

```python
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

In conclusion, I have tried to illustrate two models of computation, one of which, the evaluation model, is the ground for the deep idea about computation underlying interpreters.

