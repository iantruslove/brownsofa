---
layout: page
title: "Presentations and Publications"
date: 2013-10-20 22:03
comments: true
sharing: true
footer: true
navbar: Presentations
---

Here is a selection of my presentations, publications and posters.

## Strategies, motivations, and influencing adoption of testing for scientific code

**Ian Truslove, Erik Jasiak**

_Abstract:_

> Computation and programming are increasingly inescapable in modern Earth Sciences, but scientists and researchers receive little or no formal software engineering or programming training. At the same time, research into the reproducibility of other academic papers exposing disappointingly low rates of repeatability and high-profile retractions due to computational or data errors increase the onus on researchers to write repeatable, reliable, even reusable programs; in other words, "write better code".
> 
> Software engineering has plenty to say on the matter of "better code": metrics, methodologies, processes, tools... Of course, none are indisputable and none provide absolute guarantees. One seemingly obvious technique - testing - has enjoyed a renaissance in incarnations such as unit testing, and with approaches such as test-driven development (TDD) and behavior-driven development (BDD).
> 
> Based on our experience at the National Snow and Ice Data Center (NSIDC) with unit testing, TDD and BDD, we present a set of recommendations to scientific and research programmers about some techniques to try in their day to day programming, and possibly provide some inspiration to aim for more comprehensive approaches such as BDD. We will highlight some use cases of various types of testing at the NSIDC, discuss some of the cultural and management changes that occurred for programmers, scientists and project managers to consider and adopt processes such as TDD, make recommendations about how to introduce or expand rigorous code testing practices in your organization, and discuss the likely benefits in doing so. Computation and programming are increasingly inescapable in modern Earth Sciences, but scientists and researchers receive little or no formal software engineering or programming training. At the same time, research into the reproducibility of other academic papers exposing disappointingly low rates of repeatability and high-profile retractions due to computational or data errors increase the onus on researchers to write repeatable, reliable, even reusable programs; in other words, "write better code".
> 
> Software engineering has plenty to say on the matter of "better code": metrics, methodologies, processes, tools... Of course, none are indisputable and none provide absolute guarantees. One seemingly obvious technique - testing - has enjoyed a renaissance in incarnations such as unit testing, and with approaches such as test-driven development (TDD) and behavior-driven development (BDD).
> 
> Based on our experience at the National Snow and Ice Data Center (NSIDC) with unit testing, TDD and BDD, we present a set of recommendations to scientific and research programmers about some techniques to try in their day to day programming, and possibly provide some inspiration to aim for more comprehensive approaches such as BDD. We will highlight some use cases of various types of testing at the NSIDC, discuss some of the cultural and management changes that occurred for programmers, scientists and project managers to consider and adopt processes such as TDD, make recommendations about how to introduce or expand rigorous code testing practices in your organization, and discuss the likely benefits in doing so.

Presented at [BESSIG](http://lasp.colorado.edu/galaxy/display/BESSIG/2013/09/12/BESSIG+Meeting+Wed,+Sep+18,+4+-+6+PM) (Boulder Earth and Space Science Informatics Group) on 18 September 2013.
Slides [here](http://dx.doi.org/10.7265/N5SF2T4S)
and [local copy](./files/BESSIG_20130918_Strategies_motivations_and_influencing_adoption_of_testing_scientific_code.pdf).

Slightly modified from BESSIG and re-presented at [UCAR Software Engineering Assembly](http://sea.ucar.edu/event/testing-scientific-code) on 24 October 2013.
Slides [here](https://dl.dropboxusercontent.com/u/63673220/UCAR_SEA_20131024_Strategies_Motivations_and_Influencing_Adoption_of_Testing_Scientific_Code.pdf)
and [local copy](./files/UCAR_SEA_20131024_Strategies_Motivations_and_Influencing_Adoption_of_Testing_Scientific_Code.pdf).


## Serious JavaScript: why JavaScript is great, & how you can write great JavaScript

**Ian Truslove**

_Abstract:_

> JavaScript has enjoyed both fame and infamy since its creation in 1994. From `window.alert()` abuse, copy-and-paste coding, and identity confusion with jQuery, to Node.js and the re-emergence of server-side JavaScript, the current slew of MVC-like frameworks for the browser, and the development of ECMAScript 6, JavaScript has a rich and interesting history and continues to become more necessary for web software development than ever. This presentation aims to dispel some myths about JavaScript for the uninitiated, introduce some recent and interesting technologies emerging from the JavaScript community, and dive into a number of techniques and tools to help write well-structured, maintainable JavaScript code. In this session we will cover JavaScript's scope, object model and functional aspects, a few of JavaScript's more amusing "bad parts", some commonly-used JavaScript patterns, tools and libraries for building, testing and deploying JavaScript code, techniques and tools to ensure cross-browser compatibility, an overview of two server-side JavaScript implementations (Node.js and PhantomJS), and take a look at the future of JavaScript: ECMAScript 6, aka "Harmony".

Presented at the [2013 UCAR Software Engineering Conference](http://sea.ucar.edu/conference/2013/program#monday) on 1 April 2013.
Video was recorded and is available [online](http://sea.ucar.edu/event/serious-javascript-why-javascript-great-and-how-you-can-write-great-javascript).

HTML5 slide deck [here](http://iantruslove.github.io/serious-javascript-presentation).


## Cool Apps: Building Cryospheric Data Applications With Standards-Based Service Oriented Architecture

**Julia A Collins, Ian Truslove, Brendan W Billingsley, Joseph Oldenburg, Mary Jo Brodzik, Scott Lewis, Miao Liu**

_Abstract:_
> The National Snow and Ice Data Center (NSIDC) holds a large collection of cryospheric data, and is involved in a number of informatics research and development projects aimed at improving the discoverability and accessibility of these data. To develop high-quality software in a timely manner, we have adopted a Service-Oriented Architecture (SOA) approach for our core technical infrastructure development.
> 
> Data services at NSIDC are internally exposed to other tools and applications through standards-based service interfaces. These standards include OAI-PMH (Open Archives Initiative Protocol for Metadata Harvesting), various OGC (Open Geospatial Consortium) standards including WMS (Web Map Service) and WFS (Web Feature Service), ESIP (Federation of Earth Sciences Information Partners) OpenSearch, and NSIDC-specific RESTful services. By taking a standards-based approach, we are able to use off-the-shelf tools and libraries to consume, translate and broker these data services, and thus develop applications faster. Additionally, by exposing public interfaces to these services we provide valuable data services to technical collaborators; for example, NASA Reverb (http://reverb.echo.nasa.gov) uses NSIDC's WMS services.
> 
> Our latest generation of web applications consume these data services directly. The most complete example of this is the Operation IceBridge Data Portal (http://nsidc.org/icebridge/portal) which depends on many of the aforementioned services, and clearly exhibits many of the advantages of building applications atop a service-oriented architecture.
> 
> This presentation outlines the architectural approach and components and open standards and protocols adopted at NSIDC, demonstrates the interactions and uses of public and internal service interfaces currently powering applications including the IceBridge Data Portal, and outlines the benefits and challenges of this approach.

Presented at the 2012 American Geophysical Union Fall Meeting (session IN41C) on 6 December 2012.

Slides [here](./files/AGU_2012_Cool_Apps.pdf).


## Crawling The Web for Libre: Selecting, Integrating, Extending and Releasing Open Source Software

**Ian Truslove, Ruth E. Duerr, Hannah Wilcox, Matthew H Savoie, Luis Lopez, Michael Brandt**

_Abstract:_
> Libre is a project developed by the National Snow and Ice Data Center (NSIDC). Libre is devoted to liberating science data from its traditional constraints of publication, location, and findability. Libre embraces and builds on the notion of making knowledge freely available, and both Creative Commons licensed content and Open Source Software are crucial building blocks for, as well as required deliverable outcomes of the project.
> 
> One important aspect of the Libre project is to discover cryospheric data published on the internet without prior knowledge of the location or even existence of that data. Inspired by well-known search engines and their underlying web crawling technologies, Libre has explored tools and technologies required to build a search engine tailored to allow users to easily discover geospatial data related to the polar regions. After careful consideration, the Libre team decided to base its web crawling work on the Apache Nutch project (http://nutch.apache.org). Nutch is “an open source web-search software project” written in Java, with good documentation, a significant user base, and an active development community. Nutch was installed and configured to search for the types of data of interest, and the team created plugins to customize the default Nutch behavior to better find and categorize these data feeds.
> 
> This presentation recounts the Libre team’s experiences selecting, using, and extending Nutch, and working with the Nutch user and developer community. We will outline the technical and organizational challenges faced in order to release the project’s software as Open Source, and detail the steps actually taken. We distill these experiences into a set of heuristics and recommendations for using, contributing to, and releasing Open Source Software.

Poster session at the 2012 American Geophysical Union Fall Meeting on 3 December 2012.

Poster [here](http://goo.gl/yLp2U) or [here](./files/AGU_2012_Crawling_The_Web_for_Libre_Poster.pdf).

## Completely Test-Driven

**Ian Truslove**

_Abstract:_

> Most developers have heard of Test Driven Development (TDD), however its adoption is less widespread. This presentation outlines the philosophy, concepts and tools your team needs to completely test drive your products efficiently, from the front end down. It will define what unit tests and TDD are, and cover acceptance testing and ATDD with Cucumber, behavior driven development (BDD) and various test structures, mock objects, and fluent matchers. Examples will be in Java, JavaScript and Ruby.

Presented at the [2012 UCAR Software Engineering Conference](https://sea.ucar.edu/conference/2012/completely-test-driven) on 21 February 2012.

Slides [here](https://dl.dropboxusercontent.com/u/4292130/UCAR_SEA_20120221/CompletelyTestDriven.pdf) and [local copy](./files/CompletelyTestDriven.pdf).

<!--- 
## Agile talk at UCAR SEA 2012

## ESDSWG Tech Infusion TDD talk 2012

## AGU 2011 Poster

--->
