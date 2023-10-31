---
layout: post
title: The essence of real analysis
author: Job Hernandez Lara
---

*version*: 0.9.0, draft


### Real Numbers

 Dr. Hardy claimed in his book "A Mathematical Apology" that the theorem that states that $\sqrt{2}$ is irrational is a beautiful theorem.

**Theorem**
There is no rational number whose square is 2.
*Proof* Suppose there is a rational number $$ n $ whose square is 2. Then $$ n $$ can be represented as $$ p/q $$. Consequently, this means that $$ (p/q)^2 = 2 $$. We assume that $$ p $$ and $$ q $$ have no common factors; consequently, $$ p^2 = 2q^2 $$. This means that  $$ p^2 $$ is even so $$ p $$ is even. Given this we can rewrite $$ p $$ as $$ 2r $$. If we then substitute $$ p $$ for $$ 2r $$ we end up with: $$ 2r^2 = q^2 $$ which means that $$ q $$ is even which in turn means that this can be reduced further which is a contradiction because there were suppose to be no common factors.

To understand the main concepts of analysis, namely, continuity, convergence, differentiation and integration we must think hard about the real numbers. As a result, we start with the natural numbers which are denoted as $$ \mathbb{N} = \{ 1,2,3,4, \dots \} $$. Leopold Kronecker said 
>The natural numbers are the work of God. The rest is the work of mankind.

Given the natural numbers we can perform addition but we must extend our system to the integers, denoted as $$ \mathbb{Z} = \{\dots,-3,-2,-1,0,1,2,3, \dots \} $$, if we want to include the number zero (additive identity) and the additive inverses needed to define subtraction. Now, how do we extend our system to be able to define multiplication and division? The number one acts as a multiplicative identity but to be able define division we must extend our system to the rational numbers denoted as  $$ \mathbb{Q} $$ = {all fractions p/q where p and q are integers where q is not 0}. The set of rational numbers is a field because we can perform addition, subtraction, multiplication, and division. The set of natural numbers and the set of integers are not fields. Imagine the number line of the rational numbers. Then we will notice that there are holes in this number line when we consider numbers such as the square root of 2 or square root of 3, and so on. As a result of this observation, mathematicians extended the set of the rational numbers to the real numbers. Given the real numbers wherever there is a hole in the number line we place an irrational number.

A set $$ A \in \mathbb{R} $$ is bounded from above if there exists a  $$ b \in \mathbb{R} $$ such that $$ a \le b $$ for all $$ a $$ in $$ A $$. Similarly, a set $$ A $$ is bounded from below if there exists an $$ l $$ such that $$ l \le a $$ for all $$ a \in A $$. A real number $$ s $$ is the **least upper bound** for a set $$ A \in R $$ if it meets two conditions: 1) s is an upper bound for A; 2) let b be any upper bound for A then $$ s \le b $$. There isn’t a least upper bound in the rational numbers because you can always reduce a given rational number; but there is a least upper bound in a subset of the real numbers. The least upper bound is denoted as $$ s = sup A $$.

*lemma*
Assume s in R is an upper bound for a set $A \in R$. Then $s = sup A$ if and only if for  every choice $\epsilon > 0$ there exists an element $a \in A$ such that $s - \epsilon < a$. Here is a quote from the textbook:
	
> Given that s is an upper bound is the least upper bound if and only if any number smaller than s is not  an upper bound.
	

We will now talk about how $$ \mathbb{Q} $$ is dense in $$ \mathbb{R} $$.
*Theorem*
Archimedean Property:
* Given any number $$ x \in \mathbb{R} $$, there exists an $$ n \in \mathbb{N} $$ such that $$ n > x $$.
* Given any real number $$ y>0 $$ there exists an $$ n \in \mathbb{N} $$ satisfying $$ 1/n < y $$.


*Proof*. Part (i) states that $$ \mathbb{N} $$ is not bounded. Assume that $$ \mathbb{N} $$ is bounded. Then by the Axiom of Completeness $$ \mathbb{N} $$ should have a least upper bound so we can set $$ \alpha = sup \mathbb{N} $$. Now if we consider $$ \alpha - 1 $$ then we no longer have an upper bound given lemma 2. Therefore there exists an $$ n \in \mathbb{N} $$ satisfying $$ \alpha - 1 < n $$ but this is to say $$ \alpha < n + 1 $$ and as a result $$ n+1 \in \mathbb{N} $$ which contradicts the fact that $$ \alpha $$ is supposed to be an upper bound for $$ \mathbb{N} $$. Part (ii) follows from (i) by letting $$ x=1/y $$.
### Sequences and Series

#### Limit of a Sequence
Firstly, we need to understand what a *sequence* is. A sequence is a function whose domain is $$ \mathbb{N} $$. In other words, given a function $$ f: \mathbb{N} \to \mathbb{R} $$ then $$ f(n) $$ is the $$ nth $$ term of the sequence.
Now that we have defined what a $$ \textit{sequence} $$ is  we are ready to think about $$ \textit{convergence} $$ of a sequence. A sequence $$ a_{n} $$ converges to $$ \textit{a} $$ whenever the following proposition is true: if $$ n \ge N $$ then $$ \left|a_{n}-a\right| $$ $$ < \epsilon $$ where $$ \epsilon $$ is a positive number and there exists $$ N \in \mathbf{N} $$. We denote $$ \left(a_{n}\right) $$ converges to $$ a $$ as $$ \lim_{a_{n}} = a $$. Intuitively, this means that there is an interval $$ \left(a-\epsilon, a+\epsilon\right) $$ centered on $$ a $$ where $$ N $$ is the first entry into this interval.

### The Algebraic and Order Limit Theorems

A sequence is bounded if every term of $$ x_{n} $$ is contained in an interval $$ \left[-M, M\right] $$ or more formally: Assuming there exists a number $$ M > 0 $$ such that $$ \left|x_{n}\right| \le M $$ for all $n \in \mathbf{N}$ then the sequence $x_{n}$ is bounded. The theorem that follows from this definition is the following:
*Theorem*
*Every convergent sequence is bounded.*
*Proof.* Suppose $$ \left(x_{n}\right) $$ converges to a limit $$ \textit{l} $$. And furthermore suppose $$ \epsilon = 1 $$ and by the definition of convergence then we know that $$ x_{n} $$ is in the interval
$$ \left(l-\epsilon, l+\epsilon\right) $$ so there exists an $$ N \in \mathbf{N} $$ such that $$ n \ge N $$. So, we can conclude that $$ \left|x_{n}\right| < \left|l\right| + 1 $$ for all $$ n \ge N $$.

How do sequences behave with respect to the operations of addition, multiplication, division and order?

*Thereom*
Algebraic Limit Theorem.
Let $$ \lim_{a_{n}} = a $$ and $$ \lim_{b_{n}} = b $$. Then,
* $$ \lim_{ca_{n}} = ca $$, for all $$ c \in \mathbb{R} $$;
* $$ \lim_{a_{n} + b_{n}} = a + b $$;
* $$ \lim_{a_{n} * b_{n}} = ab $$;
* $$ \lim_{a_{n}/b_{n}} = a/b, provided b is not 0 $$.

### Continuity
#### Continuous Functions
In short, a function $$ f $$ is continuous at $$ c \in A $$ if $$ \lim_{x\to c} f\left(x\right) = f\left(c\right) $$. This means that $$ f\left(x\right) $$ converges to $$ f\left(c\right) $$ by definition.

*Theorem*
Characterization of continuity.
Let $$ f \colon A \to \mathbb{R} $$ and let $$ c \in A $$. The function $$ f $$ is continuous at $$ c $$ if and only if anyone of the following three conditions is met:

* For all $$ \epsilon > 0 $$ there exists a $$ \delta > 0 $$ such that $$ \left|x-c\right| < \delta $$ and $$ x \in A $$ implies $$ \left|f\left(x\right) - f\left(c\right)\right| < \epsilon $$.
* For all $$ V_{\epsilon}\left(f\left(c\right)\right) $$ there exists a $$ V_{\delta}\left(c\right) $$ with the property that $$ x \in V_{\delta}\left(c\right) and x \in A $$ implies $$ f\left(x\right) \in V_{\epsilon}\left(f\left(c\right)\right) $$.
*  if $$ x_{n} \to c $$ with $$ x_{n} \in A $$ then $$ f\left(x_{n}\right) \to f\left(c\right). $$

If $$ c $$ is the limit point of $$ A $$ then the above conditions are equivalent to  $$ \lim_{x\to c} f\left(x\right) = f\left(c\right) $$.
    
*Theorem*
Algebraic continuity theorem

Assume $$ f \colon A \to \mathbb{R} $$ and $$ g \colon A \to \mathbb{R} $$ are continuous at point $$ c \in A $$. Then,

* kf\left(x\right)$ is continuous at $$ c $$ for all $$ k \in \mathbb{R} $$;
*  $$ f\left(x\right) + g\left(x\right) $$ is continuous at point $$ c $$;
* $$ f\left(x\right)g\left(x\right) $$ is continuous at point $$ c $$;
* $$ f\left(x\right) / g\left(x\right) $$ is continuous at point $$ c $$, provided the quotient is defined.
	


