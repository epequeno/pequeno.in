---
layout: post
title: "EX236 - Hybrid Cloud Storage"
date: "2014-04-25 13:24:10 -0500"
published: false
---

This is the first in my series of posts for each of the five exams I will take to earn the RHCA. I'll begin with the EX236 - Hybrid Cloud Storage. This exam is closely related to RH436 - Enterprise Clustering and Storage Management.

* [Introduction](#intro)
* [Deploy to physical and virtual hardware](#obj1)
* [Configure a Red Hat Storage Server storage pool](#obj2)
* [Create individual storage bricks](#obj3)
* [Create various Red Hat Storage Server volumes](#obj4)
    * [Distributed](#obj4-1)
    * [Replicated](#obj4-2)
    * [Distributed-replicated](#obj4-3)
    * [Stripe-replicated](#obj4-4)
    * [Distributed-striped](#obj4-5)
    * [Distributed-striped-replicated](#obj4-6)
* [Format the volumes with an appropriate file system](#obj5)
* [Extend existing storage volumes](#obj6)
* [Configure clients to use NFS](#obj7)
* [Configure clients to use SMB](#obj8)
* [Configure quotas and ACLs](#obj9)
* [Configure IP failover for NFS-and SMB-based cluster services](#obj10)
* [Configure geo-replication services](#obj11)
* [Configure unified object storage](#obj12)
* [Troubleshoot Red Hat Storage Server problems](#obj13)
* [Monitor Red Hat Storage Server workloads](#obj14)
* [Perform management tasks](#obj15)

# Introduction <a name="intro">#</a>

Ok, I've made my coffee, cleaned my work area, have my CentOS 7 VM fired up and ready to go; Let's take a look at the objectives. Hmm, CentOS 7 doesn't come with Red Hat Storage Server... great. There is however, a [solution][1] suggested by the GlusterFS team themselves: use the community version of glusterfs-server. They provide a link to the main download site for GlusterFS but since I am using CentOS it will be easier to use the repo. It wasn't obvious to me at first but there is a repo available for CentOS 7 buried a few levels deep:

    http://download.gluster.org/pub/gluster/glusterfs/LATEST/CentOS/glusterfs-epel.repo

###### A quick note on terminology
A Gluser cluser is a group or "**pool**" of servers or "**nodes**" which are made of many shared filesystems or "**bricks**." A "**client**" is any device which accesses the resources available through the pools. The client may access through SAMBA, NFS, or other network protocol. More details can be found in the [common setup criteria][3].

# Deploy to physical and virtual hardware <a name="obj1">#</a>

The first objective is: "_Deploy the Red Hat Storage Server appliance on both physical and virtual hardware and work with existing Red Hat Storage Server appliances._" 

I'm not actually sure what they mean by "deploy." For now I'm going to assume that they mean "install" so I'll describe installing the glusterfs-server on a node. 

There isn't any difference between installing on physical or virtual hardware, the difference will come in how the pools and bricks are configured. The docs on installing to [bare metal][4], for example, details the infastructure: DNS, network fabric, etc.

Once the repo (as described in the Introduction) is configured I can follow the guide provided by [Server World][2] to install and start the GlusterFS daemon. I will repeat this procedure for as many nodes as I want in the pool.

    [root@node1 ~]# yum install glusterfs-server -y
    [root@node1 ~]# systemctl start glusterd
    [root@node1 ~]# systemctl enable glusterd


# Configure a Red Hat Storage Server storage pool <a name="obj2">#</a>

# Create individual storage bricks <a name="obj3">#</a>

_Create individual storage bricks on either physical devices or logical volumes_

# Create various Red Hat Storage Server volumes <a name="obj4">#</a>
The full objective is written as follows:

_Create various Red Hat Storage Server volumes such as:_
* _Distributed_
* _Replicated_
* _Distributed-replicated_
* _Stripe-replicated_
* _Distributed-striped_ 
* _Distributed-striped-replicated_

I'll break them down into their own individual sections so I can focus more directly on each.

##### Distributed <a name="obj4-1">#</a>
To create a Distributed volume:
##### Replicated <a name="obj4-2">#</a>
To create a Replicated volume:
##### Distributed-replicated <a name="obj4-3">#</a>
To create a Distributed-replicated volume:
##### Stripe-replicated <a name="obj4-4">#</a>
To create a Stripe-replicated volume:
##### Distributed-striped <a name="obj4-5">#</a>
To create a Distributed-striped volume:
##### Distributed-striped-replicated <a name="obj4-6">#</a>
To create a Distributed-striped-replicated volume: 

# Format the volumes with an appropriate file system <a name="obj5">#</a>

# Extend existing storage volumes <a name="obj6">#</a>
_Extend existing storage volumes by adding additional bricks and performing appropriate rebalancing operations_

# Configure clients to use NFS <a name="obj7">#</a>
_Configure clients to use Red Hat Storage Server appliance volumes using native and network file systems (NFS)_

# Configure clients to use SMB <a name="obj8">#</a>
_Configure clients to use Red Hat Storage Server appliance volumes using SMB_

# Configure quotas and ACLs <a name="obj9">#</a>
_Configure Red Hat Storage Server features including disk quotas and POSIX access control lists (ACLs)_

# Configure IP failover for NFS-and SMB-based cluster services <a name="obj10">#</a>

# Configure geo-replication services <a name="obj11">#</a>

# Configure unified object storage <a name="obj12">#</a>

# Troubleshoot Red Hat Storage Server problems <a name="obj13">#</a>

# Monitor Red Hat Storage Server workloads <a name="obj14">#</a>

# Perform management tasks <a name="obj15">#</a>
_Perform Red Hat Storage Server management tasks such as tuning volume options, volume migration, stopping and deleting volumes, and configuring server-side quorum_

[1]: http://blog.gluster.org/2014/07/wait-what-no-glusterfs-server-in-centos-7/
[2]: http://www.server-world.info/en/note?os=CentOS_7&p=glusterfs
[3]: http://www.gluster.org/documentation/Getting_started_common_criteria/
[4]: http://www.gluster.org/community/documentation/index.php/Getting_started_setup_baremetal