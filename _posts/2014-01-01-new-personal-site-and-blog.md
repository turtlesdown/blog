---
layout: post
title: New Personal Site and Blog
author: Gunnar Aasen
date: January 1, 2014
description: This is my latest attempt at creating a personal website/blog/brand that will (hopefully) not become as outdated before. To kick off the first post I'll go over my set up and the tools/libraries/code that both sites are built on.
categories: meta
tags:
  - jkl
  - meta
  - golang
---
This is my latest attempt at creating a personal website/blog/brand that will (hopefully) not become as outdated before. To kick off the first post I'll go over my set up and the tools/libraries/code that both sites are built on.

### New host

Administering my previous blog on Linode was overkill. I've now switched to the comparatively easy solution of hosting my blog and personal site on [GitHub Pages](http://pages.github.com/). If you haven't heard of it, Github Pages allows you to serve static files from one of your github repositories. It's ridiculously easy to set and use. It's also free and has an option to serve pages from a custom domain. It's basically a slightly easier version of hosting a static site on Amazon S3.

### New programming language

For the last half year, I've been learning a [Go](http://golang.org/) (or golang if you want to effectively google it). I've found golang to be easy to pick up due to it's straightforward syntax and concise core library. The [spec](http://golang.org/ref/spec) is not only readable, but intelligible. The community is also really helpful.

My only gripe is Go's youthfulness as a language (the first stable long-term version Go 1 was only [released](http://golang.org/doc/devel/release.html) two years ago). It's newness tends to show in the number of different libraries and the size of the community compared to other popular languages like ruby and python. Judging from the activity on the [golang-nuts](https://groups.google.com/forum/#!forum/golang-nuts) group, I am confident those aspects will continue to improve over time.

### Static site generators

Since my blogging needs are limited, I decided to eschew the typical wordpress/tumblr options and use a static site for my blog. In the spirit of learning, I crossed out Jekyll and Octopress from my list and took a look at several static site generators written in Go. I ultimately decided on using [jkl](https://github.com/drone/jkl) over the more popular [Hugo](http://hugo.spf13.com/). I'll write a future blog post about how I came to that decision.

### The current setup

I'm currently using my own [modified version](https://github.com/gunnaraasen/jkl) of jkl to generate my personal site and this blog. As you can probably tell, I handcrafted the design myself. On the blog, I've tried using the new css [flexbox](http://caniuse.com/flexbox) layout to obtain a modified "holy grail" layout on the blog. The flexbox was actually fairly easy to set up with the [Autoprefixer](https://sublime.wbond.net/packages/Autoprefixer) package in Sublime Text. Some other tools I'm using include:

* [highlight.js](http://highlightjs.org/) for syntax highlighting.
* [Moot](https://moot.it/) for comments.
* Google Analytics for analytics.

### Work in progress

Since jkl is fairly bare bones, I still have some work to do enable pagination and category/tag sub-pages. All in all, I'm satisfied with the site at the moment. It was a great tutorial on working with Go's template library.