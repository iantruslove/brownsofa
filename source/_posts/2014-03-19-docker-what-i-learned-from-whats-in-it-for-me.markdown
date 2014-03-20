---
layout: post
title: "Docker: what I learned from working out what's in it for me"
date: 2014-03-19 20:31
comments: true
categories:
- docker
- tools
- devops
---

{% img left /images/docker-what-i-learned-from-whats-in-it-for-me/docker-logo.png 'Docker logo: from www.docker.io' 'Docker whale logo' %}

I recently spent some time researching and using [Docker][], specifically
with a view of how it would help me develop a scalable SOA. I gave a
couple of [presentations](/presentations/#docker_whats_in_it_for_me)
on some of the basics I picked up.

[Docker]: http://www.docker.io

In the spirit of the order of the talk, I concluded that Docker had
some really interesting use cases:

*For developers*: Docker is great for pulling in application
 dependencies.  For example, starting up an ElasticSearch server is as
 simple as two commands at the shell.

*For devops*: Docker is great for getting developers to package up
 apps with all their dependencies into a declaratively constructed,
 directly deployable, entirely repeatable artifact.

*For architects*: Docker is entirely in line with the tenets of
 [Twelve Factor Apps][12fa], and can easily be used in various
 infrastructure scenarios.

[12fa]: http://12factor.net/

I didn't have time in the talk to cover some of the interesting
details I discovered along the way, so I'll cover them here instead:

* Structure Dockerfiles hierarchically
* Use Versions in Tags
* Check Dockerfiles into GitHub
* Running in the foreground
* Inter-container communication and web services APIs

<!-- more --> 

---

## Structure Dockerfiles hierarchically

It seems like premature optimization, but I found distinct
develop-build-test cycle time improvements when I paid attention to
the image tree Docker maintains.

My earlier attempts to iteratively build two Docker images, each from
their own Dockerfile, was unnecessarily slow.  Each of the Dockerfiles
was large and monolithic, and the commands in each of the Dockerfiles
had no real order.  Each of the images had some similarities
(Ubuntu-based, Oracle Java), and some notable differences (one runs
ElasticSearch with Supervisor, the other packages up and runs a
Clojure application from GitHub sources).  Changes in the shared code
in one were replicated in the other, and a change early in one
Dockerfile meant rebuilding the entire image, not just incremental
changes in the later "states" Docker manages.

The approach I moved to, and what I recommend, is nothing novel. Since
each docker command run within a container yields a new container with
that altered state, the best way to minimize build-time churn is to
keep the base images stable. To wit: develop a hierarchy of images
whose base layers are the most stable, and where changes occur as
close to the leaf nodes of the tree as possible.

For illustration, here is the hierarchy of some images I built
according to that scheme:

```sh
% docker images -t
├─511136ea3c5a Virtual Size: 0 B
│ ├─1c7f181e78b9 Virtual Size: 0 B
│ │ └─9f676bd305a4 Virtual Size: 178 MB Tags: ubuntu:13.10, ubuntu:saucy
│ │   └─55048c64aafb Virtual Size: 178 MB
│ │     └─b45b04c61d37 Virtual Size: 181.3 MB
│ │       └─cfd0a2113031 Virtual Size: 266.1 MB Tags: iant/base:0.1.0
│ │         └─873825ff7806 Virtual Size: 266.1 MB
│ │           └─c5ec6d485e19 Virtual Size: 332.1 MB Tags: iant/common:0.1.0
│ │             └─12284038f771 Virtual Size: 332.1 MB
│ │               └─8a7772c4e933 Virtual Size: 388 MB
│ │                 └─59dc3b6ad0a8 Virtual Size: 388 MB
│ │                   └─f3b21f3b00fe Virtual Size: 391.4 MB
│ │                     └─8afce624e6a1 Virtual Size: 394.2 MB
│ │                       └─88acfebd9fa9 Virtual Size: 808.3 MB Tags: iant/java:0.1.0
│ │                         └─f829acd28896 Virtual Size: 808.3 MB
│ │                           └─add88843e338 Virtual Size: 808.4 MB
│ │                             └─b29cab6d5c13 Virtual Size: 808.4 MB
│ │                               └─f782125c3355 Virtual Size: 822.1 MB Tags: iant/clojure:0.1.0
│ │                                 └─63cd57a9fe58 Virtual Size: 822.1 MB
│ │                                   └─d33ab8f97b66 Virtual Size: 822.2 MB
│ │                                     └─6dcc5827a7b3 Virtual Size: 822.2 MB
│ │                                       └─472668b960a2 Virtual Size: 822.2 MB
│ │                                         └─38ee3cb17dd7 Virtual Size: 851.3 MB
│ │                                           └─909f8927c47a Virtual Size: 851.3 MB
│ │                                             └─6d5f57d4355e Virtual Size: 851.3 MB
│ │                                               └─7dde5563b9bc Virtual Size: 851.3 MB Tags: iant/search-clojure-archives:0.1.0
...etc
```

The `base` image is simply an updated `ubuntu:saucy` image.  `common`
adds common command line tools (curl etc) to `base`.  `java` is an
image based on `common` with Oracle Java 8 installed; it has a number
more intermediate images each corresponding to all the trickery to get
Java installed
["unattended"](https://github.com/iantruslove/dockerfiles/blob/master/java/Dockerfile#L14).
`clojure` adds basic Clojure development tools, and
`search-clojure-archive` builds on that to retrieve, build and run the
web app.

Your scheme will be different.  I view Java as a fairly base-level
dependency.  Yours might be Ruby, or Nginx.  The point is to push the
rapidly changing configurations to the edges of the hierarchy, and get
considerably quicker build and test times.

## Use Versions in Tags

When building Dockerfiles into images, make consistent use of `docker
build -t <tag>`.  The documentation isn't superbly lucid, but you are
encouraged to structure the "tag" used here as
`<username>/<image>:<version>`.  The `version` identifier can be
anything, but `latest` has some semantic significance.

I picked a semver scheme, I would just recommend being consistent, and
keeping Dockerfile inline documentation and published image versions
consistent.

## Check Dockerfiles into GitHub

Because why not?

Whilst finding ways to solve some problem or accomplish some goal, I
found looking at others' Dockerfiles very useful.  My issues were only
very slightly unique, the problems had generally all been solved by
others.

Quinten Krijger was kind enough to have put a bunch of Dockerfiles in
[his repo](https://github.com/Krijger/docker-cookbooks), they are
worth a look.

## Running in the foreground

To package your app up into a container (or when writing pretty much
any other Dockerfile) that serves as a single purpose self-contained
application, you quickly find that you need to run your app as a
foreground process, and likely as the default command to run.

If the app container you're creating is based on your own code, this
is entirely within your control.

If the app container is based on a third-party daemonizing service, or
if your container must have additional services running, using
something like [Supervisor][] might help.  There are plenty of
examples of configuring Supervisor, including in
[Docker's own documentation](http://docs.docker.io/en/latest/examples/using_supervisord/).

Before adding too many services into a single image, ask yourself
whether this is really necessary - could these dependencies be
fulfilled by other independent Docker containers, with communication
over the network?

[Supervisor]: http://supervisord.org/

## Inter-container communication and web services APIs

If a good Docker container is single-purpose, then how can you build a
system of containers?  Docker's `docker run --name <name>` and `docker
run --link <name>:<alias>` is designed to help you with just that.

Run a service container with `--name foo` and its exposed ports are
available to be mapped into client containers.  Run a client container
with `--link foo:bar`, and the "foo" container's ports are exposed via
a variety of environment variables within the client, using "bar" as
an identifier.

For example, a client application depends on an ElasticSearch service.
An ES service would typically expose its HTTP API on port 9200.  The
application within the client container can be written to expect the
ElasticSearch dependency to be fulfilled by injection of environment
variables corresponding to the alias you decide to always run it with
("elasticsearch"), so specifically `$ELASTICSEARCH_PORT_9200_TCP_ADDR`
and `$ELASTICSEARCH_PORT_9200_TCP_PORT`.

By running the ElasticSearch container with `--name es`, and the
client with `--link es:elasticsearch`, the service is wired into the
client via the predetermined env vars, whose values are only set at
run time.  Very neat, and
[full documentation for linking containers][1] is on Docker's site.

[1]: http://docs.docker.io/en/latest/use/working_with_links_names/
