---
title: "Leetcode 363: Max Sum of Rectangle No Larger Than K"
date: 2021-07-04T20:07:21+07:00
draft: false
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
post is a "reference" in the sense that I am going to clarify some sample codes.

## Description

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

`result` is `2` since the selected submatrix is:

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

It means an obvious brute force solutions is a no, and we should find something
else.

## Heart of The Problem

At heart, the problem can be splitted into two subproblems:

1. Find the maximal sum of contiguous subarrays that does not exceed `k`
2. Apply the pattern to a matrix (2D array)

In my own words, a "contiguous subarray" of an array is a _subarray that
consists an arbitrary number of same-ordered elements_. Let us look at an
example to understand it easier:

```python
L  = [1, 2, 3, 4, 5, 6]
L1 = [1, 2, 3]  # L1 is a contiguous subarray of L
L2 = [3, 4, 5]  # L2 is a contiguous subarray of L, too
L3 = [4, 6, 7]  # L3 is **not a contiguous subarray of L**
L4 = [6, 5, 4]  # L4 is **not a contiguous subarray of L**, either
```

## Subproblem 1: Find the maximal sum of contiguous subarray...

The problem seems hard to be solved without brute force. Nevertheless, if we are
clever enough about data transformation, the subproblem can be solved elegantly,
however.

Let us be familiar with a definition called "prefix sum": it means we "prefix"
our elements with the ones behind it.

```python
L = [1, 2, 3, 4, 5, 6]
prefix_sums = [
    0,  # (no elements)
    1,  # 1
    3,  # 1 + 2
    6,  # 1 + 2 + 3
    10, # 1 + 2 + 3 + 4
    15, # 1 + 2 + 3 + 4 + 5
    21, # ...
]
```

It applies here if we realize that we can _present sum of any contiguous
subarray as the subtraction of prefix sums_.

```python
L = [1, 2, 3, 4, 5, 6]
prefix_sums = [
    0,  # (no elements)
    1,  # 1
    3,  # 1 + 2
    6,  # 1 + 2 + 3
    10, # 1 + 2 + 3 + 4
    15, # 1 + 2 + 3 + 4 + 5
    21, # ...
]

sum_first_to_fourth = prefix_sums[4] - prefix_sums[0]
# = (1 + 2 + 3 + 4) - 0
# = 10

sum_second_to_fifth = prefix_sums[5] - prefix_sums[1]
# = (1 + 2 + 3 + 4 + 5) - (1)
# = 2 + 3 + 4 + 5
# = 14
```

The searching problem thus become a more approachable searching problem:
- Pick the "right-side" prefix sum, and then
- Pick the "left-side" prefix sum.
- If the right element minus the left element is less than `k`,
- Pick the substraction as the new found max.

A naive implementation looks like this:

```python
from itertools import accumulate

k = 4
L = [...]
prefixed_L = [...]

maximal_sum = float("-inf")
for right_index in range(0, len(L)),
    for left_index in range(right_index + 1, len(L)):
        right_sum = prefixed_L[right_index]
        left_sum = prefixed_L[left_index]
        current_sum = right_sum - left_sum
        maximal_sum = (
            current_sum
            if (
                current_sum <= k and
                current_sum > maximal_sum
            )
            else maximal_sum
        )
```

The implementation can be improved by using a sorted array to store the
encountered sums, and find a suitable left sum from it.

```python
from itertools import accumulate
from bisect import (
    bisect_left,
    insort,
)

k = 4
L = [...]
prefixed_L = accumulate(L)

maximal_sum = float("-inf")
encountered_sums = [0, float("inf")]
# 0 and +inf are used here to mitigate our efforts
# at assigning the right value to maximal_sum

for right_sum in prefixed_L:
    target = right_sum - k
    left_sum_index = bisect_left(
        a=prefixed_L,
        x=target,
    )
    left_sum = prefixed_L[left_sum_index]
    current_sum = right_sum - left_sum

    maximal_sum = max(maximal_sum, current_sum)
    insort(
        a=encountered_sums,
        x=right_sum,
    )
```

## Subproblem 2: Apply the subproblem 1's solving method to a matrix (2D array)

At first, I could hardly understand how and why must we apply the method. It
took me a while to grasp it. I will try to walk through the process step by step
here.

Let us have a look at another example:

```python
matrix = [
    [11, 2, -3, 6],
    [3,  4, -4, 0],
    [-9, 7, 9,  1],
    [6,  2, 7,  8],
]
```

We choose a random subrectangle:

```python
matrix = [
    [11, 2, -3],
    [3,  4, -4],
    [-9, 7, 9],
]
```

The sum can be calculated easily: `11 + 2 + 3 + 4 + -9 + 7 + -3 + -4 + 9 = 20`

If we "squeeze" the matrix into one row:

```python
squeezed_row = [5, 13, 2] 
```

The sum is still the same: `5 + 13 + 2 = 20`

It if the sum is all we care, we can represent any subrectangle as an equivalent
row:

```python
subrectangle_1 = [
    [-9, 7, 9,  1],
    [6,  2, 7,  8],
]
row_1 = [-3, 9, 16, 9]

subrectangle_2 = [
    [11, 2],
    [3,  4],
]
row_2 = [14, 6]
```

`row_1`, `row_2`, and any `row_n` is a normal array. It means we can apply the
prefix sum method on them, and then find the maximal sum that is no larger than
`k`.

## Implementation

Our final implementation seems straight forward once we tackled the two
subproblems. The steps are:

- Turn the rows of the matrix into prefixed ones
- Choose the full-width subrectangles, and find the maximal sum of each 
- Compare and choose the greatest maximal sum

Sample code can be found
[here](https://github.com/thanhnguyen2187/random-problem-solving/blob/master/leetcode/n0363_max_sum_of_rectangle_no_larger_than_k.py).

## Conclusion

It took me quite the time understanding, and implementing the problem. The label
"hard" was there not for a random reason, so please take your time. Things will
"click" after a while.
