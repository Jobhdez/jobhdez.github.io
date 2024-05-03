---
layout: post
title: "Compiler problems and their implementation"
author: Job Hernandez
tags: compilers 
---

*version 0.9.0*, draft 5/2/24

So, far in my compiler journey I have faced three interesting problems. The problems were translating a subset of language to $$ a normal form $$, compiling tuples to x86, and floating point math.

### Compiling without continuations

A-Normal form is an intermediate language for functional programming languages but it can be used to compile languages such as Python.

It was introduced in a paper called “The Essence of Continuations”, in which the authors argued that ANF is equivalent to continuation passing style but much simpler.

ANF consists of atomic and complex expressions. An atomic expression is an expression that is readily computable in x86 assembly.

Recently, I built a compiler in Haskell for a subset of a language.

Here is the grammar:

```
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
```

Once A-Normalization is applied you end up something like this:

```
atomic  :: number | boolean | var | < | > | + | -
complex :: (atomic, atomic,..., atomic) | (if <atomic> <complex> <complex>) | (while <atomic> <complex>) | (tuple <atomic>)
exp     :: (let var <complex>) | atomic | complex
```

The interesting conversion was the $$ if expression $$. It required some thought. I initially wrote it in Racket and then rewrote it in Haskell. Here is both of the implementations:

Racket:

```
(define (syntax->anf ast)

  (define counter (make-parameter 0))
 
  (define (generate-temp-name name)
	(let* [(current-counter (counter))
       	(name* (string-append name (number->string current-counter)))]
  	(counter (+ current-counter 1))
  	name*))
 
  (define (to-anf ast)
	(match ast
  	    ;; …
           
     	[(py-if-exp cnd thn els)
      	(let* [(temp-name (generate-temp-name "temp-"))
             	(temp-name2 (generate-temp-name "temp-"))]
        	(list (atomic-assignment (atomic (py-id temp-name)) (to-anf cnd))
              	(atomic-assignment (atomic (py-id temp-name2))
                                 	(to-anf (py-if-exp (py-id temp-name)
                                                    	thn
                                                    	els)))
              	(to-anf (py-if-exp (py-id temp-name2) then-exp else-exp))))])]

```

And here is my Haskell implementation:

```haskell
toMon (IfExp (Bool b) thn els) counter =
  MonIf (toMon (Bool b) counter) (toMon thn counter) (toMon els counter)
toMon (IfExp (Var v) thn els) counter =
  MonIf (toMon (Var v) counter) (toMon thn counter) (toMon els counter)
toMon (IfExp (IfExp cnd thn els) thn2 els2) counter =
  let tmpname = "temp_" ++ show counter in
	let counter2 = counter + 1 in
  	let tmpname2 = "temp_" ++ show counter2 in
    	let counter3 = counter2 + 1 in
      	let counter4 = counter3 + 1 in
        	MonLstSeq (MonLet tmpname (toMon cnd counter3)) (MonLet tmpname2 (toMon (IfExp (Var tmpname) thn els) counter4)) (toMon (IfExp (Var tmpname2) thn2 els2) counter4)
   	 
toMon (IfExp cnd thn els) counter =
  MonIf (toMon cnd counter) (toMon thn counter) (toMon els counter)
```

This problem requires thinking recursively which I enjoyed. The main differences between these two solutions is that in the Racket version I use mutation to create the temp variables. On the other hand, the Haskell version required me to think recursively as well but I needed to pass a counter and increment this counter to create the temp variables. 

### Interfacing with the garbage collector

Tuples motivate the need for garbage collection. The garbage collector needs a way to identify tuples (pointers) from other data. As a result, a 64 bit tag gets placed in the front of tuple data.

In the garbage collector, there are two regions: a fromSpace and toSpace region. When you allocate a tuple, a graph is constructed internally. To make space for new allocations all the tuples reachable from the root set of the graph gets copied into the toSpace region. After this the fromSpace region gets turned into the toSpace region and the toSpace becomes the fromSpace region.

#### Example

Suppose you have a tuple: (1,1,3).

The 64 bit tag for this tuple would be 000 000011 1.In assembly I will be using instructions that manipulate 64 bits.

The first three 0 bits are 0s because when processing a tuple you assign a 0 bit to an integer and  a 1 bit to a tuple.

000011 on the other hand is the length of the tuple. In this case 3.

The last bit represents whether it's reachable from the root set.

example using this code:

```
> makeTag (1;1;3)
7
```
The tag 7 will be placed in front of the tuple. 

In my Haskell compiler, the example 

```
let x = (4;5;6);; print(x[1]);
```

Compiles to the following x86:

```
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
    movq free_ptr(%rip),%rax
    addq $24, %rax
    movq fromspace_end(%rip),%r13
    cmpq %r13,%rax
    jl block_77
    jmp block_78
block_77:
    movq $0,%r13
    jmp block_80
block_78:
    movq %r15,%rdi
    movq $24,%rsi
    callq collect
    jmp block_80
block_80:
    movq free_ptr(%rip),%r11
    addq $32, free_ptr(%rip)
    movq $7,0(%r11)
    movq %r11,%r14
    movq %r14,%r11
    movq $4,8(%r11)
    movq $5,16(%r11)
    movq $6,24(%r11)
    movq 16(%r11),%rdi
    callq print_int
    jmp conclusion
conclusion:
    subq $8, %r15
    addq $0, %rsp
    popq %rbp
    retq
```

### Floating point math

Before my contribution to LLVM, I took floating point numbers as granted but it turns out that floating point mathematics is an entire world. 

Floating point math is interesting because real numbers can be infinite but the machine is finite; as a result, one needs to represent an infinite number in a finite machine.

The issue I did for LLVM is not floating point arithmetic but it was indeed a floating point issue. 

I implemented fmaximum and fminimum and their variants. In this type of problem I had to return NaN values in some cases.

But working on this issue required me to change around 140 files! So, it was the most complex thing, engineering wise, that I had done. My git skills improved a lot and my emacs’ skills did to.

But anyways, here is the code that I wrote:

```c++
template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fmaximum(T x, T y) {
  FPBits<T> bitx(x), bity(y);

  if (bitx.is_nan())
	return x;
  if (bity.is_nan())
	return y;
  if (bitx.sign() != bity.sign())
	return (bitx.is_neg() ? y : x);
  return x > y ? x : y;
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fminimum(T x, T y) {
  const FPBits<T> bitx(x), bity(y);

  if (bitx.is_nan())
	return x;
  if (bity.is_nan())
	return y;
  if (bitx.sign() != bity.sign())
	return (bitx.is_neg()) ? x : y;
  return x < y ? x : y;
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fmaximum_num(T x, T y) {
  FPBits<T> bitx(x), bity(y);
  if (bitx.is_signaling_nan() || bity.is_signaling_nan()) {
	fputil::raise_except_if_required(FE_INVALID);
	if (bitx.is_nan() && bity.is_nan())
  	return FPBits<T>::quiet_nan().get_val();
  }
  if (bitx.is_nan())
	return y;
  if (bity.is_nan())
	return x;
  if (bitx.sign() != bity.sign())
	return (bitx.is_neg() ? y : x);
  return x > y ? x : y;
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fminimum_num(T x, T y) {
  FPBits<T> bitx(x), bity(y);
  if (bitx.is_signaling_nan() || bity.is_signaling_nan()) {
	fputil::raise_except_if_required(FE_INVALID);
	if (bitx.is_nan() && bity.is_nan())
  	return FPBits<T>::quiet_nan().get_val();
  }
  if (bitx.is_nan())
	return y;
  if (bity.is_nan())
	return x;
  if (bitx.sign() != bity.sign())
	return (bitx.is_neg() ? x : y);
  return x < y ? x : y;
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fmaximum_mag(T x, T y) {
  FPBits<T> bitx(x), bity(y);

  if (abs(x) > abs(y))
	return x;
  if (abs(y) > abs(x))
	return y;
  return fmaximum(x, y);
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fminimum_mag(T x, T y) {
  FPBits<T> bitx(x), bity(y);

  if (abs(x) < abs(y))
	return x;
  if (abs(y) < abs(x))
	return y;
  return fminimum(x, y);
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fmaximum_mag_num(T x, T y) {
  FPBits<T> bitx(x), bity(y);

  if (abs(x) > abs(y))
	return x;
  if (abs(y) > abs(x))
	return y;
  return fmaximum_num(x, y);
}

template <typename T, cpp::enable_if_t<cpp::is_floating_point_v<T>, int> = 0>
LIBC_INLINE T fminimum_mag_num(T x, T y) {
  FPBits<T> bitx(x), bity(y);

  if (abs(x) < abs(y))
	return x;
  if (abs(y) < abs(x))
	return y;
  return fminimum_num(x, y);
}
````
