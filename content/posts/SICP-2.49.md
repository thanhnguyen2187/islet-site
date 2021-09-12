---
title: "SICP 2.49"
date: 2021-08-08T14:30:24+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - reference
tags:
  - lisp
  - sicp
  - scheme
---

It's been a while since I felt that... thrilled working on something. The
challenges did not come from the difficulties of the exercise, but rather than
that, the... random tooling problem that I had with Racket. More than that, I
felt bored at the fourth exercise, so I tried implementing a smoothing
procedure. I struggled for a while and finally made it work. Anyway, let us see
the problem first.

## Description

> **Exercise 2.49**: Use `segments->painter` to define the following primitive
> painters:
> 
> a. The painter that draws the outline of the designated frame.
> 
> b. The painter that draws an "X" by connecting opposite corners of the frame..
> 
> c. The painter that draws a diamond shape by connecting the midpoints of the
> sides of the frame.
> 
> d. The `wave` painter.

## Making Racket Works

I followed the "modern" way working with Scheme: download Racket and use
DrRacket. The sad thing is DrRacket does not seem to have a decent Vim emulator,
either. I tried Emacs a while ago, but could not spend the time customizing it
more, plus, at the time, I was not too familiar with Lisp either. Therefore, I
resorted to `vim-racket`, and somehow made an okay workflow with Vim, tmux,
and tmuxp.

![sicp-workflow](../images/sicp-workflow.png)

A few quirks irk me a lot, but I learned to live with it:

- `#lang sicp` somehow will make Vim lost its Racket syntax, and it feel damn
  horrible.
- In the expression `if (s1) s2 s3`, newline with `s2` indents two spaces, while
  the codes in SICP always indent four spaces.

```lisp
; the "normal" format
(if (s1)
  s2
  s3)

; the "right" format
(if (s1)
    s2
    s3)
```

Copying the sample code into your file probably will not work with Racket,

```lisp
(define (segments->painter segment-list)   
  (lambda (frame)
    (for-each
      (lambda (segment)
        (draw-line
          ((frame-coord-map frame) (start-segment segment))
          ((frame-coord-map frame) (end-segment segment))))
      segment-list)))
```

as you get a cryptic message:

```
draw-line: unbound identifier in module in: draw-line
```

Researching leads you to quite thorough answers on
[Stackoverflow](https://stackoverflow.com/questions/13592352/compiling-sicp-picture-exercises-in-drracket),
which basically is doing stuff to glue the code with Racket's native drawing.
The fix can look like this:

```lisp
#lang racket

(require graphics/graphics)
(open-graphics)
(define vp (open-viewport "A Picture Language" 500 500))

(define draw (draw-viewport vp))
(define (clear) ((clear-viewport vp)))
(define line (draw-line vp))

(define (vector-to-posn v)
  (make-posn (ycor-vect v)
             (xcor-vect v)))

(define (segments->painter segment-list)   
  (lambda (frame)
    (for-each
      (lambda (segment)
        (line
          (vector-to-posn ((frame-coord-map frame) (start-segment segment)))
          (vector-to-posn ((frame-coord-map frame) (end-segment segment)))))
      segment-list)))
```

Which is similar to the author's answer, except the better formatting. Also, I
want to send myself in a parallel world---who did the same mistake of not
understanding the implementations---a message. Read carefully what a `frame` is,
what a `vector` is, and understand how can two `vector`s create a new `segment`.
Understanding the reasons above codes will lead to an easier life that is not
riddled with bugs from copying and pasting code.

## Smooth Segments Created By Three "Points"

Originally, we are not asked to do it, but I felt bored with the fourth exercise
which involves a lot of manual works. Therefore, I "swapped" it with a better
one:

- create a procedure `smooth` which
- takes three "points" (`vector`s to be precise), and
- returns "smoothed points" between the first point and the third point, which
  "follows" the second point
- turns the points into segments to visualize

A picture (or two) is worth a thousand words, I guess:

|                                                                 |                                                                       |
| ---                                                             | ---                                                                   |
| ![scip-2.49-raw-segments](../images/scip-2.49-raw-segments.png) | ![sicp-2.49-smooth-segments](../images/sicp-2.49-smooth-segments.png) |

I tried looking up other algorithms, but did not felt intuitive enough about
them, so I implemented one that feel "natural" to me anyway. The core idea of
my thought is:

- The input is a list of three points (again, in this case they are `vector`s,
  but damn it anyway)
- See if the segments are "smooth" enough by checking if the middle of the
  first point and the third point is "close enough" to the second point
- In the case it is: return the three points
- In the case it is not:
    - Smooth the segments from the first point to the second point
    - Smooth the segments from the second point to the third points
    - Join the two results together

A lot of trials and errors guided me to this implementation:

```lisp
(define tolerance 0.01)
(define (smooth v1 v2 v3)
  ; smooth by averages until the distance between v2 and middle of v1 and v3 is
  ; less than tolerance
  (let ((middle-v1-v3 (middle-line v1 v3)))
    (if (< (distance middle-v1-v3 v2)
           tolerance)
        (list v1 v2 v3)
        (flatmap (lambda (vectors)
                   (let
                     ((v1 (car vectors))
                      (v2 (cadr vectors))
                      (v3 (caddr vectors)))
                     (smooth v1 v2 v3)))
                 (list (list v1
                             (middle-line v1 v2)
                             (middle-line middle-v1-v3 v2))
                       (list (middle-line middle-v1-v3 v2)
                             (middle-line v3 v2)
                             v3))))))
```

A procedure to create `segment`s  from points is implemented like this:

```lisp
(define (zip sequence-1
             sequence-2)
  (if (or (null? sequence-1)
          (null? sequence-2))
      '()
      (cons (list (car sequence-1)
                  (car sequence-2))
            (zip (cdr sequence-1)
                 (cdr sequence-2)))))

(define (make-segments vectors)
  (map (lambda (pair) (make-segment (car pair)
                                    (cadr pair)))
       (zip vectors
            (cdr vectors))))
```

The idea behind `make-segments` is that we look at the way of "joining"
points as choosing corresponding points of two parallel `list`:

```lisp
(list 1 2 3 4 5 6)
(list 2 3 4 5 6)

; somehow, we must turn the two list into

(list (1 2) (2 3) (3 4) (4 5) (5 6))
```

`zip` is the one we use here. It creates a new `list` that are tuples of
same-indexed elements from other `list`s. `zip` stops when it exhausts one of
the input `list`s.

```lisp
(zip (list 1 2 3 4 5 6) (list 2 3 4 5 6))
; ((1 2) (2 3) (3 4) (4 5) (5 6))
(zip (list 1 2 3 4) (list))
; ()
```

## Conclusion

Just read my [Yet Another SICP Advocating](../yet-another-sicp-advocating). A
messy implementation can be found on my
[GitHub](https://github.com/thanhnguyen2187/sicp-notes/blob/master/chapter-2/exercise-2.49.rkt).
