---
layout: post
title: "Adding floating point multiplication to LLVM libc"
author: Job Hernandez Lara
tags: [computer-science, floating-point-arithmetic, llvm]
---

Recently, my *fmul* pr got merged into LLVM. And I would like to talk about my contribution. You can see my pr [here](https://github.com/llvm/llvm-project/pull/91537).

The problem was to multiply two double precision numbers and return a single precision number – i.e.,

```c++
float fmul(double, double);
```

Before I go over my implementation I am going to introduce some terms.

A floating point number is made of three bit fields: the sign field, exponent field, and mantissa field. The sign field is always one bit irrespective of the type. In contrast the exponent field and mantissa field vary with the type.

For example, for double precision, the mantissa field is 52 bits, and the exponent field is 11 bits. The maximum exponent of a double precision is 1023 and minimum exponent is -1022. You will see how these maximum and minimum exponents are used in a second.

On the other hand, for single precision, the mantissa field is 23 bits, and the exponent field is 8 bits. The maximum exponent is 127 and the minimum exponent is -126.

### The implementation

First, we need to get the bit representation of the double precision inputs and compute the sign bit of the result.

```c++
 auto x_bits = fputil::FPBits<double>(x);

 auto y_bits = fputil::FPBits<double>(y);

 auto output_sign = (x_bits.sign() != y_bits.sign()) ? Sign::NEG : Sign::POS;
```

Here, 

```c++
fputil::FPBits
```

Is internal LLVM code that can help you manipulate bits.


Next, we have the following that checks for special values like *NaNs* and *infinities*:

```c++
if (LIBC_UNLIKELY(x_bits.is_inf_or_nan() || y_bits.is_inf_or_nan() ||
                	x_bits.is_zero() || y_bits.is_zero())) {
	if (x_bits.is_nan())
  	return static_cast<float>(x);
	if (y_bits.is_nan())
  	return static_cast<float>(y);
	if (x_bits.is_inf())
  	return y_bits.is_zero()
             	? fputil::FPBits<float>::quiet_nan().get_val()
             	: fputil::FPBits<float>::inf(output_sign).get_val();
	if (y_bits.is_inf())
  	return x_bits.is_zero()
             	? fputil::FPBits<float>::quiet_nan().get_val()
             	: fputil::FPBits<float>::inf(output_sign).get_val();
	// Now either x or y is zero, and the other one is finite.
	return fputil::FPBits<float>::zero(output_sign).get_val();
  }
```

After that, you need to extract the mantissa of the inputs and extract the exponents and then compute the leading zeros of the mantissas. Why do we compute the leading zeros. We do this to normalize the inputs. After the following lines of code is executed the highest bits of *mx* and *my* are both 1. As a result, when we compute the product of the mantissas the leading one of the full product will be either the highest bit or the second highest bit. Then in the next step we know we need to shift by some amount. If we had not shifted the zeros of *mx* and *my* the leading one could be anywhere.

```c++

  uint64_t mx, my;

  // Get mantissa and append the hidden bit if needed.
  mx = x_bits.get_explicit_mantissa();
  my = y_bits.get_explicit_mantissa();

  // Get the corresponding biased exponent.
  int ex = x_bits.get_explicit_exponent();
  int ey = y_bits.get_explicit_exponent();

  // Count the number of leading zeros of the explicit mantissas.
  int nx = cpp::countl_zero(mx);
  int ny = cpp::countl_zero(my);
  // Shift the leading 1 bit to the most significant bit.
  mx <<= nx;
  my <<= ny;
```

If *x* is normal then *mx’s* leading bit is at the 53rd bit. This associated with the fact that the mantissa field of a double precision is 53 bits long. But since we shifted the leading bit to the 64th bit for normalization we need to shift it left by 11 bits.

If x is normal nothing changes because the leading bit is not lower than the 53rd bit. So, if its normal, *ex* represents the real value of the leading 1 bit. But if x is denormal then the leading 1 bit is lower than the 53rd bit so it does not represent  the real value of the leading bit of *mx* anymore.

So, we adjust the exponent accordingly:

```c++
// Adjust exponent accordingly: If x or y are normal, we will only need to
  // shift by (exponent length + sign bit = 11 bits. If x or y are denormal, we
  // will need to shift more than 11 bits.
  ex -= (nx - 11);
  ey -= (ny - 11);
```

Now, we need to prepare the actual computation of the product’s mantissa.

For that, since the product of two 64 bit numbers is a 128 bit number we need to store the highest 64 bits and the lowest 64 bits in two variables. And we need a way to detect if the product is normal is denormal:

```c++
UInt128 product = static_cast<UInt128>(mx) * static_cast<UInt128>(my);
  int32_t dm1;
  uint64_t highs, lows;
  uint64_t g, hight, lowt;
  uint32_t m;
  uint32_t b;
  int c;

  highs = static_cast<uint64_t>(product >> 64); // top highest 64 bit of the product
  c = static_cast<int>(highs >= 0x8000000000000000); // denormal? Or normal?
  lows = static_cast<uint64_t>(product); // lowest 64 bits of the product

  lowt = (lows != 0); // sticky bit which will be used for rounding

  dm1 = ex + ey + c + fputil::FPBits<float>::EXP_BIAS; // biased exponent of the product

```

In the textbook “Handbook of Floating Point Arithmetic” the reference implementation was for single precision:

```c++
float fmul(double x, double y);
```

So the formula for *dm1* was different. It needed to be changed because since the result is a 32 bit number and hence the exponent field is 8 bits, *dm1* needs to be a number that is within the range of what an 8 bit number can represent.

And now we need to actually round the and get the products mantissa.

When a number underflows or overflows we need to wrap around by returning the highest value that a given n-bit number can represent. Well, similarly, floating point multiplication overflows when $$ dm1 \gt 255 $$ because *dm1* is 8 bits and an 8 bit number has a range of up to 255.

If $$ dm1 \lt 0 $$ then the value is denormal.

If *dm1* is 0 the mantissa needs to be right shifted by one. If it is one it needs to be right shifted by two.

So, if for the normal case you have:

```c++
m = static_cast<uint32_t>(highs >> (39 + c));
```

Then for the denormal case you have:
```c++
int m_shift = 40 + c - dm1;
int g_shift = m_shift - 1;
int h_shift = 64 - g_shift;
m = (m_shift >= 64) ? 0 : static_cast<uint32_t>(highs >> m_shift);
```

Compared to the normal case, the extra shift for the denormal case is $$ 1 - dm1 $$ – i.e., $$ m_shift =  39 + c + 1 - dm1 = 40 + c - dm1 $$.

To get the round bit, which will be used to round to all rounding modes, you shift by what you shifted *highs* minus one.

We manipulate *highs* only because when we normalized the inputs the leading 1 bit will be in the top most bits so you only need to shift *highs* to get the products mantissa *m*.

The *h_shift* has to do with clearing the mantissa and round bit so that only sticky bits are left.

In contrast for the normal case, you have this:

```c++
 m = static_cast<uint32_t>(highs >> (39 + c));
 g = (highs >> (38 + c)) & 1;
 hight = (highs << (26 - c)) != 0;
```

So, *highs* is 64 bits. In the normal case we know that the leading 1 bit is either at the 63rd or 64th bit. We want to shift that down to bit 24 to get the product's mantissa because the mantissa bit field of a single precision number is 24 bits.

If $$ c = 0 $$ if the leading 1 is at bit 63. If $$ c = 1 $$ then the leading bit is at bit 64.

Here is the code for the above:

```c++
  int round_mode = fputil::quick_get_round();
  if (dm1 >= 255) {
	if ((round_mode == FE_TOWARDZERO) ||
    	(round_mode == FE_UPWARD && output_sign.is_neg()) ||
    	(round_mode == FE_DOWNWARD && output_sign.is_pos())) {
  	return fputil::FPBits<float>::max_normal(output_sign).get_val();
	}
	return fputil::FPBits<float>::inf().get_val();
  } else if (dm1 <= 0) {

	int m_shift = 40 + c - dm1;
	int g_shift = m_shift - 1;
	int h_shift = 64 - g_shift;
	m = (m_shift >= 64) ? 0 : static_cast<uint32_t>(highs >> m_shift);

	g = g_shift >= 64 ? 0 : (highs >> g_shift) & 1;
	hight = h_shift >= 64 ? highs : (highs << h_shift) != 0;

	dm1 = 0;
  } else {
	m = static_cast<uint32_t>(highs >> (39 + c));
	g = (highs >> (38 + c)) & 1;
	hight = (highs << (26 - c)) != 0;
  }

  if (round_mode == FE_TONEAREST) {
	b = g && ((hight && lowt) || ((m & 1) != 0));
  } else if ((output_sign.is_neg() && round_mode == FE_DOWNWARD) ||
         	(output_sign.is_pos() && round_mode == FE_UPWARD)) {
	b = (g == 0 && (hight && lowt) == 0) ? 0 : 1;
  } else {
	b = 0;
  }
```

Finally, we compute the exponent and cast the result to float:

```c++
uint32_t exp16 = (dm1 << 23);

uint32_t m2 = m & fputil::FPBits<float>::FRACTION_MASK;

uint32_t result = (exp16 + m2) + b;

auto result_bits = fputil::FPBits<float>(result);
result_bits.set_sign(output_sign);
return result_bits.get_val();
```

And there you have it: the implementation of:

```c++
float fmul(double x, double y);
```

I enjoyed doing this.

### Acknowledgements

I want to thank Tue, the libc maintainer, for explaining a lot of this to me.

### References

Handbook of Floating Point Arithmetic
