---
title: "\"Good\" Practices in Angular"
date: 2021-07-02T23:31:41+07:00
draft: true
toc: true
images:
categories:
  - explanation
  - how-to-guide
tags:
  - angular
  - typescript
  - frontend
---

I named this post "Good" Practices, not "Best", since I believe that there is
hardly a silver bullet that can be used to solve everything, technology oriented
or not. The practices are good since they fit my case, and I hope that it would give you
(or my future self) some useful insights.

## Monorepo Development Style with Angular Workspace

Normally, we are used to Angular CLI's `new` command:

```bash
ng new <my-project>
```

The style is normal for a "multi-repo" development style, where we split our
projects into multiple repositories (Git repository to be precise). More
information on monorepo and multirepo can be found outside of my post. We shall
focus on Angular Workspace here however. Workspace of Angular is kind of another
word for "monorepo", where we have "projects" (another word for "repositories",
you guessed it).

A blank workspace can be created using this command:

```bash
ng new <workspace-name> --create-application false
```

I will create a sample workspace like this:

```bash
ng new sample-workspace --create-application false
```

Let us look at the workspace's file structure:

```
/.../sample-workspace
├── angular.json
├── node_modules
│  ├── @angular
│  ├── @angular-devkit
│  ├── ...
│  └── zone.js
├── package-lock.json
├── package.json
├── README.md
└── tsconfig.json
```

There is nothing special, except the... emptiness. We will see the changes now,
however. A typical use case is we want to create shared libraries and
applications within our monorepo. It can be done with these commands:

```bash
ng generate application first-app
ng generate application second-app
ng generate library common-lib
```

We shall see the newly-created files and folders:

```
/.../sample-workspace
├── angular.json
├── node_modules
│  ├── @angular
│  ├── @angular-devkit
│  ├── ...
│  └── zone.js
├── package-lock.json
├── package.json
├── projects (!)
│  ├── common-lib
│  ├── first-app
│  └── second-app
├── README.md
└── tsconfig.json
```

Our workflow with Angular changes a little bit. Instead of a simple command:

```bash
ng serve
# ng build
```

We specify our application or library with `--project`:

```bash
ng serve --project first-app --port 4200
# ng serve --project second-app --port 4210
# ng build --project common-lib --watch
```

Another useful option that I often use with libraries is `--watch`, to build the
library against changes.

## Module

## Getter and Setter

## Directive

## Conclusions

## External Links

- https://angular.io/guide/file-structure
