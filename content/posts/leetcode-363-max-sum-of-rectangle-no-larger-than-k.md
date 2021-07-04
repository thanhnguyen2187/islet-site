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

The constraints are:

```python
m == len(matrix)
n == len(matrix[i])
1 <= m, n <= 100
-100 <= matrix[i][j] <= 100
-10**5 <= k <= 10**5
```

It means anything greater than `n**2` (or `n^2`) is a no, since we are dealing
with `10**4` elements in our matrix.

## First Approach: Dynamic Programming

For me in a different timeline, if I did not even know what dynamic programming
is, please do some research and try implementing some problems myself. Be
understanding this "core" principle however:

> Dynamic programming means you:
> - Memorize the "optimal" solution,
> - and calculate the next result base on the previous "optimal" solution.

At first, I thought that dynamic programming is enough to solve the case. I only
need to:
- find the largest sum possible,
- and compare it to `k`,
- and then find the minimum distance.

Let us walk through how do I "wrongly" solved the original example. The input
look like:

```python
matrix = [
    [1, 0,  1],
    [0, -2, 3],
]
```

I would need a table to store what is the maximum sum I can find for each
element in the matrix, if I _must_ use the element in my final result. I would
have four cases to consider:
- It can be built from the left-side cells
- It can be built from the above cells
- It can be built from the left-side _and_ above cells
- It can be built from itself only

The result table should initially look like this:

|      |      |      |
|------|------|------|
| -inf | -inf | -inf |
| -inf | -inf | -inf |

To make the work easier, I would pad a blank row and column in my table:

|   |      |      |      |
|---|------|------|------|
| 0 | 0    | 0    | 0    |
| 0 | -inf | -inf | -inf |
| 0 | -inf | -inf | -inf |

The table would be filled like this:

|   |   |      |      |
| - | - | -    | -    |
| 0 | 0 | 0    | 0    |
| 0 | 1 | 1    | 2    |
| 0 | 1 | 0 | -inf |

In code, it would look like this wrapped by a for-loop:

```python
current_element = matrix[row_index][column_index]
left_cell = cells[row_index][column_index - 1]
above_cell = cells[row_index - 1][column_index]

cells[row_index][column_index] = max(
    left_cell + current_element,
    above_cell + current_element,
    left_cell + above_cell + current_element,
    current_element,
)
```

```python
```

## Subproblems

## Solving Subproblem 1,

## Subproblem 2:

## Subproblem 3:

##
