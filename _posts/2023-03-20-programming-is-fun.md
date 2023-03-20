---
layout: post
title: "Programming is fun"
author: Job Hernandez
---

Programming is fun! I love to program; whether I am working on web apps, math software or compilers I get a kick out of solving problems; moreover, I enjoy programming because it allows me to focus on a problem and think. 

# Noticing Patterns
Programming is fun because sometimes you solve problems by noticing patterns and in turn you learn an insight about the problem domain; for example, a few weeks ago I was writing a Scheme compiler in Python and I was trying to compile `let` expressions into assembly (x86). To notice a pattern you start by writing down the different cases of valid `let` expressions. Consider the following examples:

```
Example 1:
       (let ((x.1 2)) (+ (let ((x.2 5)) x.2) x.1))
       ->
       start:
         x.1 = 2
         x.2 = 5
         return x.1 + x.2
    Example 2:
        (let ((x.1 1)) (+ (let ((x.2 4)) (+ (let ((x.3 5)) x.3) x.2) x.1))
        ->
        start:
           x.1 = 1
           x.2 = 4
           x.3 = 5
           return x.1 + x.2 + x.4
```

Firstly, by writing down examples you get to understand the problem; that is, you know several input-output pairs so at this point you can focus on how to get from the input to the output. It is here when you get to look for patterns. Notice in my example that `Example 1` and Example 2` are instances of the same rule but `Example 2` is more nested than `Example 1`. By thinking about these examples I was able to notice a recursive pattern so all I had to do is to pattern match on this particular case. The Python code for `Example 1` and `Example 2` looks something like this:

```Python
def explicate_control(ast, counter, vars, assignments):
    match ast:
    	    case x if isinstance(x, Let):
        	    if isinstance(x.bindings, Binding):
            	    bindings = x.bindings.bindings
            	    var, exp = bindings
            	    vars[counter] = var
            	    assignments[counter] = Assign(var, exp)
            	    if isinstance(x.body, List) and x.body.expressions[0].atom == '+' and isinstance(x.body.expressions[1], Let):
                	        counter += 1
                	        explicate_control(x.body.expressions[1], counter, vars, assignments)
                	        creturn = []
                     	  creturn.append(CReturn(Prim(Atom('+'), list(vars.values()))))
                     	  return CProgram(list(assignments.values()) + creturn)
```


I am not claiming that this is the best solution to the problem or that my code is perfect but by thinking of different cases of the same instance I was able to notice a recursive pattern and all I had to do was to think and implement the pattern. In response to implementing the solution I got tremendously happy and excited. I felt like I captured the solution very clearly. The process I just mentioned above is the reason why I love programming so much; it is a process of thinking and noticing patterns and after you get the program to work it is a source of great rejoice.

# Fundamental Software Systems that keep Society going
In addition, programming is fun and interesting because you get to marvel at the fundamental software systems that enable the computing industry. Computers are embedded in our lives, whether it is the phone in your pocket or your laptop. Computers were enabled by `compilers` and `operating systems`. I think after I learned about these systems and in some cases building simplified versions of them has been a wonderful experience. I remember marveling at the fact that people can build such fundamental and complex systems. Compilers and Operating systems are millions of lines of code but the thing I marvel at is the genius of past computer scientists who invented all this. It is amazing.

Personally, I have more experience with compilers. Compilers translate a programming language A to another programming language B. Often enough a compiler translates a high level language such as C++ to the bits that the CPU can understand and there is something magical and amazing about this. Compilers are fundamental to computing; for example, consider LLVM - a compiler infrastructure. LLVM is used at places like Apple, Intel, Google, and Nvidia. If you are using an iPhone then LLVM was used to compile every compiled application in this device. Or given that Intel uses LLVM, think about the impact LLVM has: many computers have Intel CPUs, or consider the Intel CPUs used at the data centers that power the internet and web. Nvidia uses LLVM for their CUDA compiler. Think about this - deep learning systems that personalize your YouTube content and your netflix recommendations, etc, are in essence enabled due to the LLVM, a *compiler* infrastructure. This is the type of impact fundamental systems such as compilers have.

Above I have explained how impactful compilers are. I explained how important LLVM is for the computing infrastructure. But given that LLVM is open source, you can become a regular contributor thereby helping keep the computing infrastructure going which would be a lot of fun.
