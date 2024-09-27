---
layout: post
title: "Improving the performance of fmul with double-double arithmetic"
author: Job Hernandez Lara
tags: [computer-science, floating-point-arithmetic, llvm]
---

### Introduction

Lately, I co-authored a PR that improved the performance of fmul by 7x by using double-double arithmetic. The following is a summary of my conversation with Tue Ly, the maintainer of LLVM libc.

C and C++ have floating point operations whose input and output types are of the same type, but C23, the new standard, introduces a new family of functions, `[f|d]add/sub/mul/div/sqrt` whose output types are narrower than the input types; for example, `fmul: double -> double -> float`. A few months ago I added the function `fmul: double -> double -> float` in this [pr](https://github.com/llvm/llvm-project/commit/263be9fb0085001630e0431782a0ac0da7ab58ae). Later someone else templatized *fmul* to cover other signatures.

*fmul* is correct but slower than the normal functions so our goal was to improve the performance of `fmul: double -> double -> float`.

### Exact Multiplication

If $$ x_{1} $$ and $$ x_{2} $$ are radix-2 precision-p floating point numbers, whose exponents $$ e_{x1} $$ and $$ e_{x2} $$ satisfy $$ e_{x1} + e_{x2} <  e_{min} + p - 1 $$ and if $$ r $$ is $$ R(x_{1} x_{2}) $$ where R is a rounding function then $$ t = x_{1}x_{2} - r $$ is a radix 2 precision-p floating point number.[^1]

Using the _2MultFMA_ algorithm we can compute $$ t $$:

$$ r_{1} + r_{2} = x_{1} * x_{2} $$ and $$ r_{1} = RN(x_{1} * x_{2}) $$.

More specifically:

<br />
Ensure: $$ r_{1} + r_{2} = x_{1}x_{2} $$

R1 <-  $$ RN(x_{1} * x_{2}) $$

R2 <- $$ RN(x_{1} * x_{2} - r1) $$

Return (r1, r2)
<br />

So, the algorithm we have for `exact_mult` in C++ is essentially:

```c++
   prod_hi = static_cast<double>(a * b);
   prod_lo = std::fma(a, b, -prod_hi); // a * b - prod_hi exactly
```

Or if you want to play with this in Sollya:

```
> a = 2^8 + 1 + 2^-16 + 2^-31;
> b = 2^15 - 1;
> prod_hi = round(a * b, D, RN);
> prod_lo = a * b - prod_hi;
```

The above algorithm is the most performant when FMA instructions are available. When FMA instructions are not available the result are not correct in rounding modes other than *FE_TONEAREST*.

The product of `a * b` might not fit in a double precision, so *exact mult* will return `prod.hi` and `prod.lo` so that $$ a * b = prod.hi + prod.lo $$.



### Double-Double rounding errors

One type of error that we had to overcome was double-double rounding errors. I’ll explain the problem and the error.

Here is the running example:

```
> a = 2^8 + 1 + 2^-16 + 2^-31; 
> b = 2^15 - 1; 
> display = binary;
> a * b;
1.000000001111110111111110111111111111111111111111111111_2 * 2^(23)
```

Since the output type is narrower than the input type – i.e., `double -> double -> float` – then we need to round to the nearest 24 bits. That's what happens when you round to `float`, i.e., `float` is 23 bit mantissa + 1 hidden bit.

Lets put a separator after the first 24 bits:

```
1.00000000111111011111111 | 0111111111111111111111111111111_2
```

So, when we round to the first 24 bits, we have:

```
1.00000000111111011111111
```

Now, we need to check what the rounding bit of the floating point number. If the rounding bit is 0 we will round down, in which case the answer is the same as the “integer part”. In the running example, the rounding bit is the bit right after the separator which is a 0. Now, on the other hand, if the rounding bit is a 1 we need to check the sticky bits which consist of the rest of the bits.

If the sticky bits are non-zero, we round up.

If the sticky bits are zero, we have a tie. And the default rounding mode is round-to-nearest, *tie-to-even* so you round up or down depending on which direction gives you an even number, i.e., the least-significant bit of the answer is 0.

Now consider rounding `a*b` from above to the nearest 53 bits:

So rounding:
```
1.000000001111110111111110111111111111111111111111111111_2
```
to the nearest 53-bit gives you:

`1.0000000011111101111111101111111111111111111111111111 | 11_2`

All the bits on the left of `|` are integer part, the bit right on the right of `| ` is the rounding bit and
the remaining bits on the right of the rounding bit are the sticky bits.

When we round up, we simply increase the "integer part" by 1 and the integer part above is:

```
1.0000000011111101111111101111111111111111111111111111
```
and if you add the whole thing by 1 lowest bit, it will be?

```
1.0000000011111101111111101111111111111111111111111111
->
1.0000000011111101111111110000000000000000000000000000
```

and that will be the bits when we do 

```
round(a*b, D, RN)
```
Which is how we compute `prod.hi` in the running example above.

Now, we need to round `prod.hi` to the nearest 24 bits – i.e., float.

So, now we need to do:

```
> round(a * b, SG, RN); // round to 24-bit precision
```

So the *exact_mult* will return prod.hi and prod.lo so that a * b = prod.hi + prod.lo mathematically. *exact_mult* is an internal LLVM libc function that we used to get *prod.hi* and *prod.lo*.

We need to round `prod.hi` to the nearest 24 bits. So, we proceed as follows:

*prod.hi* is equal to the bit string:

```
1.0000000011111101111111110000000000000000000000000000
```

To visualize this lets add a separator after the 24th bit:


```
1.00000000111111011111111|10000000000000000000000000000
```
This means we need to round to even because the round bit is non zero and the sticky bits are 0. 
Round to even means that the last significant value is 0.

In other words, first we get the leading 24 bit ("integer part"):

```
1.00000000111111011111111
```

then we look at the rounding bit, which is 1 and whose sticky bits are the 0s after that.

You round up or down depending on which direction gives you an even number, i.e., the least-significant bit of the answer is 0.

In this case, we round up or down dispensing on what gives you an even number.

So, we have the bits,

```
1.00000000111111011111111
```

Rounding down means

```
1.00000000111111011111111 | 10000000000 -> 1.00000000111111011111111
```
And rounding up means

```
1.00000000111111011111111 | 10000000000 -> 1.00000000111111011111111 + 1 ULP
```

Since the least significant bit has to be 0 then we round up:

```
1.00000000111111100000000
```

So, as I said above,  the algorithm we have for `exact_mult` in C++ is essentially:
```
   prod_hi = static_cast<double>(a * b);
   prod_lo = std::fma(a, b, -prod_hi); // a * b - prod_hi exactly
```

Or in Sollya:

```
> prod_hi = round(a * b, D, RN);
> prod_lo = a * b - prod_hi; // Notice that I don't need to round here, because it's exact in double precision

> prod_hi;
1.000000001111110111111111_2 * 2^(23)

> prod_hi + prod_lo;
1.000000001111110111111110111111111111111111111111111111_2 * 2^(23)
```
That will calculate the exact result which is `a*b`.

In C++ `prod_hi + prod_lo` will round the exact result to `double`.

To get this done in Sollya we need to do:

```
round(prod_hi+prod_lo, D, RN);
1.000000001111110111111111_2 * 2^(23)
```

Round to single precision:

```
> round(val, SG, RN);
1.000000001111111_2 * 2^(23)
> round(a*b, SG, RN);
1.00000000111111011111111_2 * 2^(23)
```
So what's happening here is that we are rounding one extra time.

And that's why these types of errors are called `double-rounding errors`.

To fix double double rounding errors we need to handle three cases associated with `prod.lo`.

### Solving double-double rounding errors

So now, the three cases that need to be handled are:  $$ lo > 0 $$,  $$ lo = 0 $$,  and $$ lo < 0 $$.

#### $$ lo = 0 $$
Ok, lets start with $$ lo = 0 $$. When $$ lo = 0 $$ we have $$ hi+ lo = hi $$.

Ok, so now lets say we have `hi + lo` both of which are double precision. This means that $$ lo = 0 $$ — i.e.,  $$ hi + lo = hi $$ —
means that $$ result = prod_hi $$  is enough to get correctly rounding to single precision.

#### $$ lo < 0 $$

On the other hand when $$ lo < 0 $$, let's compare `round(hi + lo, SG, RN)` and `round(hi, SG, RN)`:

```
​​> hi = 1 + 2^-23 + 2^-24;
> lo = -2^-55;
> hi = 1 + 2^-23 + 2^-24;
> lo = -2^-55;
> hi;
1.000000000000000000000011_2
> lo;
-1_2 * 2^(-55)

```
Now, let’s do the rounding — i.e., `round(hi + lo, SG, RN)` and `round(hi, SG, RN)` —

```
> hi = 1 + 2^-23 + 2^-24;
> lo = -2^-55;
> round(hi + lo, SG, RN);
1.00000000000000000000001_2
> round(hi, SG, RN);
1.0000000000000000000001_2
```

`round(hi, SG, RN)` did round up, but actually it needs to round down to match `round(hi + lo, SG, RN)`, the correct result.

So, in order for `round(hi, SG, RN)` to round up, we must have:

```
- rb in hi is 1
and at least
1) lsb in hi is 1, or
2) remaining sticky bits in hi is > 0

```
so for the default rounding modes, after getting (truncating) the "integer part", we will round up if:
```
rb && (lsb || (r != 0))
```
where

`rb` is the rounding bit; `lsb` is the least significant bit (of the integer part) and `r != 0` is that the `sticky bits` are positive.

So, in order for `round(hi, SG, RN)` to round up, we must have:

```
- rb in hi is 1
and at least
1) lsb in hi is 1, or
2) remaining sticky bits in hi is > 0
```

now, if it is case 2: `remaining sticky bits in hi is > 0`
because `|lo| <= ulp(hi) / 2`,
`remaining sticky bits in (hi + lo)` is still > 0

so the rounding action we take with respect to `hi + lo` is still round up,

i.e. `round(hi, SG, RN) = round(hi + lo, SG, RN)` in that case.
Right? Yes

so now, let consider the case `round(hi, SG, RN)` is round-up because:
```
- rb is 1,
- lsb is 1,
- but sticky bits of hi is 0
```
(which is exactly what we have with `hi = 1 + 2^-23 + 2^-24`)
so now, because `lo < 0`, if you look at the exact bits of `hi + lo`, you will see that:

```
- lsb is still 1
- rb is 0
- sticky bits of hi + lo are > 0
```
so because `rb` of `hi + lo` is 0, the correct rounding of `hi + lo` is round down, not round up

so to highlight it:
```
> hi;
1.000000000000000000000011_2
> hi + lo;
1.0000000000000000000000101111111111111111111111111111111_2
```

and lets highlight rounding bit with `[ ]`:
```
> hi;
1.00000000000000000000001[1]_2
> hi + lo;
1.00000000000000000000001[0]1111111111111111111111111111111_2
```
so in the first case, the logic is something like:

```
// At this point, hi is not 0, and lo is smaller than hi.
FPBits<double> hi_bits(hi), lo_bits(lo);
if (lo_bits.sign() != hi_bits.sign()) {
  // Check if sticky bit of hi are all 0
  constexpr uint64_t STICKY_MASK = 0xFFF'FFFF;  // Lower (52 - 23 - 1 = 28 bits)
  uint64_t sticky_bits = (hi_bits.uintval() & STICKY_MASK);
  uint64_t result_bits = (sticky_bits == 0) ? (hi_bits.uintval() - 1) : hi_bits.uintval();
  double result = FPBits<double>(result_bits).get_val();
  return static_cast<float>(result);
}
```

So, in conclusion we have something like:

```
DoubleDouble prod = exact_mult(x, y);

FPBits<double> hi_bits(prod.hi), lo_bits(prod.lo);

if (LIBC_UNLIKELY(hi_bits.is_inf_or_nan() || hi_bits.is_zero())) {
  // Deal with exceptional cases all in here
  // Your second exception check in the current PR is a dead code, to be removed.
  // All exceptional cases are in here.
  ....
}

// When low part is 0.
if (prod.lo == 0.0)
  return static_cast<float>(prod.hi);

if (lo_bits.sign() != hi_bits.sign()) {
  // Check if sticky bit of hi are all 0
  constexpr uint64_t STICKY_MASK = 0xFFF'FFFF;  // Lower (52 - 23 - 1 = 28 bits)
  uint64_t sticky_bits = (hi_bits.uintval() & STICKY_MASK);
  uint64_t result_bits = (sticky_bits == 0) ? (hi_bits.uintval() - 1) : hi_bits.uintval();
  double result = FPBits<double>(result_bits).get_val();
  return static_cast<float>(result);

}

// Now lo is of the same sign as hi, i.e. "positive".
// We will analyze it tomorrow.  Simply return prod.hi for now.
return static_cast<float>(prod.hi);
```

### Result

The final code looked something like this:

```
#ifndef LIBC_TARGET_CPU_HAS_FMA
  return fputil::generic::mul<float>(x, y);
#else
  fputil::DoubleDouble prod = fputil::exact_mult(x, y);
  using DoubleBits = fputil::FPBits<double>;
  using DoubleStorageType = typename DoubleBits::StorageType;
  using FloatBits = fputil::FPBits<float>;
  using FloatStorageType = typename FloatBits::StorageType;
  DoubleBits x_bits(x);
  DoubleBits y_bits(y);

  Sign result_sign = x_bits.sign() == y_bits.sign() ? Sign::POS : Sign::NEG;
  double result = prod.hi;
  DoubleBits hi_bits(prod.hi), lo_bits(prod.lo);
  // Check for cases where we need to propagate the sticky bits:
  constexpr uint64_t STICKY_MASK = 0xFFF'FFF; // Lower (52 - 23 - 1 = 28 bits)
  uint64_t sticky_bits = (hi_bits.uintval() & STICKY_MASK);
  if (LIBC_UNLIKELY(sticky_bits == 0)) {
	// Might need to propagate sticky bits:
	if (!(lo_bits.is_inf_or_nan() || lo_bits.is_zero())) {
  	// Now prod.lo is nonzero and finite, we need to propagate sticky bits.
  	if (lo_bits.sign() != hi_bits.sign())
    	result = DoubleBits(hi_bits.uintval() - 1).get_val();
  	else
    	result = DoubleBits(hi_bits.uintval() | 1).get_val();
	}
  }

  float result_f = static_cast<float>(result);
  FloatBits rf_bits(result_f);
  uint32_t rf_exp = rf_bits.get_biased_exponent();
  if (LIBC_LIKELY(rf_exp > 0 && rf_exp < 2 * FloatBits::EXP_BIAS + 1)) {
	return result_f;
  }

  // Now result_f is either inf/nan/zero/denormal.
// exceptions go here
```

You might be wondering why we do 

```C++
result = DoubleBits(hi_bits.uintval() - 1).get_val();
```

This ensure that when we round down to *float*  works. Because otherwise if we have $$ x.5 $$ rounding to float would not work because looking at the *hi* it looks like a tie case but it's actually not. So, that is why when the trailing parts of *hi* are 0s and we have $$ lo < 0 $$, we change the *hi* part from $$ x.5 $$ to $$ x.4999999999 $$.

And 

```C++
result = DoubleBits(hi_bits.uintval() | 1).get_val();
```
Means $$ +1 $$ because the lo bits are 0.

#### Performance 

We actually improved the performance 7x by using *double-double* arithmetic.

These are the results. 

```
 Performance tests with inputs in denormal range:
-- My function --
 	Total time  	: 2731072304 ns
 	Average runtime : 68.2767 ns/op
 	Ops per second  : 14646276 op/s
-- Other function --
 	Total time  	: 3259744268 ns
 	Average runtime : 81.4935 ns/op
 	Ops per second  : 12270913 op/s
-- Average runtime ratio --
 	Mine / Other's  : 0.837818

 Performance tests with inputs in normal range:
-- My function --
 	Total time  	: 93467258 ns
 	Average runtime : 2.33668 ns/op
 	Ops per second  : 427957777 op/s
-- Other function --
 	Total time  	: 637295452 ns
 	Average runtime : 15.9324 ns/op
 	Ops per second  : 62765299 op/s
-- Average runtime ratio --
 	Mine / Other's  : 0.146662

 Performance tests with inputs in normal range with exponents close to each other:
-- My function --
 	Total time  	: 95764894 ns
 	Average runtime : 2.39412 ns/op
 	Ops per second  : 417690014 op/s
-- Other function --
 	Total time  	: 639866770 ns
 	Average runtime : 15.9967 ns/op
 	Ops per second  : 62513075 op/s
-- Average runtime ratio --
 	Mine / Other's  : 0.149664
```

To get to the approximate 7x, you need to do $$ 1 / 0.149664 $$.

### Conclusion

Well, there you see it. This essentially how we improved the performance of `fmul: double -> double _> float` by around 7x. 

### Acknowledgements 

I want to thank Tue Ly, the LLVM libc maintainer whom I am working with, because he helped me increase my understanding of this. 


### References 
Handbook of floating point arithmetic by Jean-Michel Muller

