---
author: Ian
comments: true
date: 2012-08-29 01:12:50+00:00
layout: post
alias: blog/archives/367/index.html
slug: application-build-tools
title: Application Build Tools
wordpress_id: 367
categories:
- agile
- java
- javascript
- programming
- ruby
- tools
- version control
---

In my experience, there’s a common set of _types_ of tools that every project needs. When it’s time to start up a new project, here’s an outline of the things to consider when putting together a new tool chain, and some examples of good tools I’ve seen recently.


## Source Control


Well, it’s not really part of the build tools, but if you’re not using [Git](http://git-scm.com/) or [Bazaar](http://bazaar.canonical.com/en/) or [Mercurial](http://mercurial.selenic.com/) or [Subversion](http://subversion.tigris.org/) or [Perforce](http://www.perforce.com/) (or even [CVS](http://www.nongnu.org/cvs/), and I’d even give you it if you were using [TFS](http://www.microsoft.com/visualstudio/en-us/products/2010-editions/team-foundation-server/overview)) or some other source code repository, you should get yourself some source control before anything else.

The latest tools (Git, Bazaar and Mercurial) are all Distributed Version Control Systems, wherein there is not necessarily a single centralized repository, but where any given clone of the repository contains the full history. They have powerful branching and merging capabilities, eclipsing those in SVN which in turn eclipsed those in CVS. If you are working in a team situation, particularly a distributed team, consider one of the DVCS tools.

Social coding sites like [GitHub](http://github.com) (Git-based) and [Bitbucket](https://bitbucket.org/) (Git and Mercurial) are supplanting the more traditional online repositories such as [SourceForge](http://sourceforge.net/) because they’re slick, free to get started, have great features, and are easy to use.

There’s just no excuse…


## Unit Tests


I’m just going to assume that you write unit tests for your code. In fact, I’m just going to say that everyone’s doing [TDD](http://www.amazon.com/Test-Driven-Development-By-Example/dp/0321146530). If you’re not doing TDD, perhaps consider it for your new project - it’s the easiest time to start. If you’re adamantly not, this should all still apply.

At the very least you need a set of basic assertions for your language of choice, and a way to run your app’s test harness. It’s not hard to roll your own, but you are likely reinventing a wheel.

More sophisticated unit test frameworks basically can give you more assertions, more structures for organizing your tests, and better integration with your platform and development environment.

Whether simple or complex, the most important thing when evaluating these frameworks is readability of code, and of test’s failure messages.

You will be generating and refactoring lots of test code, and the test frameworks can make or break whether your test code is readable. What constitutes “readable” is more personal, but read through some test code written with the framework and see how clear the _intent_ of the tests are without having to trawl the API docs.

You will also be writing tests that are in some sense designed to fail. When they do, the error message displayed by the test runner needs to be clear and informative, and either help you verify your assumptions about the as-yet unwritten code, or to quickly find the failing test and trace back through the call stack.

Simple but good frameworks I’ve used include [UnitTest++](http://unittest-cpp.sourceforge.net/) for C++, and the tools built in to the Python standard libraries (wonderfully introduced in the [Dive Into Python](http://www.diveintopython.net/unit_testing/index.html) book).

The [xUnit](http://www.martinfowler.com/bliki/Xunit.html) family of test libraries originated in a design by Kent Beck, and were made ubiquitous by [JUnit](http://www.junit.org/) and [NUnit](www.nunit.org). They are well understood and [well documented](http://www.amazon.com/xUnit-Test-Patterns-Refactoring-Code/dp/0131495054), and are surely available in just about every language imaginable.

I recommend a more [behavior-driven](http://dannorth.net/introducing-bdd/) approach. It encourages a valuable switch in focus; making your code right is made subservient to making the right code. My favorite unit test libraries adopt this style of testing: Ruby’s [RSpec](http://rspec.info/), [Jasmine](https://github.com/pivotal/jasmine/wiki) for JavaScript, and the [Hamcrest matchers](http://code.google.com/p/hamcrest/wiki/Tutorial) on the JVM all make me really happy. [Expectations](https://github.com/jaycfields/expectations) for Clojure is both simple and behavioral.


## Integration/Acceptance Tests


Continuing the theme of “write the right code”, it’s important to make sure the whole system meets whatever needs are set out. At some point you need to stick together your a number of the various bits of your app and make sure they at least look like the whole thing might work.

Acceptance tests should be written in the language of your problem domain, and ideally will be readable by your pointy-hair types. They should express what the system should be doing. See any of [Gojko Adzic](http://gojko.net/)’s work. Since they are high level and written in terms of business needs, pick a tool that supports this. [Cucumber](http://cukes.info/) and [FitNesse](http://fitnesse.org/) both have implementations in many languages, and have various plugins or extensions to support development of different types of application. I currently use [Watir WebDriver](http://watirwebdriver.com/) for web UI testing; tests are written in Ruby, and because of its WebDriver roots it’s very easy to test in multiple browsers using Selenium Grid service providers like [Sauce Labs](http://saucelabs.com/). Other tools include [Capybara](https://github.com/jnicklas/capybara) (which I believe is somewhat tailored to Rails apps) and [Selenium](http://seleniumhq.org/).

_One note regarding Selenium IDE and perhaps other similar tools: I strongly recommend avoiding the GUI test recorder. It is tempting to think it will save time and give everyone the ability to write acceptance tests; you end up with a pile of unmaintainable XPath expressions and brittle tests that none of your developers want to touch with a bargepole. This is my experience at least; YMMV. Working together as a cross-functional team is the right way to build the tests anyway._

Integration tests allow you to compose a small subset of your entire system, and ensure that the independent pieces work correctly with each other (a unit test, in comparison to integration tests, ideally should not instantiate more than one of your system’s classes. See the “I” of FIRST). I go back and forth on this one, but my current approach is to use my unit test framework to write the tests, but explicitly and intentionally separate the integration tests from the unit tests.


## Mock Objects


Remember the [FIRST](http://pragprog.com/magazines/2012-01/unit-tests-are-first) acronym for unit tests? “F”, “I” and I believe to some degree, “T” are dependent on being able to substitute the collaborators your code under test interacts with for some type of stand-in. Whether you only need a simple test double, or you want a full-on programmable mock, [these test-time objects](http://martinfowler.com/bliki/TestDouble.html) are easier to make if you enlist the help of a mock object library.

Examples. [So many](http://lmgtfy.com/?q=mock+object+framework) examples… [Sinon](http://sinonjs.org/) for JavaScript rocks. [Mockito](http://code.google.com/p/mockito/) is my go-to in Java.

If you haven’t used mocks before, this is one case I’d absolutely recommend you write these types of objects from scratch the first time. The frameworks will absolutely help you later on, but your knowledge of what type of test stand-in to use, when, and why, will be greatly assisted if you roll your own for a while.


## Static Analysis


This is an interesting area, one which appeals to the metrics nerd in me. It is possible to have a program inspect your source code and perform various types of analyses. Examples I’ve used include:



	
  * checking for basic syntactical errors (javac, gcc, g++, …)

	
  * more subtle syntactical and best-practice conformance ([JSHint](http://www.jshint.com/), [JSLint](http://www.jslint.com/), [Checkstyle](http://checkstyle.sourceforge.net/checks.html))

	
  * conformance with team coding conventions ([FXCop](http://blogs.msdn.com/b/codeanalysis/), which I think now ships with Visual Studio)

	
  * inclusion of appropriate license headers ([maven-license-plugin](http://code.google.com/p/maven-license-plugin/))

	
  * scanners for common web security vulnerabilities ([RIPS](http://sourceforge.net/projects/rips-scanner/))

	
  * code coverage of your unit tests ([EMMA](http://emma.sourceforge.net/), which strictly speaking isn’t a static analyzer)

	
  * calculating your code’s cyclomatic complexity ([JavaNCSS](http://javancss.codehaus.org/), though I haven’t used it)


It’s possible to get an incredible amount of information about your code, some more useful than others.


## Documentation


Agile or not, there is a need for some amount of (valuable) documentation. As a somewhat responsible software developer, I think there are a few high value, low cost artifacts that I should take responsibility for: auto-generated source code documentation, developer READMEs, and API documentation. I do most of my work on web systems, and I realize my opinions tend towards those types of products.


### Auto-generated source code docs


By this I mean things like [JavaDoc](http://www.oracle.com/technetwork/java/javase/documentation/index-jsp-135444.html) comments. These are simple formatting rules applied in comment blocks to mark up the content of the comment. The source code can be post-processed, documentation can be extracted and reprocessed into immediately publishable HTML, PDF, whatever. Some IDEs also use the documentation comments to enrich the context-sensitive help and autocompletion tools. There are many implementations of this type of tool, including [Doxygen](http://www.doxygen.org) (supports a multitude of languages), [RDoc](http://rdoc.sourceforge.net/) and [Yard](http://yardoc.org/) for Ruby, and [PHPDocumentor](http://www.phpdoc.org/). These tools are often particularly powerful in strongly typed languages, since the documentation parser can easily parse the source code to determine types and hierarchies.

From the JavaScript community, a less-formal type of tool has emerged, exemplified by [Docco](http://jashkenas.github.com/docco/). In these tools, few (if any) assumptions are made about the markup content of the comments, but the output rendering of the documentation places the comments in the left column, lined up with the code they annotate in the right hand column. Docco will convert any [Markdown](http://daringfireball.net/projects/markdown/) content to marked up HTML in the output too. The [Underscore documentation](http://documentcloud.github.com/underscore/docs/underscore.html) is generated by Docco, and it’s useful and looks good.

For those who share my general distaste of commented code, it is true that the documentation will only be as good as the comments that you write; if your comments aren’t maintained as your source code changes, your documentation will be confusing at best, and horribly wrong at worst. If you’re going to do this, there’s a certain commitment that’s required.


### Developer READMEs


Nothing groundbreaking here. Look at most projects’ home page on GitHub, and the README.md file is presented front and center. Put whatever you want in it, but make sure you cover at least the items below. Think about what a new developer getting started on the project would need (or yourself in three months time when you’ve inevitably shifted everything unnecessary out of your brain, including how _this_ project works):



	
  * What dependencies you need to install by hand

	
  * How to install the dependencies your project installs

	
  * How to build the software

	
  * How to run the software

	
  * Any special magic one needs to know in order to write new code or fix bugs


Plain text files have a nice low barrier to entry, and Markdown adds a little sugar and options for post-processing for a low cost.


### API Documentation


This section assumes you’re writing at least a little bit of code that someone else will re-use - whether it’s a RESTful web service or a redistributed code library.

Your API is a contract - by releasing an API you implicitly agree to certain reasonable conventions around that API. These include that the code does what it says it will do, and by extension, what the documentation says the code will do. You also need to provide users of your code some indication of how to use it. If you think you write perfectly understandable code that doesn’t need documentation, well, perhaps just think of the rest of us who haven’t attained your level of coding mastery.

First, explain _how_ your code should be used. Coherent APIs have a philosophy, a way of doing things. Introduce the neophyte to that way of thinking. Succinct, complete, accurate examples go a long way.

Second, document the precise inputs and outputs for all components of the API, and any requirements for use that a programmer must manage themselves. This is your contract. When you are changing the code, be aware of when you are breaking the contract in a way that is not backwards-compatible; you will need to write a new contract. See Versioning for a tad more discussion.

Tools to generate this type of documentation vary widely. You may just mark up your source code with a little extra documentation, then generate and publish your docs from that. RESTful APIs need a little more work, and some sources would indicate that all one needs to publish are some schema and the media types. I think a little more effort allows other developers to use your API more easily, and this is surely a large reason for using REST in the first place. [IODocs](https://github.com/mashery/iodocs) has a great approach: a terse JSON-formatted input file describes the API in a very concise way. IODocs then generates a documentation website around the API description, automatically generating tools to allow you to interact with the API in real time. Hands-on is often an effective way to learn.


## Dependency Management


Ooh, sounds like fun, doesn’t it! Well, it’s not. It can go horribly, awfully wrong - search for DLL hell, JAR hell, Gem hell, NPM hell, pick your poison. Here’s the issue: your code depends on Library A v5, and on Library B v3. Library A depends on Library B v4. What version of Library B do you need? Gah.

Transitive dependency management, artifact caching, searching for and resolving artifacts. Things such as these have been taken care of by the smart people behind [Maven](http://maven.apache.org/), [Ant](http://ant.apache.org/), [RubyGems](http://rubyforge.org/projects/rubygems/), [NPM](https://npmjs.org/), and many other tools. If you’re going to be writing code any more complex than “Hello World”, you will likely benefit from one of these tools.


## Versioning


This one’s easy: use [Semver](http://semver.org/) and all the rules and conventions therein. Its lack of ambiguity makes it so easy to adopt.


## Tools to wrap it all up


You need something to tie all of the above up in a neat bow. You could write some shell scripts, but there are better options. It also seems that whilst there aren’t huge numbers of tools in this area, the pace at which new tools are being released is increasing rapidly.

If you’re a C programmer, you’ve written a `makefile` for [make](http://www.gnu.org/software/make/). `makefile`s are a declarative configuration file that describes how your source code needs to be compiled, assembled and linked. Without `make`, C programs beyond one or two source files are too much effort.

Fast-forward 7 years after make’s inception, and [Ant](http://ant.apache.org/) first hits the scene. Ant is a tool primarily designed to compile and build Java applications, but it can be used as a more genera-purpose tool. It uses XML files to describe how the project should be built, but again in a declarative manner.

Soon after Ant was released, [Maven](http://maven.apache.org/) hit the interwebs. Maven also uses XML to describe the project configuration (typically `pom.xml`), but in exchange for its highly opinionated idea of how source code should be laid out and named, Maven allows you to forego a huge amount of boilerplate build file configuration. One common criticism of Maven, however, is that it’s really hard to get Maven to do something non-standard. Writing a plugin isn’t usually an effective solution; I’ve seen plenty of solutions where Maven calls out to an Ant task to do something hacky.

It’s clear that depending on what you want to do, there’s going to be differing sweet spots of declarative vs imperative solutions, and convention versus configuration.

Some other notable tools I’ve run into and like are [Grunt](https://github.com/cowboy/grunt) for JavaScript, [Rake](http://rubyforge.org/projects/rake/) for Ruby, [Gradle](http://www.gradle.org/) for various JVM languages, and [Leiningen](https://github.com/technomancy/leiningen) for Clojure. Each has its own set of strengths and trade-offs. There are still certain Java tasks I might use Maven for, and I might also recommend learning how to use Ant as a handy fallback for weird situations, using [Ivy](http://ant.apache.org/ivy/) to add dependency management capabilities. My go-to replacement for Ant, however, is Rake. The configuration files (`rakefile`s) are written in Ruby, but Rake is easily applicable to any task. There’s a nice set of declarative tasks available, and at any time you can drop into vanilla Ruby to do whatever needs doing.


## Other Considerations




### CI Integration


That’s right, “Continuous Integration Integration”. How do all of these things fit into your CI system? Having your CI server run the entire build and release process is so valuable - in order to be confident enough to do this automatically you need a high degree of confidence in your tools, which implies that your tools are likely solid, powerful, and get the job done. These are all good things to have, even if you don’t [continuously deploy](http://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912/) your software [a dozen times a day](http://code.flickr.com/#foot).

Think about how the output from the various tools can be parsed by your CI - JUnit XML format is a common output format for test runners whether or not it has anything to do with Java.

Take advantage of the visualization tools and plugins for your CI. They’re fun and pretty, but sometimes surprisingly useful: I have dropped a test result history plot (i.e. largely green, trending up-and-to-the-right) into a presentation and the pointy hair types definitely grokked that.


## Conclusion


In short: myriad options. Chasing down and using the latest cool library is often unproductive, but starting up a new project is a great time to improve your tool chain and improve your productivity or quality, increase your visibility into your code, get faster feedback on errors, improve your code in general, or just geek out on graphs and charts. Take some time to learn what’s out there, take inspiration from other great tools, and perhaps roll your own tool to rule them all.

Drop me a comment and let me know what awesome tools I have missed - I’ll take a look for the next project.
