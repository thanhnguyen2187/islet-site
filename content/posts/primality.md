---
title: "On Primality"
date: 2021-07-05T23:53:24+07:00
draft: true
toc: true
tags:
categories:
- explanation
- tutorial
- how-to-guide
- reference
math: true
---

I encountered the topic again and again over the years, but did not take my time
deep diving into it. SICP mentioned it again, and I was like: "Okay. It must be
now."

In the post, I will try to "build" the stuff up, from simple to complex, with
some sample code in Python. At first, I thought that I would try implementing
the stuff in Lisp, but I could not get comfortable enough at the language the
time. My target is to understand the methods, and let them be references for my
reader (or at least my future self).

## Fundamentals

From [Wikipedia]():

> A prime number (or a prime) is a:
> - natural number
> - greater than 1 [...]
> - not a product of two smaller natural numbers

Or [Khan Academy]():

> Prime numbers are numbers that have only 2 factors: 1 and themselves.
> For example, the first 5 prime numbers are 2, 3, 5, 7, and 11.

$2$ is a prime number, while $4$ is not, since $4 = 2*2$.

$4$ is a "composite number".

> A natural number greater than 1 that is not prime is called a composite
> number.

## A Simple Implementation

Let us jump into the simplest implementation:

```python
def is_prime_naive(x: int) -> bool:
    return (
        x > 1 and
        not any(
            x % factor == 0
            for factor in range(2, x)
        )
    )
```

## An Improved Implementation

The above method seems... inefficient, if we notice these properties:

- Apart from $2$, other primes are odd numbers
- We mostly do not need to check the factors in range
  $[\lceil\frac{x}{2}\rceil,x - 1]$ (in case you are wondering, $[a, b]$ means
  it is a range `>= a` and `<= b`, and $\lceil x \rceil$ means `ceil(x)`)
- Or if we squint it hard enough, we can swap $\lceil\frac{x}{2}\rceil$ with
  $\lceil\sqrt x \rceil + 1$

Anyway, an improved version can look like this:

```python
from math import (
    isqrt,
)

def is_prime_better(x: int) -> bool:
    return (
        x > 1 and
        x % 2 != 0 and
        not any(
            x % factor == 0
            for factor in range(3, isqrt(x), 2)
        )
    )
```

Notice that we are using `range(3, isqrt(x), 2)` here, to find the odd factors
start from $3$.

## Fermat Primality Test

Let us quote [Wikipedia](https://en.wikipedia.org/wiki/Fermat_primality_test) again:

> Fermat's little theorem states that if p is prime and a is not divisible by p, then
> $$ a^{p - 1} \equiv 1 \pmod{p} $$

## More "Fundamentals"

## A Better Method

## The "Best" Method
