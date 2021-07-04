---
title: "Leetcode 363: Max Sum of Rectangle No Larger Than K"
date: 2021-07-04T20:07:21+07:00
draft: true
toc: true
images:
categories:
  - explanation
  - reference
tags:
  - leetcode
---

I struggled with the problem for a while, and reading a lot of explanations did
help, but I thought that it would be great if there is something better. The
post is a "reference" in the sense that I am going to clarify the example codes.

## Problem

You can read the original version
[here](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/),
but I will summarize it in term of data like this:

```python
matrix = [
    [1, 0,  1],
    [0, -2, 3],
]
result = 2
```

The `result` is `2` since the selected submatrix is:

```python
submatrix = [
    [0,  1],
    [-2, 3],
]
```

## First Approach

At first, I thought that dynamic programming is enough to solve the case. We
only need to:
- find the largest sum possible,
- and compare it to `k`,
- and then find the minimum distance.

```python
```

## Subproblems

## Solving Subproblem 1,

## Subproblem 2:

## Subproblem 3:

## 
