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

## Monorepo Development Style

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

## "Clean" Importing

The Official Documentation has some lengthy and verbose descriptions on Module,
but I will try to make it clearer and more comprehensive: _Angular Module is the
Angular's way to group source files_, nothing more or less.

New developer often skip Module creating, and I do not blame them for that (as I
did not care about Module at first, either). When the application gets bigger
and need to be splitted into smaller chunks, we then do see how Angular Module
shines. Lazy loading is another important feature that we need to notice.

At first, let us create an application somewhere, and say yes to routing since
it is going to be useful later:

```bash
ng new sample-app
# cd sample-app
```

Our newly-created application shall look like this:

```
/.../sample-app
├── angular.json
├── karma.conf.js
├── node_modules
│  ├── @angular
│  ├── @angular-devkit
│  ├── ...
│  └── zone.js
├── package-lock.json
├── package.json
├── README.md
├── src
│  ├── app
│  ├── assets
│  ├── environments
│  ├── favicon.ico
│  ├── index.html
│  ├── main.ts
│  ├── polyfills.ts
│  ├── styles.css
│  └── test.ts
├── tsconfig.app.json
├── tsconfig.json
└── tsconfig.spec.json
```

Let us navigate into `src/app` and have a look at the main module called `app`,
which is used to "bootstrap" the whole application (dear me from an alternative
timeline which did not know what "bootstrap" is; it means you are
starting/running the application using itself):

```
/.../sample-app/src/app
├── app-routing.module.ts
├── app.component.css
├── app.component.html
├── app.component.spec.ts
├── app.component.ts
└── app.module.ts
```

`app.module.ts` is the important one right now:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Let us create a new component:

```bash
ng generate component components/first-component
```

And look at how `app.module.ts` changed:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
// new line added
import { FirstComponentComponent } from './components/first-component/first-component.component';

@NgModule({
  declarations: [
    AppComponent,
    // new line added
    FirstComponentComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

It does not look bad, but let us imagine that we have the second, third, and
fourth components...

```bash
ng generate component components/second-component
ng generate component components/third-component
ng generate component components/fourth-component
```

The file quickly becomes more convoluted:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
// new lines added
import { FirstComponentComponent } from './components/first-component/first-component.component';
import { SecondComponentComponent } from './components/second-component/second-component.component';
import { ThirdComponentComponent } from './components/third-component/third-component.component';
import { FourthComponentComponent } from './components/fourth-component/fourth-component.component';

@NgModule({
  declarations: [
    AppComponent,
    // new lines added
    FirstComponentComponent,
    SecondComponentComponent,
    ThirdComponentComponent,
    FourthComponentComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Typically, we should split our application into smaller components, and we then
can imagine the mess within `app.module.ts`. This is one main reason why Angular
Module got created.

Our workflow evolves from a single step

```bash
ng generate component <component-name>
```

to a few steps:

```bash
ng generate module modules/feature
ng generate component modules/feature/first-component
ng generate component modules/feature/second-component
ng generate component modules/feature/third-component
```

Our folder structure then becomes:

```
/tmp/sample-app/src
├── app
│  ├── app-routing.module.ts
│  ├── ...
│  └── app.module.ts
├── assets
├── environments
│  ├── environment.prod.ts
│  └── environment.ts
├── favicon.ico
├── index.html
├── main.ts
├── modules
│  └── feature (*)
│     ├── feature.module.ts
│     ├── first-component
│     ├── second-component
│     └── third-component
├── polyfills.ts
├── styles.css
└── test.ts
```

Let us have a look inside `feature-module.module.ts`:

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FirstComponentComponent } from './first-component/first-component.component';
import { SecondComponentComponent } from './second-component/second-component.component';
import { ThirdComponentComponent } from './third-component/third-component.component';


@NgModule({
  declarations: [
    FirstComponentComponent,
    SecondComponentComponent,
    ThirdComponentComponent
  ],
  imports: [
    CommonModule
  ]
})
export class FeatureModuleModule { }
```

Angular is "smart" (!) enough to add the components to the module itself. The
last thing we have to do is add the module to the right place (in this example,
we add it in `app.module.ts`):

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { FeatureModule } from '../modules/feature/feature.module'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    // the new module is added below
    FeatureModule, 
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

To summarize the long section, we shall have an "action list":

## Getter and Setter

It is good practice in Angular (and in every other programming language) to
encapsulate our data.

Let us come to a fictional "classic":

```ts
```

## Directive

## Conclusions

## External Links

- https://angular.io/guide/file-structure
