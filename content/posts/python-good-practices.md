---
title: "Python \"Good Practices\""
date: 2021-08-20T23:29:53+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - how-to-guide
tags:
  - python
---

I have been dabbling in Python and its ecosystem for a while. I wrote and looked
at enough code bases to have a sense of good style, so... trust me on this, I
guess. I have taken by heart the motto "no silver bullet"; therefore I named the
post "Good Pracices" instead of "Best Practices". 

## Type Hinting

Python's type hinting actually does no work, if we do not use mypy or something
to enforce the typing, but using it just for the completion seems really nice.

At first, I considered using a "basic" example like this:

```python
# untyped
def f(a, b):
    return a + b

# typed
def f(
    a: int,
    b: int,
) -> int:
    return a + b
```

But let us look at something more "untrivial". This is a class to store the data
of a Sudoku board (a part of my solution to [Leetcode 36. Valid
Sudoku](https://leetcode.com/problems/valid-sudoku/)):

```python
class Position(NamedTuple):
    x: int
    y: int


class Board:

    rows: DefaultDict[int, DefaultDict[str, int]]
    # rows[n] represents how many particular character appeared in the (n+1)th row

    columns: DefaultDict[int, DefaultDict[str, int]]
    # columns[n] represents how many particular character appeared in the (n+1)th column

    sub_boards: DefaultDict[Position, DefaultDict[str, int]]
    # 0 <= i, j <= 2
    # sub_board[i][j] represents how many particular character appear in the i-th row and j-th column

    def is_valid(self) -> bool:

        def check(dict_: DefaultDict[Any, Dict]) -> bool:
            return all(
                value <= 1
                for values in dict_.values()
                for value in values.values()
            )

        valid_rows = check(dict_=self.rows)
        valid_columns = check(dict_=self.columns)
        valid_sub_boards = check(dict_=self.sub_boards)

        return (
            valid_rows and
            valid_columns and
            valid_sub_boards
        )

    def add_cell(
        self,
        position: Position,
        value: str,
    ):
        if value != ".":
            self.rows[position.x][value] += 1
            self.columns[position.y][value] += 1

            sub_board_position = Position(
                x=position.x // 3,
                y=position.y // 3,
            )
            self.sub_boards[sub_board_position][value] += 1

    def __init__(self):
        self.rows = defaultdict(lambda: defaultdict(lambda: 0))
        self.columns = defaultdict(lambda: defaultdict(lambda: 0))
        self.sub_boards = defaultdict(lambda: defaultdict(lambda: 0))
```

I would argue that without `DefaultDict[...]` and stuff, writing and reading
code would feel like a land mine, waiting to be stepped on.

My guess is people complain that large code bases in dynamic programming
language becomes unmaintainable since they can hardly grasp the "complex" data
transformations and interactions. Plus, coding without code completion is more
prone to typos and misguided guess works. They are all valid reasons.

![pycharm-type-hinting](../images/python-good-practices-1.gif)

Python's type hinting partially solve the problems. It does not enforce
correctness unless you strictly requires it, but it does lessen the cognitive
load for you (and your IDE/editor).

## Generator

I planned to write about list list comprehension first, but then realized that
the topic would not make sense without explaining generator beforehands. In
some ways, Python's generator is similar to SICP's definition of "stream", and
Observable in RxJS.

In its simplest form, a generator goes with `yield`, and look like this:

```python
def f(x):
    yield x

generator = f(3)
# <generator object f at 0x7f933f1ac938>
next(generator)
# 4
next(generator)
# StopIteration

def g():
    yield 1
    yield 2
    yield 3

generator_2 = g()
# <generator object g at 0x7f933f1ac990>
list(generator_2)
# [1 2 3]
```

A generator then is something you can pass into `next` to get a value. You
understood that and started thinking: what is the point of using it? Let me goes
through a more complex sample:

```python
class Solution:

    def natural_numbers(self) -> Iterator[int]:
        current = 0
        while True:
            current += 1
            yield current

    def firstMissingPositive(self, nums: List[int]) -> int:
        existed_numbers = {
            num
            for num in nums
        }

        iterator = self.natural_numbers()
        while (num := next(iterator)) in existed_numbers:
            pass

        return num
```

That was the solution for [Leetcode 41. First Missing
Positive](https://leetcode.com/problems/first-missing-positive/). I doubt that a
hand rolled solution can be more elegant than generator usage.

We then see the application of generator: when we need "lazy" sequences, a.k.a
things that only pass their values once, to be frugal on our memory usage, or
persists a state of the last execution.

Understanding generator then lead us to more interesting stuff: list
comprehension, and "hidden gems" within Python's standard library.

## List Comprehension

I will show my take on a seemingly trivial problem from Project Euler: add all
the natural numbers below one thousand that are multiples of 3 or 5.

The first "traditional" version would look like this:

```python
result = 0
for number in range(1001):
    if (
        number % 3 == 0 or
        number % 5 == 0
    ):
        result += number
```

The second version with list comprehension would look like this:

```python
numbers = [
    number
    for number in range(0, 1001)
    if (
        number % 3 == 0 or
        number % 5 == 0
    )
]
result = sum(numbers)
```

In the second version, I created a list of suitable numbers, and then calculate
the sum of it. For me, the second version is better, since it is more
"declarative", in other words, you "declare" the result, and let the computer
calculate it for you.

We then vaguely see the pattern of list comprehension:

```python
odd_numbers = [
    x                   # how we want to "take" an element
    for x in range(10)  # the "ritual" to point out what is the sequence
    if x % 2 == 1       # the filtering condition
]
# [1, 3, 5, 7, 9]
```

A nested for loop then can be written like this:

```python
pairs = [
    (x, y)              # how we want to "take" the elements
    for x in range(10)  # loop of the first sequence
    for y in range(10)  # loop of the second sequence
]
# [(0, 0), (0, 1), (0, 2), ..., (9, 8), (9, 9)]
```

At this point, the syntax `for ... in ...` can be used in a "normal" for loop,
but we then realize it probably has some other meanings. If we try typing `x for
x in range(10)` into Python's interpreter, wrapped in `()`:

```python
(x for x in range(10))
# <generator object <genexpr> at 0x7f933f1ac938>
```

It was a generator all along! Our explanation then can be this concise:

```python
odd_numbers = [
    x                   # `next`
    for x in range(10)  # "iterable"
    if x % 2 == 0       # condition
]
# [0, 2, 4, 6, 8]
```

What does "iterable" means is left as an exercise for the reader.

## Hidden Gems of Standard Library

After we had a good grasp of generator, we then want to write some cool stuff
using generator by ourselves:

```python
def natural_numbers() -> Iterator[int]:
    current = 0
    while True:
        current += 1
        yield current
```

Python provided them inside a package called `itertools`, however:

```python
from itertools import count
```

`count` is roughly equivalent to:

```python
def count(start=0, step=1):
    n = start
    while True:
        yield n
        n += step
```

Just using the standard library, a "cool" function that I really like to "replace"
nested loop is `product`:

```python
from itertools import product

list(
    product(
        range(3),
        range(3),
    )
)
# [(0, 0), (0, 1), (0, 2), (1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
```

Numbers are used in this case, but we can use any other kind of sequence in the
place. In case we want "strictly increased" and "unique" elements
(combinations), we can use...  `combinations` instead. `permutations` also has
its use. [Python official
documentation](https://docs.python.org/3/library/itertools.html) summarized it
better than I can:

| Examples                         | Results                                           |
| ---                              | ---                                               |
| `product('ABCD', repeat=2)`      | `AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD` |
| `permutations('ABCD', repeat=2)` | `AB AC AD BA BC BD CA CB CD DA DB DC`             |
| `combinations('ABCD', repeat=2)` | `AB AC AD BC BD CD`                               |

## Bonus: "Good" Third-party Libraries

### Version Management

![asdf-vm](../images/asdf-vm.png)

`pyenv` is probably good enough, but I still recommend `asdf`, however, since it
works with other programming languages in the same fashion. You can look at the
tool [here](https://asdf-vm.com/).

### Dependency Management

Let me summarize the way "dependency management" works in Python, using xkcd's
comic:

![xdcd-standards](../images/xdcd-standards.png)

We have `pip`, which goes with `requirements.txt` as the standard practice. We
have `virtualenv` which is the "old preferred way", and superseded by `python -m
venv`. More than that, we have `pipenv`, and `poetry`, and literally 'dephell'
which stands for dependency hell itself.

Please let me give you a single solution which seems "good enough", and save
your time on researching: go all-in with
[Poetry](https://python-poetry.org/docs/).

### Install and Run Python Applications in Isolated Environments

![pipx](https://raw.githubusercontent.com/pypa/pipx/master/pipx_demo.gif)

The tool is called `pipx`, which is similar to `npx` of `npm`. The explanation
is very well written on [their documentation](https://pypa.github.io/pipx/):

> - pipx is a tool to help you install and run end-user applications written in
>   Python. It's roughly similar to macOS's brew, JavaScript's npx, and Linux's
>   apt.
> - It's closely related to pip. In fact, it uses pip, but is focused on
>   installing and managing Python packages that can be run from the command line
>   directly as applications.
