---
layout: post
title: "Improving the performance of fmul with double-double arithmetic"
author: Job Hernandez Lara
tags: [computer-science, floating-point-arithmetic, llvm]
---

*draft*

Lately, I co-authored a PR that improved the performance of fmul by 7x by using double-double arithmetic. The following is a summary of my conversation with Tue Ly, the maintainer of LLVM libc.

If $$ x_{1} $$ and $$ x_{2} $$ are radix-2 precision-p floating point numbers, whose exponents $$ e_{x1} $$ and $$ e_{x2} $$ satisfy $$ e_{x1} + e_{x2} <  e_{min} + p - 1 $$ and if $$ r $$ is $$ R(x_{1} x_{2}) $$ where R is a rounding function then $$ t = x_{1}x_{2} - r $$ is a radix 2 precision-p floating point number.

Using the _2MultFMA_ algorithm we can compute $$ t $$:

$$ r_{1} + r_{2} = x_{1} * x_{2} $$ and $$ r_{1} = RN(x_{1} * x_{2}) $$.

More specifically:

Ensure: $$ r_{1} + r_{2} = x_{1}x_{2} $$

R1 <-  $$ RN(x_{1} * x_{2}) $$

R2 <- $$ RN(x_{1} * x_{2} - r1) $$

Return (r1, r2)

So, the algorithm we have for `exact_mult` in C++ is essentially:

```c++
   prod_hi = static_cast<double>(a * b);
   prod_lo = std::fma(a, b, -prod_hi); // a * b - prod_hi exactly
```

Or in sollya:

```
> prod_hi = round(a * b, D, RN);
> prod_lo = a * b - prod_hi;
```

The product of `a * b` might not fit in a double precision, so _exact mult_ will return `prod.hi` and `prod.lo` so that $$ a * b = prod.hi + prod.lo $$.

Example:

```
> a = 2^8 + 1 + 2^-16 + 2^-31;
> b = 2^15 - 1;
> display = binary;
> a * b;
1.000000001111110111111110111111111111111111111111111111_2 * 2^(23)
```

Basically round to the nearest 24 bits. That's what happens when you round to `float`, i.e., `float` is 23 bit mantissa + 1 hidden bit.

```
1.00000000111111011111111 | 0111111111111111111111111111111_2
```

And when we use round to nearest, the 24 bit precision will be?

```
1.00000000111111011111111
```

Then we look at the rounding bit, the one right after the separator that was added which is `0` in this case so you will round down, and the answer is the same as the "integer part":

```
1.00000000111111011111111
```

If it's 1, we need to check the sticky bits. If the sticky bits are non-zero, we round up.

If the sticky bits are zero, we have a tie. And the default rounding mode is round-to-nearest, _tie-to-even_ so you round up or down depending on which direction gives you an even number, i.e., the least-significant bit of the answer is 0.

Now can you try to round:

```
1.000000001111110111111110111111111111111111111111111111_2
```

to the nearest 53-bit?

`1.0000000011111101111111101111111111111111111111111111|11_2`

All the bits on the left of `|` are integer part, the bit right on the right of `| ` is the rounding bit and 
the remaining bits on the right of rounding bit are sticky bits.

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

and that will be the bits when we do `round(a*b, D, RN)`.

Which is the same as `prod.hi` now, your next step is to round that (`prod.hi`) to 24 bits.

```
1.0000000011111101111111110000000000000000000000000000
```

And to the nearest 24 bits, following the same process, we do:

```
> round(a * b, SG, RN); // round to 24-bit precision
> round(a * b, D, RN); // round to 53-bit precision
```

So the rounding process, should always start with picking the "integral part".

So the exact mult will return prod.hi and prod.lo so that $$ a * b = prod.hi + prod.lo $$; for example,

we can actually see it in action with Sollya:

```
> a = 2^52 + 1; b = 2^52 + 1;
> prod_hi = round(a*b, D, RN); prod_lo = a*b - prod_hi;
> a * b;
> prod_hi;
> prod_lo;
> prod_hi + prod_lo;
```

```
prod_hi = round(a*b, D, RN);
```

so we will round that bit string to 24 bits:

```
1.0000000011111101111111110000000000000000000000000000

1.00000000111111011111111|10000000000000000000000000000
```

So we need to round to even since the round bit is non zero and the sticky bits is 0. Round to even means that the last significant value is 0.

So first we get the leading 24 bit ("integer part"):

`1.00000000111111011111111`

then we look at the rounding bit, which is?
The 1 after the bar and the sticky bits are?

The 0s after that.

So you round up or down depending on which direction gives you an even number, i.e., the least-significant bit of the answer is 0.

In this case, up or down (i.e. keep the integer part) will give you even bits?

So, we have the bits,

`1.00000000111111011111111`

Round down:

```
1.00000000111111011111111 | 10000000000 -> 1.00000000111111011111111
```

Round up:

```
1.00000000111111011111111 | 10000000000 -> 1.00000000111111011111111 + 1 ULP
```

So the round up it is:

1.00000000111111100000000

So, as I said above, the algorithm we have for `exact_mult` in C++ is essentially:

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

In C++ `prod_hi + prod_lo` will round the exact result to `double` which in sollya you do by:

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

so whats happening here is that we are rounding one extra time.

And that's why this type of errors is called `double-rounding errors`.

The fix of the double rounding errors is handling `prod.lo`.

So now, we will handle 3 cases: `lo > 0, lo = 0, lo < 0`.

Ok, lets start with $$ lo = 0 $$. When $$ lo = 0 $$ we have $$ hi + lo = hi $$.

Ok, so now lets say we have $$ hi + lo $$, 
both of which are double precision. What does this mean?

When $$ lo = 0 $$ — i.e., $$ hi + lo = hi $$ —
means that $$ result = prod_hi $$ is enough to get correctly rounding to single precision.

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

`round(hi, SG, RN)` did round up, but actually it needs to round down to match `round(hi + lo, SG, RN)`, the correct result

so, in order for `round(hi, SG, RN)` to round up, we must have:

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

`rb` is the rounding bit; `lsb` is the least significant bit (of the integer part) and `r != 0` is just `sticky bits` are positive.

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
