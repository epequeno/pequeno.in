---
layout: post
title: "EX236 - Hybrid Cloud Storage"
date: "2014-04-25 13:24:10 -0500"
published: false
---

This is the first in my series of posts for each of the five exams I will take to earn the RHCA. To kick things off we begin with the EX236 - Hybrid Cloud Storage. This exam is closely related to RH436 - Enterprise Clustering and Storage Management.

* Introduction

# Introduction

Ok, got my coffee, I've cleaned my work area, have my CentOS 7 VM fired up and ready to go let's take a look at the objectives. Hmm, CentOS 7 doesn't come with Red Hat Storage Server... great. There is however, a [workaroud][1] suggested by the GlusterFS team themselves: use the community version of glusterfs-server. They provide a link to the main download site for GlusterFS but since we are using CentOS you are going to have to dig a little deeper to get to the repo they provide.

    http://download.gluster.org/pub/gluster/glusterfs/LATEST/CentOS/glusterfs-epel.repo

From there we can follow the guide provided by [Server World][2] to install and start the GlusterFS daemon:

    yum install glusterfs-server -y
    systemctl start glusterd
    systemctl enable glusterd

Now that Gluster is installed we can actually get to work.

[1]: http://blog.gluster.org/2014/07/wait-what-no-glusterfs-server-in-centos-7/
[2]: http://www.server-world.info/en/note?os=CentOS_7&p=glusterfs