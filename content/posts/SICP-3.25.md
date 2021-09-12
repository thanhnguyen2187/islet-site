---
title: "SICP 3.25"
date: 2021-09-12T20:37:19+07:00
draft: true
toc: true
images:
categories:
  - explanation
  - reference
tags:
  - sicp
  - lisp
  - scheme
---

I struggled with this exercise for quite the time. Therefore, I am going to post
some code here in the hope that people will have look at it and immediately
understand what the hell has I done. Just kidding. I will try to explain the
work.

## Description

> **Exercise 3.25**: Generalizing one- and two-dimensional tables, show how to
> implement a table in which values are stored under an arbitrary number of keys
> and different values may be stored under different numbers of keys. The
> `lookup` and `insert!` procedures should take as input a list of keys used to
> access the table.

## `make-ptr`

At first, I did not understand why we need `*table*` within a list for `ptr`,
but then I understood it when having "fun" `set!`ing stuff with the copy of a
list. I decided to implement a "rich man's" pointer, and let us look how simple
it is with message passing:

```lisp
(define (make-ptr value)

  (define (dispatch message)
    (cond ((eq? message 'value) value)
          ((eq? message 'set-value!) (lambda (new-value)
                                       (set! value new-value)))
          (else (error "Unknown operation: MAKE-PTR" message))))

  dispatch)
```

The dereferencing can be done with a simple call:

```lisp
(define ptr (make-ptr 13))
(ptr 'value)
; 13
```

## `make-kv-pair`

Also, let us not forget to define a key-value pair, since I was not really
satisfy with all the `cadr`ing of the original implementation:

```lisp
(define (make-kv-pair key value)

  (define (set-value! new-value)
    (set! value new-value))

  (define (dispatch message)
    (cond ((eq? message 'key)   key)
          ((eq? message 'value) value)
          ((eq? message 'set-value!) set-value!)
          (else (error "Unknown operation: MAKE-KV-PAIR" message))))

  dispatch)
```

## `make-table`

### `kv-pairs`

Key-value Pairs of our table then is a pointer, but why? You will see it later.

```lisp
(define (make-table same-key?)

  (let ((kv-pairs-ptr (make-ptr (list))))

    ...

    (define (dispatch message)
      (cond ((eq? message '...) (...))
            (else (error "Unknown operation: MAKE-TABLE" message))))

    dispatch))
```

### `assoc`

To find the pair itself between pairs then can be done like this:

```lisp
(define (assoc key kv-pairs)
  (if (null? kv-pairs)
      false
      (let ((kv-pair (car kv-pairs)))
        (if (same-key? key (kv-pair 'key))
            kv-pair
            (assoc key (cdr kv-pairs))))))
```

### `lookup-single`

The function is used to find the value of of a key within the key-value pairs.
It also complements `lookup`, that we are going to have a look at right after.

```lisp
(define (lookup-single key kv-pairs)
  (let ((kv-pair (assoc key kv-pairs)))
    (if kv-pair
        (kv-pair 'value)
        false)))
```

### `lookup`

Oh, let us come to the interesting part: look a value up with `keys`, instead of
`key`:

```lisp
(define (lookup keys kv-pairs)
  ; Assume that ther is at least one key. If there is more keys, call the
  ; function again with those keys and the result of lookup-ing one key,
  ; which is potentially key-value pairs.
  (let ((key (car keys))
        (rest-keys (cdr keys)))
    (let ((lookup-result (lookup-single key kv-pairs)))
      (if (and (not (null? rest-keys))
               (eq? (type lookup-result) 'ptr))
          (let ((rest-kv-pairs lookup-result))
            (lookup rest-keys rest-kv-pairs))
          lookup-result))))
```

The comment itself was fairly enough, but the type checking? It is because
later, if we look at the table as key-value pairs, a two-dimensional table would
be a key-value pair, which has its value as key-value pairs itself.

We can roughly see them like this:

```lisp
((key-1
  ((key-1-1 1) (key-1-2 2) (key-1-3 3)))
 (key-2
  ((key-2-1 4) (key-2-2 5) (key-2-3 6))))
```

The type checking is left as an exercise... No. They will be explained later.

The code has a bug however: if we use a pointer as a value somewhere, it broke
the implementation. It can be fixed if we treat `kv-pairs-ptr` as a type itself,
but I got lazy at the work.

### `insert-single!`

The implementation is simple enough with a pointer, which was kinda similar to
the `set-cdr!` part of the original implementation.

```lisp
(define (insert-single! key value kv-pairs-ptr)
  (let ((kv-pairs (kv-pairs-ptr 'value)))
    (let ((kv-pair (assoc key kv-pairs)))
      (if kv-pair
          ((kv-pair 'set-value!) value)
          ((kv-pairs-ptr 'set-value!) (cons (make-kv-pair key value)
                                            kv-pairs)))
    'ok)))
```

### `insert!`

Here come the interesting part (again). The idea of `insert!` is kinda similar
to `lookup!`, which is we try to do our work with the key again and again, until
there is one key left. Also, do not forget to `insert-single!` with the current
key.

```lisp
(define (insert! keys value kv-pairs-ptr)
  ; Assume that there is at least one key. If there is more keys, "save" the
  ; first key as a pointer and insert it to the current key-value pairs to
  ; let `lookup` do its work later.
  (let ((kv-pairs (kv-pairs-ptr 'value)))
    (let ((key (car keys))
          (rest-keys (cdr keys)))
      (if (null? rest-keys)
          (insert-single! key value kv-pairs-ptr)
          (let ((rest-kv-pairs (cdr kv-pairs)))
            (let ((rest-kv-pairs-ptr (make-ptr rest-kv-pairs)))
              (insert-single! key rest-kv-pairs-ptr kv-pairs-ptr)
              (insert! rest-keys value rest-kv-pairs-ptr))))))
  'ok)
```

Here lies the reason why we need a pointer: without a pointer, `set!` and its
variations simply does not work. The underlying reason is Scheme does
pass-by-value to function's arguments. `set!` the copy does nothing to the real
variable.

Another way to implement this without `set!` is to do the mutation within
`dispatch`, but sadly the solution did not occur to me at the time.

### `dispatch`

It is a boring function, but we need it to do the work:

```lisp
(define (dispatch message)
  (cond ((eq? message 'lookup) (lambda (keys)
                                 (lookup keys (kv-pairs-ptr 'value))))
        ((eq? message 'lookup-single) (lambda (key)
                                        (lookup-single key (kv-pairs-ptr 'value))))
        ((eq? message 'insert!) (lambda (keys value)
                                  (insert! keys value kv-pairs-ptr)))
        ((eq? message 'insert-single!) (lambda (key value)
                                         (insert-single! key value kv-pairs-ptr)))
        ((eq? message 'assoc) (lambda
                                (key) (assoc key kv-pairs-ptr)))
        ((eq? message 'kv-pairs) kv-pairs-ptr)
        (else (error "Unknown operation: MAKE-TABLE" message))))
```

### `type`

It also is a boring function, but we need it anyway:

```lisp
(define (type value)
  (cond ((number? value) 'number)
        ((symbol? value) 'symbol)
        (value 'type)))
```

Within `ptr`'s `dispatch`, we can add a line:

```lisp
((eq? message 'type) 'ptr)
```

## Conclusion

It was fun working on the problem. I saw the power of Scheme/Lisp within
reimplementing `cons` and `cdr`, but to leverage the power myself to implement a
pointer is another fun thing to do. The full code can be found on my GitHub.

Also, here is a meme for you for slogging through this post:

![sicp-3.25-meme](../images/sicp-3.25-meme.png)

I will not try to tell you to read SICP! It is such an interesting book that I
want to keep it to myself!
