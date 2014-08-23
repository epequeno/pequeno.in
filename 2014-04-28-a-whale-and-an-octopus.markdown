---
layout: post
title: a whale and an octopus
date: "2014-04-28 23:20:13 -0500"
comments: true
categories: 
  - docker
  - tutorial
published: false
---

In our last post, [Octopress and Ubuntu](http://pequeno.in/blog/2014/04/25/octopress-and-ubuntu/) we took a look at installing [Octopress](http://octopress.org/) on Trusty Tahr manually with [nginx](http://wiki.nginx.org/Main) as our webserver. Octopress was originally released in [2009](https://github.com/imathis/octopress/graphs/contributors), and nginx in 2004 so combining these components manually felt antiquated so today we're gonna look at doing things the 2014 way and create an easily distributable [dockerfile](https://www.docker.io).

<!-- more -->

Let's begin with our base system.