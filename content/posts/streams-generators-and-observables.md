---
title: "Streams, Generators, and Observables"
date: 2021-10-30T04:44:43+07:00
draft: true
toc: true
images:
categories:
  - explanation
tags:
  - sicp
  - lisp
  - python
  - javascript
---

I have just finished the third chapter of SICP and learned a lot from it. The
chapter dives dive into the two way that we can use to model our programs:
objects or streams.

> We can model the world as a collection of separate, time-bound, interacting
> objects with state, or we can model the world as a single, timeless, stateless
> unity. Each view has powerful advantages, but neither view alone is complete
> satisfactory. A grand unification has yet to emerge.

Working with Python's `yield`ing, and `.subscribe`ing Observables within
JavaScript (or RxJS to be precise) in the past, combines with the recently SICP
reading on streams gave me some interesting thoughts on the concepts'
similarities.

## Streams

The core of Scheme, or Lisp's streams lies in this implementation:

```lisp
(define (delay expression)
  (lambda () expression))

(define (force expression)
  (expression))
```

If it looks... underwhelming, I have nothing else to say except... it is. Let us
see another underwhelming piece of code:

```lisp
(define (cons-stream a b)
  (cons
    a
    (delay b)))
```

A "lazy" list of natural number would looks like this:

```lisp
(define (integers-starting-from n)
  (cons-stream n (integers-starting-from (+ n 1))))

(define integers
  (integers-starting-from 1))

;; `integers` then is
;; `(cons-stream 1 (cons-stream 2 (cons-stream 3 ...)))`
;; but keep in mind that the calculations only take `1`, and leave
;; `(cons-stream 2 (cons-stream 3 ...))` intact; it only continue evaluating
;; only if we specifies so
```

It allows us to implement a "cool" and "elegant" Fibonacci sequence like this:

```lisp
(define fibs
  (cons-stream 0
               (cons-stream 1
                            (add-streams (stream-cdr fibs)
                                         fibs))))

(take-stream fibs 5)
;; (0 1 1 2 3)
```

A bank account's withdrawing transactions can be modeled as a stream of
"withdrawing numbers":

```lisp
(define (stream-withdraw balance amount-stream)
  (cons-stream balance 
               (stream-withdraw (- balance (stream-car amount-stream))
                                (stream-cdr amount-stream))))
```

## Generators

Python's generators, conceptually, are strangely similar: `yield`ing gives us a
"generator", and we have to do something with the generator to have its real
values.

Let us consider this "standard" example of Python's generator that has the same
functionalities as Scheme's `integers` above:

```python
def create_natural_numbers_generator():
    counter = 0
    while True:
        counter += 1
        yield counter
```

We use `next` to take values from the generator:

```python
generator = create_natural_numbers_generator()
generator
# <generator object create_natural_numbers_generator at 0x7fdfcd035900>

next(generator)
# 1
next(generator)
# 2
next(generator)
# 3
```

## Observables

"Reactive Programming" gives us "observables" in which instead of getting "raw
value", we assign "observers" (in other words, a group of functions) to process
each fetched value.

```ts
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

In a simple diagram, observables and observers look like this:

```
+------------+                                    +----------+
|            |                                    |          |
| Observable |---(event)----(event)----(event)--->| Observer |
|            |                                    |          |
+------------+                                    +----------+
```

Within an observer, there are "handlers" for the case we receive one, or we have
some error, or when the observable announce that it completed and we can stop
"listening":

```
+---------------------+
|                     |
| Observer            |
|                     |
                      |
---(event)--+         |
            |         |
|           v         |
| +-----------------+ |
| |                 | |
| | "next" handler  | |
| |                 | |
| +-----------------+ |
|                     |
| +-----------------+ |
| |                 | |
| | "error" handler | |
| |                 | |
| +-----------------+ |
|                     |
| +-----------------+ |
| |                 | |
| |   "complete"    | |
| |    handler      | |
| |                 | |
| +-----------------+ |
|                     |
+---------------------+

(please forgive me of the "unimaginative" and probably horrible and misleading
presentation; I hope that you get the point however)
```

## The Main Idea

For streams and generators, the idea is the same: we have some "wraps" around
our real values. The values themselves are not calculated until we explicitly
"unwrap" them.

```
+----------------+
|                |
| there might be | ---(unwrap)----> the "naked" value
| value here     |
|                |
+----------------+
```

Replace "wrap" with "delay", and "unwrap" with "force", and we get Scheme's
streams. Replace "wrap" with "yield", and "unwrap" with "next", and we get
Python's generators. We have to call the "unwrap"-ing explicitly.

For observables, the idea changes a bit.

```
+----------------+                    
|                |                    +----------+
| there might be | <--(subscribe)---- |          |
| values here    | --(push values)--> | Observer |
|                |                    |          |
+----------------+                    +----------+
```

We have to actively do the subscription. Only then the value comes to us.

The three concepts all intersect a bit on "lazy evaluation", or, using SICP's
word, "normal order", as the value cannot be used "directly", but only after a
bit of "magic".

## Applications

Please be noted that I do not include the applications of Observables here,
since they are too easily seen within... Angular (or I got too lazy with the
post).

### Memory Usage Reducing

The idea is on working with big files that you cannot load directly into memory,
generator provides a "clean" way to work with them line-by-line:

```python
def readlines_lazy(file):
    while True:
        line = file.readline()
        if line:
            yield line
        else:
            return

with open('big_file.txt') as file:
    for line in readlines_lazy(file):
        do_stuff(line)
```

### Infinite Data Structures

I am going to come up with a toy problem: take every even-indexed elements
within a list (suppose we have an array called `A`; we need `A[0]`, `A[2]`,
etc.).

There is a lot of way to solve this, but I want to present an elegant way to do
it with an infinite list of `True`s and `False`s:

```python
def create_true_false_generator():
    while True:
        yield True
        yield False

true_false_generator = create_true_false_generator()
A = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
B = filter(
    lambda _: next(true_false_generator),
    A,
)

B
# [1 3 5 7 9]
```

What I have done there, is to create an infinite list of `True`s and `False`s,
then filter the array just with that `True`s and `False`s interleaving.

### Changes Auditing

Streams is the way that we need to audit the changes that was made. Let us look
at a "standard" table presentation of our data:

```
+----+------------+
| Id | Name       |
+----+------------+
| 1  | John Smith |
| 2  | John Doe   |
+----+------------+
```

We obviously needs another table to log the changes (I am not using EAV to
simplify my writing):

```
+----+---------+--------------+
| Id | User Id | Name         |
+----+---------+--------------+
| 1  | 1       | John Smith   |
| 2  | 2       | John Doe     |
| 3  | 2       | John Doesn't |
+----+---------+--------------+
```

```lisp
(update-table "user" "name" "John Doesn't")
(insert-table "user_audit" "John Doesn't")
```

And that audit table has the same idea as streams: we store every changes for
later checks.

## Conclusion

I wrote some of my understanding on streams, generators, and observables here,
since they have some interesting similarities. I also gave you some example
applications of Scheme streams and Python generators. Let us hope that these
tools will give us better choices on our problem understanding and solving
journey!
