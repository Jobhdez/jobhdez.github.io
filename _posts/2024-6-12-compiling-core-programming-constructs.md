---
layout: post
title: "Compiling core programming constructs"
author: Job Hernandez Lara
tags: compilers internals
---

I would like to share a bit about compiling core programming constructs; recently, I implemented a compiler in Haskell that lowered core constructs such as tuples, if expressions, loops, comparison operators (i.e., <, >) and assignment and simple arithmetic.

Compilers have been traditionally broken up into three main components, namely, the frontend, optimizer, and the backend; nevertheless, in my project, I did not implemented an optimizer.

### Front-end

The purpose of the front-end is to create an intermediate representation. A compiler front-end consists of lexical analysis whereby the source program is broken up into tokens, the parse tree whereby the tokens get composed into a tree based on the grammar of the language; for example, the tokens of my small language are the following:

```haskell
%token
if         { TokenIf }
let        { TokenLet }
then       { TokenThen }
else       { TokenElse }
while      { TokenWhile }
var        { TokenVar $$ }
int        { TokenInt $$ }
print      { TokenPrint }
true       { TokenTrue $$}
false      { TokenFalse $$}
defun      { TokenDefun }
'='        { TokenEq }
'+'        { TokenPlus }
'-'        { TokenMinus }
'{'        { TokenCurlyL }
'}'        { TokenCurlyR }
'('        { TokenOP }
')'        { TokenCP }
']'        { TokenSqBR }
'['        { TokenSqBL }
'<'        { TokenLess }
'>'        { TokenGreater }
':'        { TokenColon }
';'        { TokenSemicolon }
%%
```

and this is the grammar, which gives the semantics:

```haskell
Exp : var  { Var $1 }
| let var '=' Exp ';' { Let $2 $4 }
| Exp '+' Exp ';' { Plus $1 $3 }
| Exp '-' Exp ';' { Minus $1 $3 }
| Exp '<' Exp ';'   { LessThn $1 $3 }
| Exp '>' Exp ';' { GreaterThn $1 $3 }
| if Exp then Exp else Exp ';' { IfExp $2 $4 $6 }
| true { Bool $1 }
| false { Bool $1 }
| print '(' Exp ')' ';' { PrintExp $3 }
| Exp ';' Exp  { Exps $1 $3 }
| while Exp ':' Exp  { While $2 $4 }
| defun Exp '(' Exp ')' '{' Exp '}'   { DefunExp $2 $4 $7 } 
| '(' Exp ')'   { TupleExp $2 }
| Exp '[' int ']'   { TupleIndex $1 $3 }
| int  { Int $1 }
| '-' int { Negative $2 }
{
```

Consider the expression:

```
"let x = 0;; while x < 4;: if x < 5; then print(x);; let x = x + 1;; else print(3);;"
```

The lexer, which breaks programs such as this into tokens would produce the following tokens:

```haskell
[
    TokenLet,
    TokenVar "x",
    TokenEq,
    TokenInt 0,
    TokenSemicolon,
    TokenSemicolon,
    TokenWhile,
    TokenVar "x",
    TokenLess,
    TokenInt 4,
    TokenSemicolon,
    TokenColon,
    TokenIf,
    TokenVar "x",
    TokenLess,
    TokenInt 5,
    TokenSemicolon,
    TokenThen,
    TokenPrint,
    TokenOP,
    TokenVar "x",
    TokenCP,
    TokenSemicolon,
    TokenSemicolon,
    TokenLet,
    TokenVar "x",
    TokenEq,
    TokenVar "x",
    TokenPlus,
    TokenInt 1,
    TokenSemicolon,
    TokenSemicolon,
    TokenElse,
    TokenPrint,
    TokenOP,
    TokenInt 3,
    TokenCP,
    TokenSemicolon,
    TokenSemicolon
]
```

and using the above grammar, the parser generator creates the following abstract syntax tree:

```haskell
Exps (Let "x" (Int 0)) (While (LessThn (Var "x") (Int 4)) (IfExp (LessThn (Var "x") (Int 5)) (Exps (PrintExp (Var "x")) (Let "x" (Plus (Var "x") (Int 1)))) (PrintExp (Int 3))))
```

In other words, one of the main purposes of the compiler's front-end is to translate concrete syntax -- i.e., the text of the program -- to an abstract syntax tree which is a representation used inside the compiler.

After generating the parse tree, the other purpose of the front-end is to generate a machine independent intermediate representation; this could be a graph or an abstract syntax tree.

In my compiler, I used an intermediate language called *A Normal Form* which lowers the AST generated by the parser to a language consisting of only atomic and complex expressions. Atomic expressions are expressions that have no side effects and can essentially be part of assembly instructions.

For example, consider the AST generated for this expression:

```
let x = 4 + -3
```

The *A Normal Form* generated for this is the following:

```haskell
SeqMon (MonLet "temp_0" (MonNegative 3)) (MonLet "x" (MonPlus (AtmInt 4) (AtmVar "temp_0")))
```

But a more interesting problem from an implementation side is generating the *A Normal Form* for the following expression:

```
if if true then true else false; then 1 else 2;
```

Given that the condition of the *if* expression can be another *if* expression you need a cool recursive algorithm:

```haskell
toMon (IfExp (IfExp cnd thn els) thn2 els2) counter =
  let tmpname = "temp_" ++ show counter in
    let counter2 = counter + 1 in
      let tmpname2 = "temp_" ++ show counter2 in
        let counter3 = counter2 + 1 in
          let counter4 = counter3 + 1 in
            MonLstSeq (MonLet tmpname (toMon cnd counter3)) (MonLet tmpname2 (toMon (IfExp (Var tmpname) thn els) counter4)) (toMon (IfExp (Var tmpname2) thn2 els2) counter4)
```

which generates the following A Normal Form for the *if* expression above

```haskell
MonLstSeq (MonLet "temp_0" (AtmVar "true")) (MonLet "temp_1" (MonIf (AtmVar "temp_0") (AtmVar "true") (AtmVar "false"))) (MonIf (AtmVar "temp_1") (AtmInt 1) (AtmInt 3))
```

#### Directed Acyclic Graph

The ANF that you see above is an abstract syntax tree but instead of an AST you can lower the AST of the parse tree to a directed acyclic graph (DAG). A DAG is similar to a tree so it can be constructed similary and, although I have not implemented a DAG, the way to do it is while traversing the AST of the parse tree, you can check whether a node has been constructed in the DAG. If it is you can just return that. The advantage of DAGs over ASTs is that you do not create a node for identical sub expressions and as a result you can generate assembly consisting of less instructions.

### Back-end

Lets continue with the above example:

```
let x = 0;; while x < 4;: if x < 5; then print(x);; let x = x + 1;; else print(3);;
```

The ANF generated for the above example is this:

```
SeqMon (MonLet "x" (AtmInt 0)) (MonWhile (MonLessThn (AtmVar "x") (AtmInt 4)) (MonIf (MonLessThn (AtmVar "x") (AtmInt 5)) (SeqMon (MonPrint (AtmVar "x")) (MonLet "x" (MonPlus (AtmVar "x") (AtmInt 1)))) (MonPrint (AtmInt 3))))
```

After the ANF pass, the instruction selection pass is next. The purpose of the instructor selector is to *select* the appropriate x86 instructions. The intermediate language for this will be *three address code* which models assembly instructions such as:

```x86asm
movq $1, %rax
```

As you can see, the instruction above has three components.

The instruction selection pass generates the following for the running example:

```haskell
[
    ("movq", ImmInt 0, ImmStr "x"),
    ("whiletest", ImmStr "whiletestlabel", ImmStr ":"),
    ("cmpq", ImmInt 4, ImmStr "x"),
    ("jge", ImmStr "exit", ImmStr "dummy"),
    ("jmp", ImmStr "iftest", ImmStr "dummy"),
    ("iftest", ImmStr "iftestlabel", ImmStr "dummy"),
    ("cmpq", ImmInt 5, ImmStr "x"),
    ("jmp", ImmStr "block_0", ImmStr "dummy"),
    ("je", ImmStr "block_1", ImmStr "dummy"),
    ("block_0", ImmStr "blkdummy", ImmStr "dummy"),
    ("movq", ImmStr "x", ImmReg "%rdi"),
    ("print", ImmStr "dummy", ImmStr "dummy"),
    ("incq", ImmStr "x", ImmStr "dummy"),
    ("jmp", ImmStr "whiletest", ImmStr "thanks"),
    ("block_1", ImmStr "dummy", ImmStr "dummy"),
    ("movq", ImmInt 3, ImmReg "%rdi"),
    ("print", ImmStr "dummy", ImmStr "dummy"),
    ("jmp", ImmStr "whiletest", ImmStr "thanks"),
    ("exit", ImmStr "retq", ImmStr "dummy")
]

```

The relevant code that generates this is as follows:

```haskell

toSelect (MonLet var (AtmInt n)) =
  [("movq", ImmInt n, ImmStr var)]

toSelect (MonWhile cnd exps) =
  let selectcnd = toSelect cnd in
    let selectexps = toSelect exps in
      [("whiletest", ImmStr "dummy", ImmStr "dummy")] ++ selectcnd ++ [("jge", ImmStr "exit", ImmStr "dummy"), ("jmp", ImmStr "loop", ImmStr "dummy")]++[("loop", ImmStr "tst", ImmStr "tstdummy")] ++ selectexps ++ [("jmp", ImmStr "whiletest", ImmStr "dummy")] ++ [("exit", ImmStr "retq", ImmStr "dummy")]

toSelect (MonLessThn (AtmInt n) (AtmInt n2)) =
  [("cmpq", ImmInt n2, ImmStr (show n))]
In this pass, instead of using stack locations or registers we use the actual variable names. In the next pass we assign stack locations

toSelect (MonWhile cnd (MonIf cnd2 thn els)) =
  let cif = toCLikewhile (MonIf cnd2 thn els) 0 ("jmp", "to", MonBlock "whiletest", "thanks", ":)")in
    let cselect = toSelect cif in
      let selectcnd = toSelect cnd in
      [("whiletest", ImmStr "whiletestlabel", ImmStr ":")] ++ selectcnd ++ [("jge", ImmStr "exit", ImmStr "dummy"), ("jmp", ImmStr "iftest", ImmStr "dummy")] ++ [("iftest", ImmStr "iftestlabel", ImmStr "dummy")] ++ cselect ++ [("exit", ImmStr "retq", ImmStr "dummy")]

```

Note: the dummy arguments are one of the weaknesses of my implementation. In the next iteration of the project I am going to build a much cleaner design.

Anyways, the instruction selection pass generates three address code whose addresses are variables from the source program.

Since I did not implement register allocation, I have another pass which assigns stack locations:

```haskell
[
    ("movq", ImmInt 0, ImmStack "-8(%rbp)"),
    ("whiletest", ImmStr "whiletestlabel", ImmStr ":"),
    ("cmpq", ImmInt 4, ImmStack "-8(%rbp)"),
    ("jge", ImmStr "exit", ImmStr "dummy"),
    ("jmp", ImmStr "iftest", ImmStr "dummy"),
    ("iftest", ImmStr "iftestlabel", ImmStr "dummy"),
    ("cmpq", ImmInt 5, ImmStack "-8(%rbp)"),
    ("jmp", ImmStr "block_0", ImmStr "dummy"),
    ("je", ImmStr "block_1", ImmStr "dummy"),
    ("block_0", ImmStr "blkdummy", ImmStr "dummy"),
    ("movq", ImmStack "-8(%rbp)", ImmReg "%rdi"),
    ("print", ImmStr "dummy", ImmStr "dummy"),
    ("incq", ImmStack "-8(%rbp)", ImmStr "dummy"),
    ("jmp", ImmStr "whiletest", ImmStr "thanks"),
    ("block_1", ImmStr "dummy", ImmStr "dummy"),
    ("movq", ImmStr "3", ImmReg "%rdi"),
    ("print", ImmStr "dummy", ImmStr "dummy"),
    ("jmp", ImmStr "whiletest", ImmStr "thanks"),
    ("exit", ImmStr "retq", ImmStr "dummy")
]

```

And the code that generates stack locations is here:

```haskell
toStackHelper :: [(String, Imm, Imm)] -> Int -> Map.Map String String -> [(String, Imm, Imm)]
toStackHelper [] _ _ = []
toStackHelper (("movq", ImmInt n, ImmReg "%rdi"):xs) counter hashmap =
  ("movq", ImmStr (show n), ImmReg "%rdi") : toStackHelper xs counter hashmap
  
toStackHelper (("movq", ImmInt imm1, ImmStr tmp1):xs) counter hashmap =
    let (stacklocation, counter', hashmap') =
            if Map.member tmp1 hashmap
            then (hashmap Map.! tmp1, counter, hashmap)
            else let counter' = counter + 8
                     stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                     hashmap' = Map.insert tmp1 stacklocation' hashmap
                 in (stacklocation', counter', hashmap')
    in
        ("movq", ImmInt imm1, ImmStack stacklocation) : toStackHelper xs counter' hashmap'

toStackHelper (("addq", ImmInt imm1, ImmStr tmp1):xs) counter hashmap =
  let (stacklocation, counter', hashmap') =
        if Map.member tmp1 hashmap
        then (hashmap Map.! tmp1, counter, hashmap)
        else let counter' = counter + 8
                 stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                 hashmap' = Map.insert tmp1 stacklocation' hashmap
               in (stacklocation', counter', hashmap')
   in
    ("addq", ImmStr (show imm1), ImmStack stacklocation) : toStackHelper xs counter' hashmap'

toStackHelper (("addq", ImmStr imm1, ImmStr tmp1):xs) counter hashmap =
  let (stacklocation, counter', hashmap', stacklocation2) = (hashmap Map.! tmp1, counter, hashmap, hashmap Map.! imm1)
       
   in
     ("movq", ImmStr stacklocation, ImmReg "%rax") : ("addq", ImmReg "%rax", ImmStack stacklocation2) : toStackHelper xs counter' hashmap'

toStackHelper (("incq", ImmStr x, ImmStr dummy):xs) counter hashmap =
  let (stacklocation, counter', hashmap') = (hashmap Map.! x, counter, hashmap)
  in
    ("incq", ImmStack stacklocation, ImmStr "dummy") : toStackHelper xs counter' hashmap'
  
toStackHelper (("subq", ImmInt n, ImmStr tmp1):xs) counter hashmap =
  let (stacklocation, counter', hashmap') =
        if Map.member tmp1 hashmap
        then (hashmap Map.! tmp1, counter, hashmap)
        else let counter' = counter + 8
                 stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                 hashmap' = Map.insert tmp1 stacklocation' hashmap
               in (stacklocation', counter', hashmap')
   in
    ("subq", ImmStr "dummy", ImmStack stacklocation) : toStackHelper xs counter' hashmap'
    
toStackHelper (("movq", ImmStr "True", ImmStr tmp1):xs) counter hashmap =
    let (stacklocation, counter', hashmap') =
            if Map.member tmp1 hashmap
            then (hashmap Map.! tmp1, counter, hashmap)
            else let counter' = counter + 8
                     stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                     hashmap' = Map.insert tmp1 stacklocation' hashmap
                 in (stacklocation', counter', hashmap')
    in
        ("movq", ImmStr "True", ImmStack stacklocation) : toStackHelper xs counter' hashmap'

toStackHelper (("movq", ImmStr "False", ImmStr tmp1):xs) counter hashmap =
    let (stacklocation, counter', hashmap') =
            if Map.member tmp1 hashmap
            then (hashmap Map.! tmp1, counter, hashmap)
            else let counter' = counter + 8
                     stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                     hashmap' = Map.insert tmp1 stacklocation' hashmap
                 in (stacklocation', counter', hashmap')
    in
        ("movq", ImmStr "True", ImmStack stacklocation) : toStackHelper xs counter' hashmap'
        
toStackHelper (("cmpq", ImmStr bool, ImmStr tmp1):xs) counter hashmap =
  let (stacklocation, counter', hashmap') = (hashmap Map.! tmp1, counter, hashmap)
       
   in
     ("cmpq", ImmStr bool, ImmStack stacklocation) : toStackHelper xs counter' hashmap'

toStackHelper (("cmpq", ImmInt n, ImmStr tmp1):xs) counter hashmap =
  let (stacklocation, counter', hashmap') = (hashmap Map.! tmp1, counter, hashmap)
       
   in
     ("cmpq", ImmInt n, ImmStack stacklocation) : toStackHelper xs counter' hashmap'

toStackHelper (("movq", ImmStr imm1, ImmReg "%rdi"):xs) counter hashmap =
    let (stacklocation, counter', hashmap') =
            if Map.member imm1 hashmap
            then (hashmap Map.! imm1, counter, hashmap)
            else let counter' = counter + 8
                     stacklocation' = "-" ++ show counter' ++ "(%rbp)"
                     hashmap' = Map.insert imm1 stacklocation' hashmap
                 in (stacklocation', counter', hashmap')
    in
        ("movq", ImmStack stacklocation, ImmReg "%rdi") : toStackHelper xs counter' hashmap'

toStackHelper (("print", ImmStr "dummy", ImmStr "dummy"):xs) counter hashmap =
  ("print", ImmStr "dummy", ImmStr "dummy") : toStackHelper xs counter hashmap

toStackHelper (("movq", ImmStr v, ImmStr v2):xs) counter hashmap =
   let (stacklocation, counter', hashmap', stacklocation2) =
            if Map.member v hashmap && not (Map.member v2 hashmap)
            then let counter' = counter + 8
                     stacklocation2 = "-" ++ show counter' ++ "(%rbp)"
                     hashmap' = Map.insert v2 stacklocation2 hashmap
                 in
                   (hashmap Map.! v, counter', hashmap', stacklocation2)
            else (hashmap Map.! v, counter, hashmap, hashmap Map.! v2)
    in
     ("movq", ImmStack stacklocation, ImmReg "%rax") : ("movq", ImmReg "%rax", ImmStack stacklocation2) : toStackHelper xs counter' hashmap'

toStackHelper (x:xs) counter hashmap =
  x : toStackHelper xs counter hashmap

```

And finally we generate the actual x86:

```x86asm
    .globl main
main:
    pushq %rbp
    movq %rsp, %rbp
    subq $8, %rsp
    movq $0, -8(%rbp)
whiletest:
    cmpq $4, -8(%rbp)
    jge exit
    jmp iftest
iftest:
    cmpq $5, -8(%rbp)
    jmp block_0
    je block_1
block_0:
    movq -8(%rbp), %rdi
    callq print_int
    incq -8(%rbp)
    jmp whiletest
block_1:
    movq $3, %rdi
    callq print_int
    jmp whiletest
exit:
    addq $8, %rsp
    popq %rbp
    retq

```

which is associated with this code:

```haskell
toX86' :: [(String, Imm, Imm)] -> String
toX86' [] = ""

toX86' (x:xs) =
  toX86W x ++ toX86' xs
  
toX86W :: (String, Imm, Imm) -> String
toX86W ("start", ImmStr dummy, ImmStr dummy2) =
  "\t.globl main\n" ++ "main:\n" ++ "\tpushq %rbp\n" ++ "\tmovq %rsp, %rbp\n" ++ "\tsubq $8, %rsp\n" 

toX86W ("movq", ImmStr "True", ImmStack stack) =
  "\tmovq $1, " ++ stack ++ "\n" 

toX86W ("cmpq", ImmStr "True", ImmStack stack) =
  "\tcmpq $1, " ++ stack ++ "\n" 
toX86W ("cmpq", ImmReg reg, ImmReg reg2) =
  "\tcmpq " ++ reg ++ "," ++ reg2 ++ "\n"
  
toX86W ("jmp", ImmStr block, ImmStr dummy) =
  "\tjmp " ++ block ++ "\n" 

toX86W ("je", ImmStr block, ImmStr dummy) =
  "\tje " ++ block ++ "\n"

toX86W ("jge", ImmStr block, ImmStr dummy) =
  "\tjge " ++ block ++ "\n"

toX86W ("jl", ImmStr block, ImmStr dummy) =
  "\tjl " ++ block ++ "\n" 
  
toX86W ("exit", ImmStr "retq", ImmStr dmy) =
  "exit:\n" ++ "\taddq $8, %rsp\n" ++ "\tpopq %rbp\n" ++  "\tretq\n"
  
toX86W ("movq", ImmReg reg, ImmReg reg2) =
  "\tmovq "  ++ reg ++ "," ++ reg2 ++ "\n"

toX86W ("movq", ImmInt n, TupleMem tmp) =
   "\tmovq "  ++ "$" ++ show n ++ "," ++ tmp ++ "\n"
   
toX86W ("movq", TupleMem tmp, ImmReg reg) =
   "\tmovq " ++ tmp ++ "," ++ reg ++ "\n"
   
toX86W ("movq", ImmStr n, ImmReg reg) =
  "\tmovq "  ++ n ++ "," ++ reg ++ "\n"

   
toX86W ("movq", ImmInt n, ImmReg reg) =
  "\tmovq "  ++ "$" ++ show n ++ "," ++ reg ++ "\n"
  
toX86W ("movq", ImmInt n, ImmStack stk) =
  "\tmovq $" ++ show n ++ ", " ++ stk ++ "\n"

toX86W ("movq", ImmStack stk, ImmReg reg) =
  "\tmovq " ++ stk ++ ", " ++ reg ++ "\n"

toX86W ("incq", ImmStack stk, ImmStr dummy) =
  "\tincq " ++ stk ++ "\n"

toX86W ("callq", ImmStr fn, ImmStr dmy) =
  "\tcallq " ++ fn ++ "\n"
  
toX86W ("cmpq", ImmInt n, ImmStack stk) =
    "\tcmpq $" ++ show n ++ ", " ++ stk ++ "\n"

toX86W ("print", ImmStr dm, ImmStr dm2) =
  "\tcallq " ++ "print_int\n" 
  
toX86W (b1, ImmStr dm, ImmStr dm2)=
  b1 ++ ":\n"

toX86W ("addq", ImmInt n, ImmReg reg) =
  "\taddq " ++ "$" ++ show n ++ ", " ++ reg ++ "\n"

toX86W ("addq", ImmInt n, ImmStr fp) =
  "\taddq " ++ "$" ++ show n ++ ", " ++  fp ++ "\n"
```

### Tuples

It was fun compiling tuples. Tuples motivate garbage collector. The garbage collector of my system (which was not written by me) is a two space collector. The collector is divided into regions: *ToSpace* and *FromSpace*. To make room, the garbage collector, moves the reachable objects from the *FromSpace* to *ToSpace*.

To compile tuples you need to make a 64 bit tag for the garbage collector to distinguish it.
Tuples go through the same passes as above. In the 64 bit tag, there are bits corresponding to the size of the tuple, bits that corresponds to whether an element of the tuple is a number or another tuple and bits that correspond to whether the tuple is reachable from the root set.

A given tuple containing expression

```
let x = (4;5;6);; print(x[1]);
```

it generates the following x86

```x86asm
    .globl main
main:
    pushq %rbp
    movq %rsp, %rbp
    subq $0, %rsp
    movq $65536, %rdi
    movq $65536, %rsi
    callq initialize
    movq rootstack_begin(%rip), %r15
    movq $0, 0(%r15)
    addq $8, %r15
    jmp start
start:
    movq free_ptr(%rip), %rax
    addq $24, %rax
    movq fromspace_end(%rip), %r13
    cmpq %r13, %rax
    jl block_77
    jmp block_78
block_77:
    movq $0, %r13
    jmp block_80
block_78:
    movq %r15, %rdi
    movq $24, %rsi
    callq collect
    jmp block_80
block_80:
    movq free_ptr(%rip), %r11
    addq $32, free_ptr(%rip)
    movq $7, 0(%r11)
    movq %r11, %r14
    movq %r14, %r11
    movq $4, 8(%r11)
    movq $5, 16(%r11)
    movq $6, 24(%r11)
    movq 16(%r11), %rdi
    callq print_int
    jmp conclusion
conclusion:
    subq $8, %r15
    addq $0, %rsp
    popq %rbp
    retq

```

### Lessons from using Haskell

I have written compilers in Common Lisp and after writing a compiler in  Haskell I noticed that I was able to structure my code better. In compilers you deal with a lot of abstract syntax tree transformations. Defining data types with `data` enables you to manipulate ASTs better. Haskell’s pattern matching facility makes AST transformation feasible as well. I liked how I can break down a function into small modules based on the AST pattern. All of this enables you to structure your code.

Haskell lets you write:

Correct code, if a Haskell program compiles then it just works. Why is this? It has to do with the type system. By defining data types for each intermediate language and a signature enforced by the type system, you are forced to write more complete/less sloppy code and you are also guaranteed correctness. In addition, since the type system is about proving certain aspects of your program, when developing a program in Haskell, if the given program does not compile then the type errors will guide you to, in essence, prove your program correct iteratively.

If type checkers are primitive proof systems then type checkers prove parts of your system correct. If type checkers prove parts of you’re program correct then type errors will guide you prove the correctness of some parts of your program correct iteratively. 

There are a few things I would do differently next time. I think I need to think ahead of time the structure of my data; for example, the instruction selection pass (ToSelect.hs) returns an Imm data type. An Imm is either an ImmStr, ImmReg, or ImmTupMem. Some ImmStr’s contain dummy variables “dummy”.

