---
title: "The Art of Text Editing With Vim"
date: 2021-11-14T19:11:58+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - how-to-guide
tags:
  - vim
  - text-editing
---

The topic has been in my mind for a while. I planned to have a talk about it
within my company, but procrastinated for quite long time. The sections' order
is probably quite unconventional, but I hope that the content makes sense for
you.

![vim](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9f/Vimlogo.svg/1022px-Vimlogo.svg.png)

## Recommendations

### Getting Started

You mainly have two options:

- Typing `vim` (or `nvim`) within your terminal (1)
- Installing a Vim emulating plugin within your current IDE/text editor (2)

I highly recommend (2) for a better learning curve. If you start with (1), apart
from the key binding itself, you also have to deal with:

- Configuring Vim/Neovim
- Finding a plugin manager
- Choosing the plugins for your purpose

If you are not to familiar with the terminal nor how to work efficiently with
it (installing shell suggestion, using `tmux` for tab management), you are going
to have a bad time living within the terminal.

More specificly, within JetBrain's IDEs, you can install a plugin called
"IdeaVim". Within VSCode, there is another plugin named "VSCode Vim" for you to
try. I am not too familiar with other IDEs/text editors, but I am pretty sure
you can find good Vim emulating plugins within them.

### Vim versus Neovim?

![neovim](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/Neovim-logo.svg/2560px-Neovim-logo.svg.png)

I am bullish on Neovim for a few reasons:

- Built-in LSP, and
- Lua for plugin writing, both lead to a
- Flourishing community
- Backwards compatible with VimScript (Neovim plugins are not usable within Vim,
  but Vim plugins that are written in VimScript are usable within Neovim)

### Settings

- `set nu`: display line number
- `set rnu`: display relative line number
- `set nohl`: display search highlighting within Neovim
- `set sb`: split below
- `set spr`: split right

### Plugins

- `packer.nvim`: for plugin management within Neovim
- `vim-commentary`: to quickly add comments
- `vim-surround`: surrounding magic
- `vim-repeat`: to make `vim-surround` and complex chords "repeatable"
- `vim-exchange`: a rarely used but useful plugin
- `vim-ReplaceWithRegister`: another rarely used but useful plugin

## Demonstration

Too bad, I had some idea for my talk, but did not figure it out how to do it
within text form.

## The Art of Text Editing

A few other perks of being familiar with Vim is often listed as:

- Be universal (most bare-bones servers have them installed following POSIX
  standard)
- Boost your text editing productivity
- Help you to be less scared of editing configuration files

But for me, the reason is using Vim makes text editing more enjoyable, and less
of a chore. In its purest sense, Vim works following this model: "verb" +
"adjective" + "noun". Sometimes, you can also combine "verb" with "verb" to make
the command work on one line:

- `dap` means "*d*elete *a* *p*aragraph"
- `yiw` means "*y*ank *i*n *w*ord"
- `dd` means "*d*elete a line"
- `yy` means "*y*ank a line"

The combination is sometimes called "chording", which reminds us of... playing
a musical instrumental, or something art-related, does it not?

Typing "art definition" in Google gives us

> The expression or application of human creative skill and imagination,
> typically in a visual form such as painting or sculpture, producing works to
> be appreciated primarily for their beauty or emotional power.

Which intersects my definition at the word "beauty". I simply think that, "art
can be useless, but is something that makes our living more enjoyable and
beautiful". You are able to draw the conclusion yourself on Vim and text
editing, are you not?

## Conclusion

I had a kind of "unconventional" outline in the hope that it is going to make
the content less boring (in the talk, I planned to include short demonstration
sessions within the talk to make it a bit more interactive). The journey was
short, but hopefully enjoyable and sparked an interest in you on Vim and makes
text editing less tedious.

I did not touch Emacs here, since I am not too familiar with it (I dabbled a bit
into Doom Emacs, but a few small fixes here and there on key binding of Emacs's
`evil mode` repelled me out of the editor completely). Let us hope that I can
overcome it in the future, and write something even more compelling.
