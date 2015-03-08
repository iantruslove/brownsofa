---
layout: post
title: "Querying Elasticsearch from Hive"
date: 2015-03-08 16:56
comments: true
published: true
categories:
- databases
- hadoop
---

I wanted to query ES from Hive. They seem like really interesting
complementary tools.

## Setup<a id="sec-1-1"></a>

Obviously I needed Elasticsearch and Hive to be running. The former is
fairly straightforward, the latter requires a basic Hadoop stack.

I have been playing with the Cloudera distribution, and for a cluster
it is great, but for this I don't need the (significant) overhead.

So I found a [Vagrant setup](https://github.com/congdang/vagrant-hadoop-hive) that did a bunch of what I needed, fixed a
bunch of issues, added Elasticsearch, and pushed up
[hadoop-hive-elasticsearch-vm](https://github.com/iantruslove/hadoop-hive-elasticsearch-vm) - a single-node ES and Hive cluster.

## What I'm trying to accomplish<a id="sec-1-2"></a>

-   ES aggregations are cool, but I have definitely found use cases
    where they don't do all that I need.
-   I'm trying to do things like calculating [TF-IDF](http://en.wikipedia.org/wiki/Tf%25E2%2580%2593idf) type statistics for
    documents and groups of documents.
-   The tricky thing with aggregations was stuff like summing TFs across
    a whole account.
-   These kinds of things are really easy with SQL, so it seems like
    ES + Hive (or later ES + Spark) would be a very powerful
    combination.

## Add some data to Elasticsearch<a id="sec-1-3"></a>

I have a host `hadoop-hive-elasticsearch` running standard
installations of Elasticsearch and Hive, as close to out of the box as
one would reasonably get.

Create an index in Elasticsearch:

```sh
curl -v -XPUT http://hadoop-hive-elasticsearch:9200/idx_foo
```

PUT the mapping:

```sh
    curl -v -XPUT 'http://hadoop-hive-elasticsearch:9200/idx_foo/_mapping/analysis?ignore_conflicts=true' -d '{
      "analysis" : {
        "properties" : {
          "analyserVersion" : {
            "type" : "string",
            "store" : "true",
            "index" : "not_analyzed"
          },
          "analysisTime" : {
            "type" : "date",
            "format" : "date_optional_time"
          },
          "documentIdentifier" : {
            "type" : "string",
            "store" : true,
            "index" : "not_analyzed"
          },
          "topics" : {
            "type" : "string"
          }
        }
      }
    }'
```

Use the bulk API to add some docs:

```sh
    curl -v -XPOST http://hadoop-hive-elasticsearch:9200/_bulk -d '
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:00Z", "documentIdentifier": "doc1", "topics": ["business", "collaboration", "lifehacks", "email"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:01Z", "documentIdentifier": "doc2", "topics": ["sports", "basketball"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:02Z", "documentIdentifier": "doc3", "topics": ["lifehacks", "wardrobe"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:03Z", "documentIdentifier": "doc4", "topics": ["sports", "snowboarding", "mountains"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:04Z", "documentIdentifier": "doc5", "topics": ["outdoors", "hiking", "mountains", "lakes"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:05Z", "documentIdentifier": "doc6", "topics": ["business", "sales", "cars"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:06Z", "documentIdentifier": "doc7", "topics": ["jeep", "cars", "outdoors", "off-roading"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:07Z", "documentIdentifier": "doc8", "topics": ["sports", "fly fishing", "lakes"] }
    { "create": { "_index": "idx_foo", "_type": "analysis" } }
    { "analyserVersion": "0.0.1", "analysisTime": "2015-03-07T16:12:08Z", "documentIdentifier": "doc9", "topics": ["programming", "clojure", "databases"] }
    '
```

So now the ES instance has an index called `idx_foo`, with a handful
of documents belonging to one document type.

## Hive Time<a id="sec-1-4"></a>

Create a database in Hive to use

    CREATE DATABASE es_test;
    USE es_test;

Create a table. It needs to be an external table and pull data in from
Elasticsearch.

    -- DROP TABLE foo_analysis_doc_ids;

    CREATE EXTERNAL TABLE foo_analysis_doc_ids (
      analyser_version STRING,
      analysis_time TIMESTAMP,
      doc_id STRING,
      topics ARRAY<STRING>
    )
    STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
    TBLPROPERTIES('es.resource' = 'idx_foo/analysis',
                  'es.mapping.names' = 'doc_id:documentIdentifier, analysis_time:analysisTime, analyser_version:analyserVersion');

It turned out that mapping column names is useful.

Querying ES from Hive:

    SELECT * FROM foo_analysis_doc_ids;

    0.0.1   2015-03-07 16:12:00     be87f0b5-faad-4513-827c-15a635844eaa    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:01     doc2    ["sports","basketball"]
    0.0.1   2015-03-07 16:12:05     doc6    ["business","sales","cars"]
    0.0.1   2015-03-07 16:12:06     doc7    ["jeep","cars","outdoors","off-roading"]
    0.0.1   2015-03-07 16:12:00     b0289460-f6ef-4309-911a-d27e52155ae7    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:02     doc3    ["lifehacks","wardrobe"]
    0.0.1   2015-03-07 16:12:07     doc8    ["sports","fly fishing","lakes"]
    0.0.1   2015-03-07 16:12:03     doc4    ["sports","snowboarding","mountains"]
    0.0.1   2015-03-07 16:12:08     doc9    ["programming","clojure","databases"]
    0.0.1   2015-03-07 16:12:00     doc1    ["business","collaboration","lifehacks","email"]
    0.0.1   2015-03-07 16:12:04     doc5    ["outdoors","hiking","mountains","lakes"]
    Time taken: 0.038 seconds, Fetched: 11 row(s)

Sweet. You can even see here there are a couple of documents with UUID
identifiers that I had in ES from experiments along the way that
didn't quite get written up here.

So let's do some SQL-y things with Elasticsearch data: retrieve a list of unique topics.

    SELECT DISTINCT(topic_list.t) AS topic
    FROM (SELECT explode(f.topics) AS t FROM foo_analysis_doc_ids AS f) AS topic_list
    ORDER BY topic;

    Query ID = vagrant_20150308155959_a966862b-4aa0-489d-867f-74b880a952a1
    Total jobs = 2
    Launching Job 1 out of 2
    Number of reduce tasks not specified. Estimated from input data size: 1
    In order to change the average load for a reducer (in bytes):
      set hive.exec.reducers.bytes.per.reducer=<number>
    In order to limit the maximum number of reducers:
      set hive.exec.reducers.max=<number>
    In order to set a constant number of reducers:
      set mapred.reduce.tasks=<number>
    Starting Job = job_201503081508_0007, Tracking URL = http://localhost:50030/jobdetails.jsp?jobid=job_201503081508_0007
    <SNIP>
    Stage-Stage-1: Map: 5  Reduce: 1   Cumulative CPU: 8.57 sec   HDFS Read: 190090 HDFS Write: 617 SUCCESS
    Stage-Stage-2: Map: 1  Reduce: 1   Cumulative CPU: 1.86 sec   HDFS Read: 1113 HDFS Write: 181 SUCCESS
    Total MapReduce CPU Time Spent: 10 seconds 430 msec
    OK
    basketball
    business
    cars
    clojure
    collaboration
    databases
    email
    fly fishing
    hiking
    jeep
    lakes
    lifehacks
    mountains
    off-roading
    outdoors
    programming
    sales
    snowboarding
    sports
    wardrobe
    Time taken: 41.09 seconds, Fetched: 20 row(s)

In doing the previous I ran into a problem that looked like the
elasticsearch-hadoop jar wasn't distributed to the hadoop cluster.

-   Having the JAR in the Hive `lib` dir didn't work.
-   Putting the JAR into the Hadoop `lib` dir didn't work.
-   Using the Hive shell's `ADD JAR <file-path>` command DID WORK!
-   This is something that will need to be addressed systematically for
    a real-world scenario - something like distrbuting the JAR via HDFS.

Note the time taken to do that - 40 seconds. It ended up firing two
map-reduce jobs, one for the `explode`, and one for the `ORDER
BY`.

For local testing, there's a local mode to help:

    SET mapred.job.tracker=local;

The same query then takes 4 seconds.

Now, how about something like "what's the mapping of topics to
documents?" This has a [CTE](https://cwiki.apache.org/confluence/display/Hive/Common%2BTable%2BExpression), a `SELECT DISTINCT`, a `JOIN` - definitely
interesting stuff to layer on top of an Elasticsearch index.

    WITH topic_explosion AS (SELECT explode(f.topics) AS t FROM foo_analysis_doc_ids AS f),
    topics AS (SELECT DISTINCT(t) AS topic FROM topic_explosion)
    SELECT topics.topic, f.doc_id
    FROM foo_analysis_doc_ids f, topics
    WHERE array_contains(f.topics, topics.topic)
    ORDER BY topic, doc_id;

    Total MapReduce CPU Time Spent: 0 msec
    OK
    basketball      doc2
    business        b0289460-f6ef-4309-911a-d27e52155ae7
    business        be87f0b5-faad-4513-827c-15a635844eaa
    business        doc1
    business        doc6
    cars    doc6
    cars    doc7
    clojure doc9
    collaboration   b0289460-f6ef-4309-911a-d27e52155ae7
    collaboration   be87f0b5-faad-4513-827c-15a635844eaa
    collaboration   doc1
    databases       doc9
    email   b0289460-f6ef-4309-911a-d27e52155ae7
    email   be87f0b5-faad-4513-827c-15a635844eaa
    email   doc1
    fly fishing     doc8
    hiking  doc5
    jeep    doc7
    lakes   doc5
    lakes   doc8
    lifehacks       b0289460-f6ef-4309-911a-d27e52155ae7
    lifehacks       be87f0b5-faad-4513-827c-15a635844eaa
    lifehacks       doc1
    lifehacks       doc3
    mountains       doc4
    mountains       doc5
    off-roading     doc7
    outdoors        doc5
    outdoors        doc7
    programming     doc9
    sales   doc6
    snowboarding    doc4
    sports  doc2
    sports  doc4
    sports  doc8
    wardrobe        doc3
    Time taken: 5.945 seconds, Fetched: 36 row(s)

## Looking ahead&#x2026;<a id="sec-1-5"></a>

I can see some interesting experiments ahead:

-   How can I effectively work across multiple ES indexes? E.g. if
    there's one index per account, and analyses go into that, how can
    aggregate statistics be calculated across accounts?
-   Hive uses Hadoop MR to distribute work. This is slow. Would [Pig](http://pig.apache.org/) do
    any better? How about [Spark](https://spark.apache.org/)? How would the three perform at scale?




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
