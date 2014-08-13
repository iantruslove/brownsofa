---
layout: post
title: "Debugging in Clojure: Thoughts"
date: 2014-07-17 19:03
comments: true
categories:
- programming
- debugging
- tools
- clojure
---

{% img right /images/debugging-in-clojure/bug.jpg 'Bug' 'Bug' %} I
read a [few][1] [posts][2] recently on [r/clojure][] asking about
Clojure debugging tools.  It seemed that those asking the questions
were looking for the kind of tools we have come to expect from IDEs
such as IntelliJ, Eclipse, Visual Studio and their ilk.
It's a familiar question since I asked it myself when I started
learning Clojure.

I think I have come to the same conclusion as other Clojure
programmers in that my short answer to the question is
"there isn't one", or "it's the REPL", and that my long answer is,
well, long.

[1]: http://www.reddit.com/r/Clojure/comments/28udm4/does_clojure_have_a_breakpoint_capable_debugger/
[2]: http://www.reddit.com/r/Clojure/comments/1def3g/starter_tips_for_more_productive_clj_development/
[r/clojure]: http://www.reddit.com/r/clojure

This cryptic answer needs expanding upon, because it's not really
true.  The real answer is that, like in any other language, the best
Clojure debugger is your brain.

This is no less cryptic.

In coming up with the long answer, I found it fits best into
two parts.  This first part covers what I'm defining as debugging, and
my overall approach - my philosophy, perhaps.  The second part will
cover building debuggable systems, and finally get into the debugging
tools themselves.

<!-- more -->

# Contents
* [Scope](#scope)
* [What is "debugging" anyway?](#what)
* [Process](#process)

<a name="scope"></a>

# Scope

{% img right /images/debugging-in-clojure/terminal.png 'Hi-tech coding' 'Hi-tech coding' %} At
work, I am constrained to write Clojure in a terminal (we use remote
tmux) and really, using Emacs.  This is a fine set of constraints,
but my debugging solutions haven't considered tools like Nightcode,
Light Table, La Clojure or other IDE-like tools.  Looking at the
various product websites, point-and-click debugging has different
levels of support, and if that's what you *really* want, perhaps La
Clojure would be a good start.

I also have not yet tried out the [Ritz][] toolchain
([this article][ritz-and-nrepl] has lots of tidbits that are on my
to-try list), nor have I tried [Schmetterling][] (which makes
exception-triggered or breakpoint-triggered stacktrace inspection and
interactive REPLs available through a browser connected to your REPL),
or [Mycroft][].

[Ritz]: https://github.com/pallet/ritz/tree/develop/nrepl
[ritz-and-nrepl]: http://ianeslick.com/2013/05/17/clojure-debugging-13-emacs-nrepl-and-ritz/
[Mycroft]: https://github.com/relevance/mycroft
[Schmetterling]: https://github.com/prismofeverything/schmetterling

This article is concerned with debugging Clojure, in Emacs, using
simple Clojure code, tools and libraries to make life a little easier.
Much of it applies equally to non-Clojure, and non-functional
programming.  Solving bugs in 100k+ LOC Java codebases isn't magically
made tractable by having IntelliJ's debugger.

<a name="what"></a>

# What is "debugging" anyway?

I will define a bug as a deviation between your expectation of, and
your observation of the behavior of a system.

The "system" here comprises your code, the libraries your code uses,
the JVM, OS and all the way on down to the physical hardware (though
you can likely limit your concern to the first two - see "Understand
your assumptions" below).

Debugging is the process of investigating, verifying, isolating the
cause for, and closing the gap between your expectations and your
observations.  You are allowed to change the system; you are also
allowed to change your expectations of the system.

{% img right /images/debugging-in-clojure/chop.jpg 'Chop it up' 'Chop it up' %}
Debugging is a reductionist process: understanding of the whole is
achieved through understanding of the composite parts. The systems we
build are frequently too complex to really comprehensively understand
all at once, but by considering a bug from how it manifests at the
highest level of abstraction, and digging down into the layers of the
system, we can understand enough of the system to understand a bug,
then make informed decisions about how to address it.

Given this fairly straightforward reductionist approach, it is fairly
straightforward to sketch out an algorithm for debugging.

<a name="process"></a>

# Process

Above I laid out the case for writing "code that is easy to exercise
in the REPL, whose behavior is easily observed, and with unit test
coverage".  Here is my "algorithm" for debugging:


1. Identify the area of your system expressing the bug.  You should be
   able to clearly state the desired behavior.
2. Run that subset of your code and observe the behavior; the outcome
   will be different to your expectation.  Ensure you can repeatably
   observe that outcome is different to your expectation. If you can't
   reproduce it, you aren't in a position to fix it.
3. Read the code.  Consider the inputs, and try to understand how it
   generates the output.  Add temporary instrumentation to the code as
   necessary.
4. If the code is too large or complex to understand, divide and
   conquer.  Refactoring may be helpful, even as a temporary
   measure. You must somehow break the problem down into smaller,
   understandable pieces.  Go back to step 2 and repeat for each of
   the smaller pieces.  Proceed when you fully understand how the code
   works.
5. As long as you are always mindful of what your expectation of the
   code is, the problem will at this point become clear, and you need
   to devise a way to fix it.  Remember, the problem might be your
   expectation.

So, the debugging process can - in general - be outlined.  There are
architectural decisions you can make, and tools you can use, in order
to better pose and answer questions about the runtime behavior of your
code. The [second part of this article][part2] <strike>will
cover</strike> covers what a debuggable Clojure-based codebase
would be, and then digs into some Emacs and Clojure tools I have found
useful.

[part2]: /blog/2014/08/03/debugging-in-clojure-tools/

-----

Image credits:

* http://royalbcmuseum.bc.ca/nh-collections/entomology/
* http://blog.envylabs.com/post/62718536773/remote-pairing-and-browser-sharing-with-tmux
* http://modernfarmer.com/2013/12/chop-wood/
