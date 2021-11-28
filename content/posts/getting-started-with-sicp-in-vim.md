---
title: "Getting Started with SICP in Vim"
date: 2021-11-26T20:03:45+07:00
draft: false
toc: true
images:
categories:
  - how-to-guide
tags:
  - sicp
  - scheme
  - lisp
---

I had the guide on my mind for a while, but could hardly find the motivation to
write it. A new friend of mine inspired me on writing, however, as he said that
he treaded through the book without doing the exercise. I hope that the post can
help him (or other people) to get started with SICP easier.

Even though my tutorial uses `apt` as the system package manager, and
`packer.nvim` as Neovim's package manager, I think tailoring the commands to
other package manager and operating system is not going to be too hard.

## Installing Racket

MIT/GNU Scheme is the "official" way, but I get started with Racket, so my
tutorial is on that. Racket is more popular and the tooling around it is a bit
better than MIT/GNU Scheme, it is up to you to make the decision.

```bash
sudo add-apt-repository ppa:plt/racket
sudo apt update
sudo apt install racket

racket --version
# Welcome to Racket v8.2 [cs].
```

You need a custom "language" installed within Racket to make the syntax work:

```bash
raco pkg install sicp
```

Check if everything works by writing this content to a file named
`hello-world.rkt`:

```lisp
#lang sicp

(display "Hello world")
```

And then try to run it:

```bash
racket hello-world.rkt
# "Hello world"
```

## Vim Plugins

Have `conjure` (which is a nice REPL integration within Vim), and `vim-racket`
ready using your Vim plugin manager:

```lua
use 'olical/conjure'
use 'wlangstroth/vim-racket'
```

Congratulation! You are almost ready!

## Tips and Tricks

### Syntax Highlighting

Try editing `hello-world.rkt`, and it may look like syntax highlighting is not
working.

![Screenshot from 2021-11-26 20-45-08.png](https://2.pik.vn/20219dd5030b-2659-43c3-bec4-7a10a530131f.png)

Put a comment on the first line and try again as the work around:

![Screenshot from 2021-11-26 20-46-58.png](https://2.pik.vn/2021c19764a7-da30-4405-a139-747e84201105.png)

### Useful Conjure Keybindings

You can easily find everything with `:h conjure-mappings`, but these are the
most useful keybindings for me:

- `<localleader>lv` for a new vertical REPL split
- `<localleader>er` to evaluate the outermost form
- `<localleader>ew` to evaluate a word

![Screenshot from 2021-11-26 20-49-06.png](https://2.pik.vn/2021f0ffbd79-494f-4e76-b6ae-ccdd27aa7f03.png)

(the default key for local leader is `\`)

### Ease The "Working Out" Experience

At first, I tried typing everything within the book into a "scratch" file, then
copy the code again and again. It does not spark joy at all: typos and stuff
make you spend more time fighting the language and the tooling than actually
learning the subject.

Do yourself a favour: copy the right code from this
[place](https://mitpress.mit.edu/sites/default/files/sicp/code/index.html) to a
"big" file and work on it.

## Conclusion

The post is short and terse, but I hope that you found something useful from it!
