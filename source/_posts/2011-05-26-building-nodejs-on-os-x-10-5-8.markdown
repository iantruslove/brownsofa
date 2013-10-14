---
author: Ian
comments: true
date: 2011-05-26 03:58:44+00:00
layout: post
slug: building-nodejs-on-os-x-10-5-8
title: Building NodeJS on OS X 10.5.8
wordpress_id: 334
categories:
- javascript
- programming
---

It seems that I borked xcode, did something bad with my libssl, or just that my system's too out of date to trivially [install](http://sites.google.com/site/nodejsmacosx/), [homebrew](http://shapeshed.com/journal/setting-up-nodejs-and-npm-on-mac-osx/) or [build](http://www.devpatch.com/2010/02/installing-node-js-on-os-x-10-6/) [NodeJS](http://nodejs.org/).  I really want to install Node, so here's what I just did.





I started off downloading and extracting the latest NodeJS source tarball (0.4.2 at time of writing).  After attempting to configure and build, I was getting some complaints out of make:



I was getting some complaints out of `make`:



    
    ../src/node_crypto.cc: In function ‘void node::crypto::InitCrypto(v8::Handle<v8::object>)’:
    ../src/node_crypto.cc:2917: error: ‘SSL_COMP_get_compression_methods’ was not declared in this scope
    Waf: Leaving directory `/Users/Ian/node-latest-install/build'
    Build failed:  -> task failed (err #1): 
    	{task: cxx node_crypto.cc -> node_crypto_4.o}




After a helpful tweek from [@tonymilne](http://twitter.com/#!/tonymilne) I looked at the output of `./configure` slightly more closely: whilst it reported it found `header openssl/crypto.h`, it said that `openssl` was not found.




Getting an updated version of OpenSSL was pretty easy with homebrew:



    
    brew create http://www.openssl.org/source/openssl-0.9.8r.tar.gz
    brew install openssl




It didn't quite fully install though.  It told me "This formula is keg-only, so it was not symlinked into /usr/local", and that



    
    If you build your own software and it requires this formula, you'll need
    to add its lib & include paths to your build variables:
    
      LDFLAGS: -L/usr/local/Cellar/openssl/0.9.8r/lib
      CPPFLAGS: -I/usr/local/Cellar/openssl/0.9.8r/include




Turns out that similar but different problems were solved by [setting options](http://canonical.org/~kragen/compiling-node-on-macos.html) with configure, and running `./configure --help` in the folder with the NodeJS source shows some pertinent options.  I tried



    
    ./configure --prefix=~/local --openssl-includes=/usr/local/Cellar/openssl/0.9.8r/include  --openssl-libpath=/usr/local/Cellar/openssl/0.9.8r/lib




and it seems like it was successful - `node --vars` includes `-DHAVESSL=1`.



