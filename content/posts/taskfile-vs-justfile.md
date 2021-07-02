---
title: "Taskfile vs Justfile"
date: 2021-06-30T22:25:09+07:00
draft: false
toc: true
tags:
- taskfile
- justfile
- task-runner
categories:
- explanation
series: 
- comparison
---

A more correct title would be "`task` vs `just`", but it will look confusing for
the search engines. I did not find a comparison post on anywhere, so here we
are. In this post, I will tell my experiences and evaluations on them.

## TL; DR

- Scripting arguments: `just`
- "Subfile" and working directory: `task`
- Autocompletions: `just`

## Scripting arguments

`task` and `just` both advertise themselves as `make`'s alternative that
dedicate to task-running. Apart from the obvious different file format, I could
hardly find any differences at first, but after testing both for a while, I
think the most important question is: **do you expect to use arguments within
your scripts?**

If the answer is yes, stick with `just`.

Let us see a "real-life" `justfile` [by me](https://github.com/thanhnguyen2187/scripts/blob/master/justfile):

```makefile
create-new-script FOLDER NAME:
    mkdir -p {{FOLDER}}
    echo "#!/bin/bash" > {{FOLDER}}/{{NAME}}.sh
    chmod +x {{FOLDER}}/{{NAME}}.sh
```

And the command:

```bash
just create-new-script asdf install-nodejs
```

Compare it to a fictional equivalent `Taskfile.yml`:

```yml
version: '3'
tasks:
  create-new-script:
    cmds:
      - mkdir -p {{.FOLDER}}
      - echo "#!/bin/bash" > {{.FOLDER}}/{{.NAME}}.sh
      - chmod +x {{.FOLDER}}/{{.NAME}}.sh
```

And the command:

```bash
task create-new-script FOLDER=asdf NAME=install-nodejs
```

## "Child" `task`/`just`file and different working directory

I group the two use cases into one, since the file format can show them both.
`task` obviously handles the two cases better than `just`.

Let us consider another "real-life" Taskfile [by
me](https://github.com/thanhnguyen2187/amber/blob/master/Taskfile.yml):

```yml
version: '3'
includes:
  frontend:
    dir: ./frontend-angular-workspace
    taskfile: ./frontend-angular-workspace/Taskfile.yml
  backend:
    dir: ./backend-golang
    taskfile: ./backend-golang/Taskfile.yml
----
# frontend-angular-workspace/Taskfile.yml
version: '3'
tasks:
  build-core:
    cmds:
      - ng build
        --project amber-core
```

We can invoke the other Taskfile like this:

```bash
task frontend:build-core
```

Let us see a "fictional" equivalent Justfile:

```makefile
frontend-build-core:
    cd frontend-angular-workspace && ng build --project amber-core

# an alternative version
frontend-build-core-alt:
    #!/bin/bash
    cd frontend-angular-workspace
    ng build --project amber-core
```

The command would be:

```bash
just frontend-build-core
# just frontend-build-core-alt
```

## Autocompletions

`just` is a tad better since the `--completions` is built-in. `task` needs a
little hack [elsewhere](https://github.com/go-task/task/issues/103), and it is
not even complete.

The way to use `just --completions` is not so obvious in zsh, however. We need
to add it to `fpath`, and execute `compinit` later.

```bash
# just --completions zsh > /tmp/_just
fpath += /tmp
compinit
```

## Conclusion

It all boils down to your preference, as the lackluster functionalities of each
are not a real deal breaker when you are used to them. Even though I am using
`just` for my tasks nowadays, I do think that `task`'s Taskfile is slightly
better than Justfile. It may also comes to your programming language liking
(`task`'s Golang, versus `just`'s Rust).
