---
title: "Getting Started With ClojureScript"
date: 2021-09-04T20:21:46+07:00
draft: false
toc: true
images:
categories:
  - explanation
  - tutorial
tags:
  - clojure
  - lisp
---

Clojure is damn nice language to work with, but the tooling around is such a
pain (even for a "new" developer like me, who dabbled into Linux for a while and
is not afraid of fixing configuration files). I can imagine that a "totally new"
developer will have a much worse time getting started with Clojure (and
ClojureScript). The guide is about ClojureScript, which is Clojure turned into
JavaScript, and has some subtle differences with the "original" Clojure. For
learning purpose, I think it does not make a big different however.

## Toolings

### Java and NodeJS

You need to make sure that you have Java and NodeJS ready within your machine.
You can find ways to install them yourself, or use `asdf`, which is a tool that
can install and manage version for both Java and NodeJS. I will assume that you
had `asdf` ready within your machine, typing this within the terminal should
have a similar response:

```bash
asdf version
# v0.8.1-a1ef92a
```

You then can install NodeJS easily:

```bash
asdf plugin add nodejs
asdf install nodejs 14.17.5
asdf global nodejs 14.17.5
```

Installing Java has the same steps:

```bash
asdf plugin add java
asdf install java adoptopenjdk-11.0.12+7
asdf global java adoptopenjdk-11.0.12+7
```

If you want to plug another version of NodeJS, have a look at the supported
versions:

```bash
asdf plugin list-all nodejs
```

The same command goes with Java:

```bash
asdf plugin list-all java
```

Finally, do a "sanity check":

```bash
node --version
# v14.17.5
java --version
# openjdk 11.0.12 2021-07-20
# OpenJDK Runtime Environment Temurin-11.0.12+7 (build 11.0.12+7)
# OpenJDK 64-Bit Server VM Temurin-11.0.12+7 (build 11.0.12+7, mixed mode)
```

### IDE/Editor

You have quite amount of choices:

* Intellij + Cursive
* VSCode + Calva
* Emacs + clojure-mode
* Vim + fireplace.vim
* Neovim + Conjure
* ...

I tried Emacs, and Doom Emacs, but as a Vim user, subtle key binding changes
drove me crazy. Experiences of trying to get better code suggestion within Vim
shunned me away from the thought. Save yourself the researching and start with
Intellij + Cursive for the start.

## ClojureScript

### How It Works

For Clojure, you can "wrongly" understand it like this:

```
[Clojure Code] -(compiled)-> [Java Bytecode]
```

For ClojureScript, it gets "more" complicated:

```
[Clojure Code] -(compiled)-> [JavaScript Code] -(compiled)-> [Optimized JavaScript Code]
```

The reason for the second "compiled" step is that `[JavaScript Code]` compiled
from the first step is not optimal, you guessed it. The second step is handled
by Closure Compiler from Google, which is... guaranteed enough, I guess.

### Basic Syntax

The post is about Clojure, but I think it is easier to have a grasp with an
example from... Scheme, which is another Lisp.

```lisp
(def (add a b) (+ a b))
```

It may look like alien's language at first, but trust me. _s expression_, if you
need technical jargon, is dead simple.

- `def` stands for "define"
- `add` is a name. `a`, and `b` are arguments
- A pair of round brackets `(...)` without `def` is an execution

Congratulation on understanding 50% of Clojure's code. You substitute `def` with
`defn`, and learn some other constructions:

```clojure
(defn add [a b] (+ a b))
```

- `def` gets replaced by `defn`
- Arguments (`a` and `b`) get put into square brackets `[...]`
- Executions are still wrapped around round brackets `(...)`

Clojure also has other data structures instead of linked list, which is
intuitive enough, but we only need to focus on two of them for now:

```clojure
[1 2 3] ; a vector
{:keyword "value"} ; a map/dictionary
```

To get something from a map, we use the keyword like this:

```clojure
(:keyword a-map)
```

Another thing that you should pay attention to is the notion of "namespace",
which is Clojure's way to split code.

```clojure
(ns my-company.my-module)
```

We can import other modules and their functions in a very simple way:

```clojure
(ns my-company.my-module 
  (:require 
    [my-company.another-module]
    [my-company.another-module-2 :as another-name])
  )
```

Clojure has something called "atoms" to manage shared, synchronous, independent
state. We do `set!`, or `swap!` to change the atom's value. We use
`@atom-variable` to get the atom's value.

```
> (def x (atom 1))
> @x
1
> (set! x 10)
> @x
10
```

I covered the basics, but it is not enough. The best thing you can use is
Clojure official documentation: https://clojure.org/api/cheatsheet.

### Enough Reagent To Be Dangerous

It is simplest to give you an example and then deduce the syntax yourself:

```html
<button onclick="f()">Stuff</button>
<input type="number">
```

```clojure
[:button 
 {:on-click (f)}
 "Stuff"]

[:input 
 {:type "number"}]
```

## 2. Seven GUIs

After you install NodeJS, `npx` should also exist. You can read more on Seven
GUIs from [this](https://eugenkiss.github.io/7guis/tasks/). The purpose is...
learning, I guess.

### Project Initialization

```bash
npx create-cljs-project seven-guis
```

The created project should look have a similar structure:

```
/tmp/seven-guis
├── node_modules
│  ├── asn1.js
│  ├── assert
│  ├── base64-js
│  ├── bn.js
│  ├── brorand
│  ├── ...
│  ├── which
│  ├── ws
│  └── xtend
├── package-lock.json
├── package.json
├── shadow-cljs.edn
└── src
   ├── main
   └── test
```

Replace `shadow-cljs.edn` content with this:

```clojure
;; shadow-cljs configuration
{:source-paths
 ["src/dev"
  "src/main"
  "src/test"]

 :dependencies
 [
  [reagent "1.1.0"]
  ]

 :dev-http {8080 "public"}
 :nrepl {:port 55555}

 :builds
 {:app
  {:target :browser
   :output-dir "public/app/js"
   :asset-path "/app/js"
   :modules {:main {:init-fn entrance/init}}}}
 }
```

Create a file named `index.html` within `public`, and replace the file's content
with this:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Application</title>
</head>
<body>
<div id="app">
</div>
<script
    src="/app/js/main.js"
></script>
</body>
</html>
```

Create a file named `entrance.cljs` within `src/main`, and replace the file's
content with this:

```clojure
(ns entrance
  (:require
    [reagent.dom :as rdom]
    ))

(defn init [])

(defn app []
  "Hello world!"
  )

(rdom/render [app] (js/document.getElementById "app"))
```

Finally, run this:

```bash
npx shadow-cljs watch app
```

Open `127.0.0.1:8080` and have a look. Congratulation on your first application.
You did a lot, and deserve a throughout explanation at the next section.

### What Happened

Let us go through the simplest understanding of how the web works. We basically
have those three within our mind:

- HTML: the "structuring" language
- CSS: the "beautifying" language
- JavaScript: the "making-it-dynamic" language

ClojureScript, by some hocus-pocus magic, gets turned into JavaScript.
`shadow-cljs`, also by some hocus-pocus magic, is actually responsible for the
transformation. It also serves HTML for us. Serving is one thing, but we somehow
still need our web application to be... actually dynamic. React solves the
"actually dynamic" problem. Reagent is React's wrapper within ClojureScript.

### Entrance

Entrance is where you include other exercises as components. The entrance will
look like this:

```clojure
(ns entrance
  (:require
    [reagent.dom :as rdom]
    [reagent.core :as r]
    ;[counter]
    ;[temperature-converter]
    ;[flight-booker]
    ;[timer]
    [crud]
    ))

(defn init [])

(defn app []
  [:div
   ;(counter/component)
   ;(temperature-converter/component)
   ;(flight-booker/component)
   ;(timer/component)
   [crud/component]
   ]
  )

(rdom/render [app] (js/document.getElementById "app"))
```

Every time you want to include/exclude anything, just remove the `;` within the
`:require` and the `defn app`.

### Counter

It looks like this on my Firefox without any styling. 

![getting-started-with-clojurescript-counter](../images/getting-started-with-clojurescript-counter.gif)

The code itself is simple enough. There is nothing complicated here, except the
`swap!`, and the `@`.

```clojure
(ns counter
  (:require
    [reagent.core :as r]
    ))

(defonce current-count (r/atom 0))

(defn component []
  [:div
   @current-count
   " "
   [:button
    {:on-click #(swap! current-count inc)}
    "+1"]
   ])
```

### Temperature Converter

![getting-started-with-clojurescript-temperature-converter](../images/getting-started-with-clojurescript-temperature-converter.gif)

A little bit less trivial and not the cleanest code that I have written, but it
does work anyway. The idea is to represent the temperature as a number with its
type `'celsius` or `'fahrenheit`, and then treat the display accordingly.

An untrivial thing here is how we get the input value:

```
(-> event .-target .-value)
````

`->` is a special syntax of Clojure that makes our code cleaner. `.-target` and
`.-value` are JavaScript/DOM related.

```clojure
(ns temperature-converter
  (:require
    [reagent.core :as r]))

(defonce temperature (r/atom {:value 0
                              :type  'celsius}))

(defn fahrenheit->celsius [temperature-value]
  (* (- temperature-value 32)
     (/ 5 9)))

(defn celsius->fahrenheit [temperature-value]
  (+ (* temperature-value (/ 9 5))
     32))

(defn display-temperature [temperature display-type]
  (let [temperature-type  (:type temperature)
        temperature-value (:value temperature)]
    (case [temperature-type display-type]
      ['celsius    'celsius]    temperature-value
      ['celsius    'fahrenheit] (celsius->fahrenheit temperature-value)
      ['fahrenheit 'celsius]    (fahrenheit->celsius temperature-value)
      ['fahrenheit 'fahrenheit] temperature-value
      (str "Invalid calls: DISPLAY-TEMPERATURE " temperature display-type)))
  )

(defn atomic-input [temperature
                    input-type]
  [:input {:type "number"
            :on-change (fn [event]
                         (swap! temperature
                                assoc
                                :value (-> event .-target .-value)
                                :type  input-type))
            :value (display-temperature @temperature input-type)
            }]
  )

(defn component []
  [:div
   (atomic-input temperature 'celsius)
   " Celsius = "
   (atomic-input temperature 'fahrenheit)
   " Fahrenheit"
   ])
```

### The Rest

I did some other problems, but felt unmotivated to write more. You can find the
code on my GitHub.

## Conclusion

I can say nothing but Clojure does live up to its reputation as a beautiful
language. The hoops (research on libraries, finding a decent editor) that people
will have to jump through to work with it is another story however. Did I
mention that its syntax (LISP, s expression) looks like Alien's writing to
unfamiliar folks? Not to mention the non-existent market job.

I still love it though. Nubank, which is a start up on fin-tech built and scaled
itself with Datomic and Clojure, and recently acquired Cognitect, which is the
company behind Datomic and Clojure. For now, I believe in the "positive" future
which people copy Nubank, and Clojure gets popular with Datomic. In a "negative"
future which Clojure remains a niche, or worse, dies along with Nubank,
immutable data and functional programming will stick with me, and transform me
into a better programmer anyway.
