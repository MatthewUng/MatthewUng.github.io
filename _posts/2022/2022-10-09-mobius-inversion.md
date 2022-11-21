---
layout: post
title: Mobius Inversion
date: 2022-10-09
---

# Mobius Inversion
Our goal for this section is to come to understand
$$ \phi(n) = \sum_{d | n} \mu(d)\frac{n}{d} $$ 
to which we would need to understand: What is \\(\phi\\)? What is \\(\mu\\)?


## Multiplicative Arithmetic Functions and \\(\phi\\)
A **multiplicative arithmetic** function is a function \\(f\\) over the natural number 
for which \\(f(ab) = f(a)f(b)\\) whenever \\(gcd(a, b) = 1\\).

The most famous multiplicative function is probably \\(\phi(n)\\), 
which is defined as the number of numbers less than and coprime than \\(n\\)
For instance, \\(\phi(17)=16\\) since 17 is prime and every number less than it is coprime.  
Likewise, \\(\phi(30) = 8\\).  The numbers coprime with 30 are 1,7,11,13,17,19,23,29 for a total of 8.

Prime values of \\(\phi\\) are easily calculable:  \\(\phi(p) = p-1\\) since every 
number less than it can not possibly have \\(p\\) as a divisor.
For powers of a prime \\(phi(p^k) = p^k - p^{k-1}\\) since the only numbers 
that can share a divisor with \\(p^k\\) are the multiples of \\(p\\).
Finally, showing that \\(\phi\\) is multiplicative means showing that 
\\(\phi(ab) = \phi(a)\phi(b)\\) when \\(gcd(a,b) = 1\\).  
However, this is immediate as the Chinese Remainder Theorem establishes a bijection 
between the numbers coprime with \\(ab\\) and the product of the numbers coprime 





#### References
1. [https://en.wikipedia.org/wiki/Fixed-point_combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator)
