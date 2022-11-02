---
layout: post
title: "A Linear Algebra Compiler"
author: Job Hernandez
---
### Intro

Compilers are fun to build, so you should try to build one. Lately, I have been working on a linear algebra compiler[1]; it currently compiles a
simple linear algebra language called `Yotta` to Common Lisp and C. I would like to explain how this compiler works so you can use it as a model for
understanding way more advanced compilers.

### Architecture

The architecture of a compiler is comprised of a frontend, optimizer, and backend. Frontend -> Optimizer -> Backend. For now, we will skip the 
optimizer part since I have not implemented this component.

The `frontend` component takes the source program and tokenizes it; so in my compiler `lex-line` tokenizes the source program. For example, if you load my 
system and call `(lex-line "[3 4 5] + [4 5 6]")` (i.e., vector + vector) then the following tokens will be generated:

```
((:LEFT-BRACKET) (:NUMBER . 4) (:NUMBER . 5) (:NUMBER . 6) (:RIGHT-BRACKET)
 (:PLUS . :+) (:LEFT-BRACKET) (:NUMBER . 5) (:NUMBER . 6) (:NUMBER . 7)
 (:RIGHT-BRACKET))

```

Given the tokens then the `parse tree` will be generated; the parse tree is a tree (hence the name) so it is comprised of nodes reflecting valid expressions which
specified by the grammar. An example of how a grammar is defined is the following:

```
exp ::= scalar | vector | matrix | exp + exp 
     |  exp - exp| exp * exp | var | var = exp
     
scalar ::= int

vector ::= [int]

matrix::= [[int]]
```

All this is saying is that an `exp` is either a scalar, vector, matrix, or  a vector + vector operation, etc. The parser knows about this grammar and constructs
the parse tree from tokens. The parse tree of the corresponding tokens above is the following:

`(parse-with-lexer (token-generator tokens) **linear-algebra-grammar**)` ->
```
#S(PROGRAM
   :EXPRESSIONS #S(PLUS
                   :LEFT-EXP #S(VEC
                                :ENTRIES (#S(NUM :NUM 4) #S(NUM :NUM 5)
                                          #S(NUM :NUM 6)))
                   :RIGHT-EXP #S(VEC
                                 :ENTRIES (#S(NUM :NUM 5) #S(NUM :NUM 6)
                                           #S(NUM :NUM 7)))))


```

Each node is a Common Lisp structure, defined with `defstruct`.

From here you need to generate the `intermediate languages` which close the gap between your source language and the target language. A compiler could have multiple
intermediate languages. Intermediate Languages are implemented with Abstract Syntax trees or with a Flow Graph; in my compiler I used Abstract Syntax trees
to implement the intermediate languages. Since my compiler is simple the intermediate language is simple as well; given the parse tree above the following
intermediate abstract syntax tree is generated for the Lisp backend:

`(make-lisp-interlan parse-tree)` ->

```
#S(VECTORSUM
    :I YOTTA-VAR::I
    :N 3
    :EXP #S(SETFLISP
            :VAR #S(AREFLISP :ARRAY YOTTA-VAR::NEWARRAY :I YOTTA-VAR::I)
            :EXP #S(SUMLISP
                    :LEFTEXP #S(AREFLISP
                                :ARRAY #S(VECTORLISP
                                          :DIMENSION 3
                                          :ELEMENTS (#S(NUMLISP :N 4)
                                                     #S(NUMLISP :N 5)
                                                     #S(NUMLISP :N 6)))
                                :I YOTTA-VAR::I)
                    :RIGHTEXP #S(AREFLISP
                                 :ARRAY #S(VECTORLISP
                                           :DIMENSION 3
                                           :ELEMENTS (#S(NUMLISP :N 5)
                                                      #S(NUMLISP :N 6)
                                                      #S(NUMLISP :N 7)))
                                 :I YOTTA-VAR::I)))
    :LEFTEXP #S(VECTORLISP
                :DIMENSION 3
                :ELEMENTS (#S(NUMLISP :N 4) #S(NUMLISP :N 5) #S(NUMLISP :N 6)))
    :RIGHTEXP #S(VECTORLISP
                 :DIMENSION 3
                 :ELEMENTS (#S(NUMLISP :N 5) #S(NUMLISP :N 6)
                            #S(NUMLISP :N 7))))

```

And finally the `backend` takes this intermediate language and generates the Lisp code:

`(compile-lisp lisp-interlan)` ->


```
(PROGN
 (SETQ NEWA (MAKE-ARRAY 3))
 (DOTIMES (I 3)
   (SETF (AREF NEWA I)
           (+ (AREF (MAKE-ARRAY 3 (4 5 6)) I)
              (AREF (MAKE-ARRAY 3 (5 6 7)) I))))))
 ```


 
 There you have it -- a brief and simple explanation of how a simple compiler works. Production compilers are far more complex but they follow the same 
 patterns. Obviously, `Yotta` is a simplified version of the production compilers but I hope this helps you start.
 
 One way to improve this compiler is to implement an optimizer. Some optimizations include `loop unrolling` and `loop fusion`, `SIMD vectorization`. 
 Another idea to to generate efficient assembly.
 
 
**References**
 
 [1] [Yotta](https://github.com/Jobhdez/Yotta)
