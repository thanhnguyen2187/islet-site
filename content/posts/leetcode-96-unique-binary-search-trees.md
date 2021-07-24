---
title: "Leetcode 96: Unique Binary Search Trees"
date: 2021-07-24T10:40:07+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - reference
tags:
  - leetcode
math: true
---

## Description

You can read the original description
[here](https://leetcode.com/problems/unique-binary-search-trees/), but I will
summarize anyway.

| `n` | `result` |
| -   | -        |
| `1` | `1`      |
| `2` | `2`      |
| `3` | `5`      |
| `4` | `14`     |
| `5` | `42`     |

With `n = 1`, `result = 1` since we only can create a binary tree with a singular node:

```
(1)
```

With `n = 2`, `result = 2`:

```
  (1)  (2)
  /      \
(2)      (1)
```

With `n = 3`, `result = 5`:

```
(1)      (1)      (2)      (3)      (3)     
  \        \      / \      /        /       
  (2)      (3)  (1) (3)  (1)      (2)       
    \      /               \      /   
    (3)  (2)               (2)  (1)   
```

## Heart of The Problem

We will see a pattern, if we count the empty set `{}` as one way of arranging.

With `n = 0`, and `n = 1`, `result = 1`:

```
({})
(1)
```

Let us call the result function as `f(n)`, with `n` is the number of nodes, we
have base cases:

```py
f(0) = 1
f(1) = 1
```

Let us plug the symbol into `n = 2`, and think of every thing as "root", and
"children set":

```
root | children sets
 |   |  |
 v   |  v

     ┌──({2})
(1)──┤
     └──({})

     ┌──({1})
(2)──┤
     └──({})
```

We then can calculate `f(2)` using `f(1)` and `f(0)`:

```py
f(2) = sum(
    (
        f(0) * f(1),
        f(1) * f(0),
    )
)
```

Let us look at `f(3)` in the same way:

```
     ┌──({2, 3})
(1)──┤
     └──({})

     ┌──({3})
(2)──┤
     └──({1})

     ┌──({})
(3)──┤
     └──({1, 2})
```

The calculation then becomes:

```py
f(3) = sum(
    (
        f(0) * f(2),
        f(1) * f(1),
        f(2) * f(0),
    )
)
```

We arrive at a formula like this:

$$ f(n) = \sum_{i = 0}^{n - 1}{f(i) * f(n - 1 - i)} $$

The formula above is one way to represent [Catalan Numbers](https://brilliant.org/wiki/catalan-numbers/),

> which is a sequence of positive integers that appear in many counting problems
> in combinatorics.

## Implementation

The implementation is trivial and can be found here.

## Conclusion

The problem is not too hard after we understand the pattern behind, and think of
it as a counting problem, not a generating problem. I had the wrong
implementation at first:

- Generate every combinations
- Fit the combinations into binary trees
- Put the binary trees into a set
- Return the number of elements in the set

My old approach got a "TLE", since `10!` is already a big number, and the limit
is `19!`.
