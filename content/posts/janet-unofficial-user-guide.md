---
title: "Janet Unofficial User Guide"
date: 2021-12-08T19:03:46+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - how-to-guide
  - tutorial
tags:
  - janet
  - lisp
---

I was fiddling with the language for a while to implement bit diddling within a
personal project, and liked it a lot. The story is the same for "niche"
programming languages: there are not a lot of discussions nor tutorials and
guides. I hope that my post will be one of them.

The post aimed at people who are not afraid of tinkering around the command line
interfaces, and knew a bit of Vim (Emacs should be fine here, also). Experience
with Lisp-family languages (Clojure is the closest one) can also make your life
much easier here.

## Developing Environment Setup

Janet can be downloaded and tested easily, but I still recommend people to
download the source, do `sudo make install` and `sudo make install-jpm-git` if
it is possible, since the other "ways" can caused some strange bugs from not
having installed enough C header files (it feels ugly, but we have to bear with
it as Janet was written in C).

For code editing, Janet is well-supported within three editors:

- Emacs
- Vim
- Visual Studio Code

You can find more information at [this
repository](https://github.com/ahungry/awesome-janet).

As for me, I "engaged" to Vim (Neovim), so Conjure is the natural choice.

```lua
use 'olical/conjure'
use 'janet-lang/janet.vim'
```

## A "Nontrivial" Program

I found it simplest to explain my Janet code that serve a "real" purpose, and
then help people "picking" useful stuff along the way. Here are my "nontrivial"
requirements:

> Write a program that picks a random option from an option pool. The picking is
> random, but also depends on each option's weight.
>
> For example:
>
> - We have two options, one is weighted 1, and another is weighted 3.
> - The first one should be picked `1/(1 + 3) = 25%` of all the times.
> - The second one should be picked `3/(1 + 3) = 75%` of all the times.

For the sake of simplicity, here are the works that are going to be done:

1. Create a "special" CSV file (how is "special" defined is going to be written
   later)
2. Read the "special" CSV file
3. Mold the CSV file into structured data
4. Pick one option from the data

## "Special" CSV format

We all are familiar with the "normal" CSV:

```csv
col-1,col-2,col-3
val-1,val-2,val-3
val-4,val-5,val-6
val-7,val-8,val-9
```

A "special" one that I invented looks like this:

```csv
column 1,                column 2,                column 3
value 1,                 value 2 which is longer, value 3
value 4 which is longer, value 5,                 value 6 filled
```

Basically, there are spaces padded before the start of each value/header. The
spaces are going to be stripped in our reader.

## "Special" CSV file

Let us have some sample content:

```csv
name,   weight
Vim,    8
Emacs,  3
VSCode, 1
```

Save the content to any path you want. I am going to save it to `/tmp` and my
path then is `/tmp/input.csv`

## Lisp/Clojure/Janet 101

To define a variable, we use this syntax:

```janet
(def ...)
```

To define a function, we use this syntax:

```janet
(defn function-name
  [argument-1 argument-2]
  ...)
```

To "execute" a function, we wrap the function name within parenthesis:

```janet
(function-name)
```

To execute a function with arguments, we put them into the execution, next to
the function name:

```janet
(function-name argument-1 argument-2)
```

There is only one "main" data structure within Lisp (the language is literraly
*LIS*t *P*rocessing): linked list. Clojure adds a bit more to that however:
map/table/dictionary and set. Janet copies Go and does not implement a delicated
set in the hope that people is going to use map instead. It also defaults to
mutable data that are prefixed with an `@`.

```janet
# comments in Janet start with `#`

[1 2 3 4] # immutable
{:key value} # immutable

@[1 2 3 4] # mutable
@{:key value} # mutable 
```

## File Reading

File reading in Janet is extremely simple:

```janet
(def path "/tmp/input.csv")
(def my-file (file/open path :r))
(def raw-text (file/read my-file :all))
# name,   weight
# Vim,    8
# Emacs,  3
# VSCode, 1
```

In which we can put into small function:

```janet
(defn read-raw-text
  [path]
  (file/read (file/open path :r)
             :all))
```

The code works, but kind of hard to read. We can use the threading macro `->`
like this:

```janet
(defn read-raw-text
  [path]
  (-> path
      (file/open :r)
      (file/read :all)))
```

To be simply put, `->` is the same as our direct call with much better
readability:

```janet
(-> argument-1
    (function-1 argument-2)
    (function-2 argument-3))

# the code gets transformed into
#
# ```
# (function-2 (function-1 argument-1 argument-2) argument-3))
# ```
#
# look carefully at how `argument-1` gets put into `function-1`, and `(function-1
# ...)` gets put into `function-2`
```

## Data Transformation

Having our raw text read, the natural progress is to transform the data to our
needs:

```janet
(defn read-special-csv-lines
  [raw-text]
  (->> raw-text
       (|(string/split "\n" $))
       (map string/trim)
       (|(slice $ 1 -2))
       )
  )

(defn parse-special-csv-line
  [csv-line]
  (->> csv-line
       (string/split ",")
       (map string/trim)
       ))

(defn infere-option
  [values_]
  (def [name weight] values_)
  {:name name
   :weight (parse weight)})
```

`read-special-csv-lines` does nothing special:

- Split the raw text by line
- Trim white spaces at the beginning and the end of each line
- Gets the lines, except the first and the last, since we 

`parse-special-csv-line` also does nothing special:

- Split each line by a comma (`,`)
- Trim white spaces at the beginning and the end of each word

`infere-option` simply structure our data into a table/dictionary/map.

There are a few interesting syntaxes of Janet that can be explained here:

- `->>` is the same as `->`, but with a small twist: `->>` put the argument into
  the last position, while `->` put the argument into the second position.
- `|(...)` is Janet's syntax for short anonymous function. `$` within `|(...)`
  is the function's argument.
- `slice` create a copy of the array from a beginning position to an end
  position. `(slice ... 1 -2)` means we are skipping the first and the last
  element of the original array.

## Data Pipeline

Our data pipeline then can be seen like this:

```janet
(def path "/tmp/input.csv")
(->> path
     read-raw-text)
# @"name,   weight\nVim,    8\nEmacs,  3\nVSCode, 1\n"
```

We can put everything else into the pipeline naturally:

```janet
(->> path
     read-raw-text                # @"name,   weight\nVim,    8\nEmacs,  3\nVSCode, 1\n"
     read-special-csv-lines       # ("Vim,    8" "Emacs,  3" "VSCode, 1")
     (map parse-special-csv-line) # @[@["Vim" "8"] @["Emacs" "3"] @["VSCode" "1"]]
     (map infere-option)          # @[{:name "Vim" :weight 8} {:name "Emacs" :weight 3} {:name "VSCode" :weight 1}]
     )
```

## Functional Processing

Our function to pick an option from the above options looks like this:

```janet
(defn pick-an-option
  [options]
  (let [weights (map |(get $ :weight) options)
        weights-sum (sum weights)
        weights-accumulated (accumulate2 + weights)
        random-threshold (* weights-sum (math/random))
        picked-option-index (find-index
                              (fn [weight]
                                (>= weight random-threshold))
                              weights-accumulated)
        picked-option (options picked-option-index)]
    picked-option)
  )
```

`map` and `sum` may not need to be explained, but `accumulate2` surely needs. It
takes an operator, and a collection, and assure that the elements of the result
are the accumulated values.

```janet
(accumulate + [1 2 3 4])
# [1 (+ 1 2) (+ 1 2 3) (+ 1 2 3 4)]
```

`find-index` takes a predicate, and a collection, and return the index of the
first element that satisfies the predicate.

```janet
(find-index |(> 3 $) [1 2 3 4 5])
# 4
```

A number that gets into a form with an array is the 0-indexed "number-th"
element of that array. That was what we have done with `picked-option`.

```janet
([1 2 3] 0)
# 1
([1 2 3] 1)
# 2
```

Let us slap a quick `printf-picked-option`

```janet
(defn printf-picked-option
  [option]
  (string "The picked option: "
          (get option :name)))
```

Into the current pipeline:

```janet
(->> path
     read-raw-text
     read-special-csv-lines
     (map parse-special-csv-line)
     (map infere-option)
     pick-an-option
     printf-picked-option)
```

Evaluate it a few times and we immediately see the result:

```
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: Emacs"
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: Emacs"
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: Vim"
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: Vim"
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: Vim"
# --------------------------------------------------------------------------------
# eval (root-form): (->> path read-raw-text read-special-csv-lines (map pars...
"The picked option: VSCode"
```

We have done a lot with just less than 100 lines of code! You can see the full
code on this GitHub [gist](https://gist.github.com/thanhnguyen2187/f14032df0ada5b3ea7de3541061f13c2).

## Conclusion

I covered the basic of Janet within my post. Reading the input argument and then
compiling (yes; it is possible to compile your script into a binary with Janet)
is left as exercises for the reader.
