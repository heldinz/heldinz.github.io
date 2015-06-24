---
layout: post
title:  "Frontend Architecture with Maven and npm"
categories: frontend architecture
---
Today I was consulted on the frontend architecture for an upcoming project at work.
It is a [Maven][maven] project with several intended client platforms. These are
co-ordinated partly via [Cordova][cordova], but there are good reasons for keeping
some parts of the frontend codebase separated.

My colleagues were already sure that they wanted to use [gulp][gulp], which left two questions:

1. What is the best way to integrate gulp with Maven?
1. How can we best share JavaScript code across our different codebases?

The first was a no-brainer: the [frontend-maven-plugin][fmp] is an excellent tool
for integrating gulp, Node.js, and npm (among others) into a Maven-run build process.
It's mature, popular, and does exactly what it says on the tin. (Full disclosure:
[I contributed the gulp support][fmp-gulp] -- prior to that, the plugin only supported
Grunt and Karma.)

The second question was a little trickier, as this is a proprietary project.
However, not so long ago I came across the news that [Nexus][nexus] now
[supports npm][nexus-npm], including the ability to host private npm registries.
My suggestion was to abstract the JavaScript code to be shared into CommonsJS
modules and from there, create npm packages out of them. The [local filesystem version][npm-local]
can be linked to while the package is still under development. Once it has been
published on Nexus, other developers can install the package as normal.

I'm interested to see how this one goes!

[maven]:     https://maven.apache.org/
[cordova]:   https://cordova.apache.org/
[gulp]:      http://gulpjs.com/
[fmp]:       https://github.com/eirslett/frontend-maven-plugin
[fmp-gulp]:  https://github.com/eirslett/frontend-maven-plugin/pull/30
[nexus]:     http://www.sonatype.com/nexus/product-overview
[nexus-npm]: https://books.sonatype.com/nexus-book/reference/_introduction_7.html
[npm-local]: https://www.npmjs.com/doc/files/package.json.html#local-paths
