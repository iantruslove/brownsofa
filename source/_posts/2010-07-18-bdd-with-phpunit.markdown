---
author: Ian
comments: true
date: 2010-07-18 05:43:14+00:00
layout: post
slug: bdd-with-phpunit
title: More natural BDD with PHPUnit
alias: blog/archives/83/index.html
wordpress_id: 83
categories:
- programming
tags:
- bdd
- php
- phpunit
- Programming
- tdd
---

I've always been a little jealous of RSpec.  Those Ruby kids and their natural language BDD testing and plain text stories...  It always just seems more awkward any other way.  Cucumber looks pretty cool but getting it to play nicely with PHP is a little over my KISS threshold today.

<!-- more -->

Well, PHPUnit does have an alternative in its [Story extension](http://www.phpunit.de/manual/3.5/en/behaviour-driven-development.html).  Look at the docs and it's simple enough to put into effect, and works nicely as part of a bigger set of unit tests.  But check out how those scenarios are constructed - the argument passing just doesn't seem elegant, and run `phpunit --story <file>` on the 2-arg puppies and it's not pretty at all.  Here's a simple scenario straight from the manual:

    
        /**
         * @scenario
         */
        public function scoreForOneStrikeAnd3And4Is24()
        {
            $this->given('New game')
                 ->when('Player rolls', 10)
                 ->and('Player rolls', 3)
                 ->and('Player rolls', 4)
                 ->then('Score should be', 24);
        }


So how can we write PHPUnit scenarios to read more like natural language?   Well, first thing to go are the arguments.  I'd rather write:

    
        /**
         * @scenario
         */
        public function scoreForOneStrikeAnd3And4Is24()
        {
            $this->given('New game')
                 ->when('Player rolls 10')
                 ->and('Player rolls 3')
                 ->and('Player rolls 4')
                 ->then('Score should be 24');
        }


Not a huge change, but it takes a little more work to finish the plumbing.  Just looking at implementing the when clause, the original version read

    
        public function runWhen(&$world, $action, $arguments)
        {
            switch($action) {
                case 'Player rolls': {
                    $world['game']->roll($arguments[0]);
                    $world['rolls']++;
                }
                break;
    
                default: {
                    return $this->notImplemented($action);
                }
            }
        }


and ends up becoming something like:

    
        public function runWhen(&$world, $action, $arguments)
        {
            switch(true) {
                case preg_match('/^Player rolls \d+$/', $action): {
                    $world['game']->roll( preg_replace('/\D/', '', $action) );
                    $world['rolls']++;
                }
                break;
    
                default: {
                    return $this->notImplemented($action);
                }
            }
        }


I feel like the added complexity of the helper code is compensated by the much more natural and flexible way you can construct the scenarios.  Start adding multiple arguments to the clauses and it's even nicer.
