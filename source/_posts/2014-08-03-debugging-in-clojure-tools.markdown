---
layout: post
title: "Debugging in Clojure: Tools"
date: 2014-08-03 18:20
comments: true
published: true
categories:
- programming
- debugging
- tools
- clojure
---

{% img right /images/debugging-in-clojure/butterfly_net.jpg 'Butterfly net' 'Butterfly net' %}

[Part one][] covered the types of things I'm considering when talking
about "debugging", and the process by which I debug code.

This part is about writing debuggable code, and the tools I use or
have seen to debug code: the REPL, println debugging and better,
tracing with Spyscope and clojure.tools.trace, and other assorted good
things.

<!-- more -->

**Contents:**

* Constructing comprehensible systems
  * Easy to exercise
  * Easily observed behavior
  * Unit tests
  * Purity and state
  * Understand your assumptions
* Tools and techniques
  * The REPL
  * println
  * Tracing with Spyscope
  * Macros for debugging let forms
  * Tracing with clojure.tools.trace
  * Fogus's breakpoint macro
  * Other bits

[part one]: http://brownsofa.org/blog/2014/07/17/debugging-in-clojure-thoughts/

# Constructing comprehensible systems

So, debugging is essentially a process for gaining a more in-depth
understanding about how one particular aspect of our system
operates. How can Clojure development in particular help with that,
and how can we make that easier when developing in Clojure?

In short: write code that is easy to exercise in the REPL, whose
behavior is easily observed, and with unit test coverage.

## Easy to exercise

Make it easy to construct valid data to pass to your function.  You
will be greatly assisted by knowing exactly what constitutes valid.
Docstrings, [pre- and post-conditions][pre and post], [assertions][],
[schema][] are all helpful.  Fewer arguments keep it simpler.  Provide
helper functions to help construct complex intermediate data
structures.

[assertions]: http://clojuredocs.org/clojure_core/clojure.core/assert
[pre and post]: http://blog.fogus.me/2009/12/21/clojures-pre-and-post/
[schema]: https://github.com/Prismatic/schema

The usual programming practices have their usual benefits.  Write
clear, well-encapsulated, cohesive-but-not-coupled, singly-responsible
code (functions, in our case).

## Easily observed behavior

Your functions should express a concise piece of behavior, and your
code will be easiest to understand if that behavior is simple, and the
observation is as simple as inspecting the return value.

If the cyclomatic complexity of the function is high, observing the
function's behavior might also require observing the control flow
(yuck).  If the return value of your function needs significant
parsing or decoding to determine whether it operated correctly,
consider whether the function has too many responsibilities.

## Unit tests

I debated for a second whether tests are part of comprehensible
systems, or whether they are tools.  They are an integral part of any
code base, so here they belong.  Tests are your infallible long-term
memory, a playground for experimentation, your sanity check.  Easily
exercised, easily verified functions are easily unit tested.

## Purity and state

Writing pure functions aids the system's comprehensibility enormously.
`f(x) = f(x)` when your program starts up, or when it's crashing, and
whether `f` is running on an EC2 node or in your REPL.  When your code
is a composition of pure functions, reasoning about the code is far
simpler.  This is probably the single biggest contributor to why one
can effectively program in Clojure without a visual debugger: with
pure functions, the state of the system is always external to the
code.

Reality is a little messy.  Applications are infrequently entirely a
composition of pure functions.  You have databases, user input,
network and disk I/O to contend with.  Push the non-pure aspects of
your system to the edges, encapsulate them, and stop your abstractions
from leaking too far.

So, why?  You need to be facile enough with your code and tools so you
can observe the conditions that cause the system to behave
"unexpectedly".  If your code is structured such that it is easy to
run in the REPL (and pure functions generally are), it will be easy to
reproduce the faulty behavior in the REPL (and later in tests), and
understand what the system is doing.

If the corpus of code under examination is too large for immediate
comprehension (in reductionist terms, "emergent behavior"?), divide
and conquer. By looking at the problem into smaller and smaller chunks
(i.e. the layers of composed functions), you exclude possibilities of
where the source of the problem is, until the answer jumps out at you.

## Understand your assumptions

With Occam's Razor in mind, be ready to question your assumptions.
There's always the possibility a bug in Clojure core is the cause of
your problem.  It's not very likely, so don't start there.  The most
likely problem is in your understanding of the problem, or in your
expression of the solution (substitute "your" for whomever wrote the
code), so the assumptions built in here are the first ones to
question, to check, and to verify.


# Tools and techniques

Finally, the Clojure debugging tools easily available to you in Emacs.
They all are
typically employed in [steps 2 and 3][the process] of the debugging algorithm,
and each support one or more of the following:

[the process]: http://brownsofa.org/blog/2014/07/17/debugging-in-clojure-thoughts/#process

* executing code
* observing the result of function calls
* observing intermediate calculations
* observing control flow

## The REPL

It's obvious, but you need to run the code to see what it does.  Use
the REPL to do this.  In Emacs, you will likely be using
[cider-mode]() to fire up a REPL via [Leiningen]().

A selection of cider commands for the uninitiated:

* Use `C-c M-n` or `M-x cider-repl-set-ns` to switch to a source
file's namespace.
* `C-c C-k` or `M-x cider-load-current-buffer` loads the current
source file in the REPL.
* `C-c C-e` or `M-x cider-eval-last-sexp` evaluates an S-expression
and prints the result in the mini buffer.
* `C-c M-p` or `M-x cider-insert-last-sexp-in-repl` copies a
S-expression into the REPL and switches buffers to the REPL ready for
you to hit Enter and evaluate the code.

You may have different key bindings to these.

## println

Showing result values, inspecting intermediate calculation results,
shining a light into execution paths, `println` can do it all.  Some
uses of println I've seen:

Print out a number of values at the same time by throwing them into a
map and printing out the map:

```clojure
;; instead of:
(println "foo:" foo "bar:" bar)
;; do this:
(println "Easy!" {:foo foo :bar bar})   ;; easy to extend
```

Print out what values intermediate symbols in a `let` are bound to:

```clojure
(let [headers         (:headers ring-request)
      header-names    (keys headers)
      ;; The following underscore is a convention for "unused variable"
      _               (println "Headers:" header-names)  ;; <-- this
      header-keywords (map keyword header-names)]
;; etc
)
```

A variant of println is using `clojure.pprint/pprint` or `pr-str` to
print out more complex data structures in a more human-readable
format.

Well, this isn't great.  You have to modify the code to see what's
going on, so you might forget to remove those modifications.  You have
to duplicate code that's already there to inspect values - this
introduces the possibility of typo error.  `println` doesn't
necessarily behave how you would expect in a multithreaded context.
And yeah, it makes you itch a little too.  We can do better...

## Tracing with Spyscope

[spyscope]: https://github.com/dgrnbrg/spyscope

[Spyscope][] is a nice improvement of println debugging. With two
simple additions to your `~/.lein/profiles.clj` it adds three reader
tags:

* print: `#spy/p`
* details: `#spy/d`
* trace: `#spy/t`

Since they are reader tags, you can just pop one of them in front of
the expression you're trying to inspect, and it gets printed out. This improves the example from above

```clojure
(let [headers         (:headers ring-request)
      header-names    #spy/p (keys headers)       ;; <-- print out what header-names is
      header-keywords (map keyword header-names)]
;; etc
)
```

If you're doing println debugging, I definitely recommend the upgrade to Spyscope.

## Macros for debugging let forms

If you find yourself inspecting the values of the bindings within a
let form, I have seen a number of macros to automate the typing.

I have [Gareth Jones's `dlet` macro][dlet] in my user namespace:

[dlet]: http://blog.gaz-jones.com/2012/02/04/debug_let.html

```clojure
(defmacro dlet [bindings & body]
  `(let [~@(mapcat (fn [[n v]]
                       (if (or (vector? n) (map? n))
                           [n v]
                         [n v '_ `(println (name '~n) ":" ~v)]))
                   (partition 2 bindings))]
     ~@body))
```

To see the binding values within the let, just replace any old `let`
in your code with a `dlet`, and the bindings all get printed out:

```clojure
;; Example let form to inspect.  Note use of `dlet` not `let`...
(dlet [response (http/get "http://localhost:8080/api")
       data (json/parse-string (:body response) true)
       accounts (:accounts data)]
      ;; ... some more stuff
      )

;; This is what gets printed to the REPL:
user> response : {:orig-content-encoding nil, :trace-redirects [http://localhost:8080/api], :request-time 7, :status 200, :headers {Server Jetty (7.6.8.v20121106), Connection close, Content-Type application/json; charset=utf-8, Date Sun, 01 Aug 2014 12:34:56 GMT}, :body {
"accounts" : [ { "id" : "53defe4e-5c00-49ae-b0bf-93aa7b57394b",
                 "active" : true
                }, {
                 "id" : "c24e6ab9-c9f3-40d3-ab10-cf28c3bbf1e1",
                  "active" : false
                } ]
}}
data : {:accounts [{:id 53defe4e-5c00-49ae-b0bf-93aa7b57394b, :active true} {:id c24e6ab9-c9f3-40d3-ab10-cf28c3bbf1e1, :active false}]}
accounts : [{:id 53defe4e-5c00-49ae-b0bf-93aa7b57394b, :active true} {:id c24e6ab9-c9f3-40d3-ab10-cf28c3bbf1e1, :active false}]
```

I have also seen a similar outcome using Fogus's [evalive][] library:

[evalive]: https://github.com/tailrecursion/evalive

```clojure
(require '[evalive.core])

(defmacro dlet [bindings & body]
  `(let ~bindings
     (clojure.pprint/pprint (evalive.core/lexical-context))
     ~@body))
```


## Tracing with clojure.tools.trace

The techniques so far all involve modifying the code to inspect
values. In addition to similar tools to the above,
[clojure.tools.trace][] gives you easy ways to trace the result of function
calls without any code modification.

[clojure.tools.trace]: https://github.com/clojure/tools.trace

From the REPL, use the `trace-vars` and `trace-ns` functions to wrap
specific functions or entire namespaces with tracing calls.  In this
example, I define and use a function to square a number, apply a trace
to it, then use it again.  You can see the invocation arguments and
return value for each call:

```clojure
user> (require '[clojure.tools.trace])

user> (defn square [n] (* n n))
#'user/square

user> (map square (range 1 11))
(1 4 9 16 25 36 49 64 81 100)

user> (clojure.tools.trace/trace-vars user/square)
#'user/square

user> (map square (range 1 11))
(TRACE t13642: (user/square 1)
       TRACE t13642: => 1
       TRACE t13643: (user/square 2)
       TRACE t13643: => 4
       TRACE t13644: (user/square 3)
       TRACE t13644: => 9
       TRACE t13645: (user/square 4)
       TRACE t13645: => 16
       TRACE t13646: (user/square 5)
       TRACE t13646: => 25
       TRACE t13647: (user/square 6)
       TRACE t13647: => 36
       TRACE t13648: (user/square 7)
       TRACE t13648: => 49
       TRACE t13649: (user/square 8)
       TRACE t13649: => 64
       TRACE t13650: (user/square 9)
       TRACE t13650: => 81
       TRACE t13651: (user/square 10)
       TRACE t13651: => 100
       1 4 9 16 25 36 49 64 81 100)

;; Turn off all traces:
user> (clojure.tools.trace/untrace-ns 'user)
```

I like `clojure.tools.trace/trace-vars` a lot.  I used [Vinyasa][] to
make it really convenient to use by injecting `trace-vars` into all
namespaces in the REPL, and use it thus:

```clojure
(in-ns 'my.ns.one)

;; Trace some other namespace's fn
(>trace-vars my.ns.two/foo)
```

[Vinyasa]: https://github.com/zcaudate/vinyasa

## Fogus's breakpoint macro

The venerable [Joy of Clojure][tjoc] has the source code for a
complete Clojure breakpoint system. Reading this code was one of those
"Clojure, wow" moments...

[tjoc]: http://joyofclojure.com/

```clojure
(defn contextual-eval [ctx expr]
  (eval
   `(let [~@(mapcat (fn [[k v]] [k `'~v]) ctx)]
      ~expr)))

(defmacro local-context []
  (let [symbols (keys &env)]
    (zipmap (map (fn [sym] `(quote ~sym)) symbols) symbols)))

(defn readr [prompt exit-code]
  (let [input (clojure.main/repl-read prompt exit-code)]
    (if (= input ::tl)
      exit-code
      input)))

(defmacro break []
  `(clojure.main/repl
    :prompt #(print "debug=> ")
    :read readr
    :eval (partial contextual-eval (local-context))))
```

Really, it's just a toy, but isn't it incredible!


## Other bits

### ClojureScript

I saw [the following ClojureScript macro][cljs trace] for defining a
traced function.  I haven't tried it, but it looks interesting:

[cljs trace]: http://www.reddit.com/r/Clojure/comments/2behsz/clojurescript_debugging_tips/cj4mvok

```clojure
(defmacro defntraced
  "Define a function with it's inputs and output logged to the console."
  [sym & body]
  (let [[_ _ [_ & specs]] (macroexpand `(defn ~sym ~@body))
        new-specs
        (map
         (fn [[args body]]
           (let [prns (for [arg args]
                        `(js/console.log (str '~arg) "=" (pr-str ~arg)))]
             (list
              args
              `(do
                 (js/console.groupCollapsed (str '~sym " " '~args))
                 ~@prns
                 (let [res# ~body]
                   (js/console.log "=>" (pr-str res#))
                   (js/console.groupEnd)
                   res#)))))
         specs)]
    `(def ~sym (fn ~@new-specs))))
```

### Ring

Ring services seem like fertile ground for debugging and inspection
problems.  My next post will cover some basic Ring concepts from a
problem-solving point of view.

# Sources and References
In addition to the pages referenced, the following resources have been useful to me:

* http://z.caudate.me/give-your-clojure-workflow-more-flow/ and http://dev.solita.fi/2014/03/18/pimp-my-repl.html (which I think draws upon the former)
* Image credit: http://frostedgardner.blogspot.com/2010/07/summer-weekend-finds.html
