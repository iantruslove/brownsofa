---
author: Ian
comments: true
date: 2012-08-10 04:54:15+00:00
layout: post
slug: leiningen-for-osx-10-5-8
title: Leiningen for OSX 10.5.8
wordpress_id: 358
categories:
- clojure
- java
- jvm
- programming
---

My old MacBook Pro (A1260/MacBookPro4,1/OS X 10.5.8 Leopard) might be getting a little ragged at the edges, might be needing some new memory, but it can still play with the cool kids after fixing a couple of small issues getting [Leiningen](https://github.com/technomancy/leiningen/) (both 1 and 2) to work.  In short, using Java 1.6 fixed the problem.

Download Leiningen in the (well, my) usual manner:

    
    mkdir -p ~/local/bin
    export PATH=~/local/bin:$PATH
    # Leiningen 1 is currently "stable"
    curl -k 'https://raw.github.com/technomancy/leiningen/stable/bin/lein' > ~/local/bin/lein1
    # Leiningen 2 is "preview"
    curl -k https://raw.github.com/technomancy/leiningen/preview/bin/lein > lein
    chmod a+x ~/local/bin/lein*


Problem now is that `lein` doesn't work: I was getting `Caused by: java.lang.ClassNotFoundException: javax/tools/ToolProvider`.

It looked like I was running Java 1.5:

    
    $ java -version
    java version "1.5.0_30"
    Java(TM) 2 Runtime Environment, Standard Edition (build 1.5.0_30-b03-389-9M3425)
    Java HotSpot(TM) Client VM (build 1.5.0_30-161, mixed mode, sharing)


Google searches turned up the info that indicated Java 1.6 was probably available, just not configured. Additionally, re-symlinking `/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK` to 1.6 wasn't enough for me, I also had to update `Current`:

    
    $ cd /System/Library/Frameworks/JavaVM.framework/Versions
    sudo rm Current
    sudo ln -s 1.6 Current



Now, `java -version` was reporting 1.6, and `lein` was all happy.
