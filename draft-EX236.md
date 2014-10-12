---
layout: post
title: "EX236 - Hybrid Cloud Storage"
published: true
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

# Introduction <a name="intro"></a>

This exam covers the subject of the Red Hat Storage Server product which is the comercially supported solution that Red Hat offers. If you want to study for this exam without having to purchase a support licence there is a [solution][1] suggested by the GlusterFS team themselves: use the community version of glusterfs-server. 

The Gluster team provide a link to the main download site for GlusterFS but since I am using CentOS it will be easier to use the repo with yum. It wasn't obvious to me at first but there is a repo available for CentOS 7 buried a few levels deep:

    http://download.gluster.org/pub/gluster/glusterfs/LATEST/CentOS/glusterfs-epel.repo

This document is organized by objective as they are given by Red Hat. It is better to use this as a reference for each objective rather than a walkthrough. For example, the objective to create bricks comes **after** the objective to configure a storage pool. Normally you would create the bricks before adding them to a pool.

###  A quick note on terminology
A Gluser cluser is a group or "**pool**" of servers or "**nodes**" which are made of several shared filesystems or "**bricks**." A group of bricks make a "**volume**". A "**client**" is any device which accesses the resources available through the pools. More details can be found in the [common setup criteria][3].

# Deploy to physical and virtual hardware <a name="obj1"></a>
The first objective is: "_Deploy the Red Hat Storage Server appliance on both physical and virtual hardware and work with existing Red Hat Storage Server appliances._" 

I'm not actually sure what they mean by "deploy." For now I'm going to assume that they mean "install" so I'll describe installing the glusterfs-server on a node. 

There isn't any difference between installing on physical or virtual hardware, the difference will come in how the pools and bricks are configured. The docs on installing to [bare metal][4], for example, details the infastructure: DNS, network fabric, etc.

Once the repo (as described in the Introduction) is configured I can follow the guide provided by [Server World][2] to install and start the GlusterFS daemon. I will repeat this procedure for as many nodes as I want in the pool.

    [root@node1 ~]# yum install glusterfs-server -y
    [root@node1 ~]# systemctl start glusterd.service
    [root@node1 ~]# systemctl enable glusterd.service

# Configure a Red Hat Storage Server storage pool <a name="obj2"></a>

To create the pool, I'll name each node using a _node#_ pattern, `node1.pequeno.in`, `node2.pequeno.in`, etc. 	
    
Here, I tell the nodes to learn about each other. Once a node has been added to a pool it can inform other nodes about the ones it already knows about. When `node1` probed for `node2` I get one kind of success message:

    [root@node1 ~]# gluster peer probe node2.pequeno.in
    peer probe: success.

But when I ask `node3` to probe `node1` I get a different kind of success message since `node3` already knows about `node1`.
	
    [root@node3 ~]# gluster peer probe node1.pequeno.in
    peer probe: success. Host node1.pequeno.in port 24007 already in peer list
    

# Create individual storage bricks <a name="obj3"></a>
The full objective is: _Create individual storage bricks on either physical devices or logical volumes_. These are nodes with additional blocks of storage added to become the bricks. The added block storage is `/dev/xvdb` on each of the nodes.

    [root@node1 ~]# mkdir /glusterfs
    [root@node1 ~]# fdisk /dev/xvdb
    [root@node1 ~]# mkfs -t xfs /dev/xvdb5
    [root@node1 ~]# mount /dev/xvdb5 /glusterfs
    [root@node1 ~]# tail -1 /etc/mtab >> /etc/fstab
    [root@node1 ~]# mkdir /glusterfs/distributed

# Create various Red Hat Storage Server volumes <a name="obj4"></a>
The full objective is:

_Create various Red Hat Storage Server volumes such as:_

* _Distributed_
* _Replicated_
* _Distributed-replicated_
* _Stripe-replicated_
* _Distributed-striped_ 
* _Distributed-striped-replicated_

I'll break them down into their own individual sections so I can focus more directly on each. This section borrows heavily from the "[Setting up GlusterFS Server Volumes][6]" guide provided by the Gluster team. For the examples I'll be using the same FQDNs (`node1.pequeno.in`) for each setup but for each section assume these are newly created pools and bricks.

### Distributed <a name="obj4-1"></a>

![distributed](/../img/ex236/Distributed_Volume.png)

Distributed is the default mode GlusterFS will use if I don't specify any other configuration. In this example I'm only giving name of the volume `vol_distributed`.  

    [root@node1 ~]# gluster volume create vol_distributed \
    >node1.pequeno.in:/glusterfs/distributed \
    >node2.pequeno.in:/glusterfs/distributed \
    >node3.pequeno.in:/glusterfs/distributed;
    volume create: vol_distributed: success: please start the volume to access data
     
Start the pool

    [root@node1 ~]# gluster volume start vol_distributed
    volume start: vol_distributed: success

Even though we started the volume on `node1`, I can check the status of the volume from any node in the pool. Here, I'm checking the status from `node2`.

    [root@node2 ~]# gluster volume info
    Volume Name: vol_distributed
    Type: Distribute
    Volume ID: 72e242ac-6fa8-49a1-a209-e20b80fe90fb
    Status: Started
    Number of Bricks: 3
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/distributed
    Brick2: node2.pequeno.in:/glusterfs/distributed
    Brick3: node3.pequeno.in:/glusterfs/distributed

### Replicated <a name="obj4-2"></a>

![replicated](/../img/ex236/Replicated_Volume.png)

    [root@node1 ~]# gluster volume create replication1 replica 2 \
    node1.pequeno.in:/glusterfs/replicated \
    node2.pequeno.in:/glusterfs/replicated
    volume create: replication1: success: please start the volume to access data

    [root@node1 ~]# gluster volume start replication1
    volume start: replication1: success

I can check the status of the volume on `node2`:
    
    [root@node2 ~]# gluster volume info
    
    Volume Name: replication1
    Type: Replicate
    Volume ID: 75182cb0-07c5-4118-86c8-862f2a40d384
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/replicated
    Brick2: node2.pequeno.in:/glusterfs/replicated

### Distributed-replicated <a name="obj4-3"></a>

![distributed-replicated](/../img/ex236/Distributed_Replicated_Volume.png)

What is important to note about this configuration is that it is the exact syntax I used for the replicated volume. The difference is that I am providing more bricks (4) than the number of replication volumes (2). 

Gluster will replicate automatically across the first two nodes (`node1` and `node2` each get a copy of `file1`) and then distribute files on the latter nodes (`node3` and `node4` each get a copy of `file2`). 

It is also a better choice to have memebers of a replica set on different servers. While you could, you wouldn't want to have a physical device trying to replicate to itself. If they are on seperate physical volumes they are better protected against disk failure. 

An important note from the documentation:

> Note: The number of bricks should be a multiple of the replica count for a distributed replicated volume.

    [root@node1 ~]# gluster volume create dist-repl replica 2 \
    >node1.pequeno.in:/glusterfs/distributed-replicated/ \
    >node2.pequeno.in:/glusterfs/distributed-replicated/ \
    >node3.pequeno.in:/glusterfs/distributed-replicated/ \
    >node4.pequeno.in:/glusterfs/distributed-replicated/;
    volume create: dist-repl: success: please start the volume to access data

    [root@node4 ~]# gluster volume info

    Volume Name: dist-repl
    Type: Distributed-Replicate
    Volume ID: 4a92e432-a1cc-419a-a6a7-7280854619c1
    Status: Started
    Number of Bricks: 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/distributed-replicated
    Brick2: node2.pequeno.in:/glusterfs/distributed-replicated
    Brick3: node3.pequeno.in:/glusterfs/distributed-replicated
    Brick4: node4.pequeno.in:/glusterfs/distributed-replicated

### Stripe-replicated <a name="obj4-4"></a>

![striped-replicated](/../img/ex236/Striped_Replicated_Volume.png)

I'm not sure that image is labeled as clearly as it could be. When I send a file to the volume named `strp-repl` it will be distributed across 2 volumes with some of the data going to the first volume containing `node1` and `node2`. The rest of the data going to the second volume containing `node3` and `node4`. To call these replicated volumes is misleading since it's the bricks within the volumes that are replicated.

Data in each volume the data is replicated between the bricks. `node2` will be a duplicate of the data found on `node1`, and `node4` will be a duplicate of the data found on `node3`.

    [root@node1 ~]# gluster volume create strp-repl stripe 2 replica 2 \
    >node1.pequeno.in:/glusterfs/striped-replicated/ \
    >node2.pequeno.in:/glusterfs/striped-replicated/ \
    >node3.pequeno.in:/glusterfs/striped-replicated/ \
    >node4.pequeno.in:/glusterfs/striped-replicated/;
    volume create: strp-repl: success: please start the volume to access data



    [root@node3 ~]# gluster volume info
    
    Volume Name: strp-repl
    Type: Striped-Replicate
    Volume ID: c002f35f-68ff-45de-9021-a4e77d68842f
    Status: Started
    Number of Bricks: 1 x 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/striped-replicated
    Brick2: node2.pequeno.in:/glusterfs/striped-replicated
    Brick3: node3.pequeno.in:/glusterfs/striped-replicated
    Brick4: node4.pequeno.in:/glusterfs/striped-replicated

### Distributed-striped <a name="obj4-5"></a>

![distributed-striped](/../img/ex236/Distributed_Striped_Volume.png)

This example shows again how distribution is often implied. Here I'm asking GlusterFS to stripe across 2 but I'm giving it 4 bricks. GlusterFS will stripe `file1` across `node1` and `node2`, and `file2` will be striped across `node3` and `node4`.

    [root@node1 ~]# gluster volume create dist-strp stripe 2 \
    >node1.pequeno.in:/glusterfs/distributed-striped/ \
    >node2.pequeno.in:/glusterfs/distributed-striped/ \
    >node3.pequeno.in:/glusterfs/distributed-striped/ \
    >node4.pequeno.in:/glusterfs/distributed-striped/;
    volume create: dist-strp: success: please start the volume to access data

    [root@node2 ~]# gluster volume info
    Volume Name: dist-strp
    Type: Distributed-Stripe
    Volume ID: 5ba0b6fb-8107-473c-88d6-40f75595b2c9
    Status: Started
    Number of Bricks: 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/distributed-striped
    Brick2: node2.pequeno.in:/glusterfs/distributed-striped
    Brick3: node3.pequeno.in:/glusterfs/distributed-striped
    Brick4: node4.pequeno.in:/glusterfs/distributed-striped

### Distributed-striped-replicated <a name="obj4-6"></a>

![distributed-striped-replicated](/../img/ex236/Distributed_Striped_Replicated_Volume.png)

    [root@node1 ~]# gluster volume create dist-strp-repl stripe 2 replica 2 \
    >node1.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node2.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node3.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node4.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node5.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node6.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node7.pequeno.in:/glusterfs/dist-strp-repl/ \
    >node8.pequeno.in:/glusterfs/dist-strp-repl/;
    volume create: dist-strp-repl: success: please start the volume to access data

    [root@node2 ~]# gluster volume info    
    
    Volume Name: dist-strp-repl
    Type: Distributed-Striped-Replicate
    Volume ID: 2b93ca88-8dc9-401b-bc16-4041b9a733b5
    Status: Started
    Number of Bricks: 2 x 2 x 2 = 8
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/dist-strp-repl
    Brick2: node2.pequeno.in:/glusterfs/dist-strp-repl
    Brick3: node3.pequeno.in:/glusterfs/dist-strp-repl
    Brick4: node4.pequeno.in:/glusterfs/dist-strp-repl
    Brick5: node5.pequeno.in:/glusterfs/dist-strp-repl
    Brick6: node6.pequeno.in:/glusterfs/dist-strp-repl
    Brick7: node7.pequeno.in:/glusterfs/dist-strp-repl
    Brick8: node8.pequeno.in:/glusterfs/dist-strp-repl

# Format the volumes with an appropriate file system <a name="obj5"></a>

Gluster is flexible when it comes to the type of file system you can format a volume with. The Gluster team suggest XFS or ext4 but there aren't very many restrictions from the Gluster side of the fence. 

Red Hat, on the other hand, makes a clear choice: [XFS][5]. 

> Red Hat supports only formatting a Logical Volume by using the XFS file system on the bricks with few modifications to improve performance. Ensure to format bricks using XFS on Logical Volume Manager before adding it to a Red Hat Storage volume. 

From the documentation, you should format the device with the following command:

    # mkfs.xfs -i size=512 <DEVICE>

# Extend existing storage volumes <a name="obj6"></a>
_Extend existing storage volumes by adding additional bricks and performing appropriate rebalancing operations_

The objectives for this exam don't specify that we should know how to configure a strictly striped volume but I believe that is a useful configuration to test a rebalancing operation.

I'll be using a new `node1` and `node2` for this example. 

    [root@node1 ~]# gluster peer probe node2.pequeno.in
    peer probe: success. 
    [root@node1 ~]# gluster peer status
    Number of Peers: 1

    Hostname: node2.pequeno.in
    Uuid: 4ce512c2-00d2-4a36-8651-7885588e495a
    State: Peer in Cluster (Connected)

Now I'll create a brick on each of the nodes, I'm only showing the commands entered for `node1`, the exact commands are to be duplicated on `node2`.

    [root@node1 ~]# fdisk /dev/xvdb
    [root@node1 ~]# mkfs.xfs -i size=512 /dev/xvdb5
    [root@node1 ~]# mkdir /glusterfs
    [root@node1 ~]# mount /dev/xvdb5 /glusterfs/
    [root@node1 ~]# mkdir /glusterfs/striped

I can now create the striped volume `strp-vol`

    [root@node1 ~]# gluster volume create strp-vol stripe 2 \
    > node1.pequeno.in:/glusterfs/striped \
    > node2.pequeno.in:/glusterfs/striped;
    volume create: strp-vol: success: please start the volume to access data
    [root@node1 ~]# gluster volume start strp-vol
    volume start: strp-vol: success

    [root@gluster ~]# gluster volume info
     
    Volume Name: strp-vol
    Type: Stripe
    Volume ID: 5b54ba8c-706c-48fa-bdf3-befd19095916
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: node1.pequeno.in:/glusterfs/striped
    Brick2: node2.pequeno.in:/glusterfs/striped

I'll have a client use this volume. To configure a client using the native gluster application the fuse module must be available in the kernel. 

   [root@client ~]# modprobe fuse
   [root@client ~]# dmesg | grep -i fuse
   [ 1190.323234] fuse init (API version 7.22)
   [ 1190.348663] SELinux: initialized (dev fusectl, type fusectl), uses genfs_contexts
   [root@client ~]# yum -y install openssh-server wget fuse fuse-libs openib libibverbs
   [root@client ~]# cd /etc/yum.repos.d/
   [root@client yum.repos.d]# 
   [root@client yum.repos.d]# wget http://download.gluster.org/pub/gluster/glusterfs/LATEST/CentOS/glusterfs-epel.repo
   [root@client ~]# yum install glusterfs-core glusterfs-fuse glusterfs-rdma







# Configure clients to use NFS <a name="obj7"></a>
_Configure clients to use Red Hat Storage Server appliance volumes using native and network file systems (NFS)_

# Configure clients to use SMB <a name="obj8"></a>
_Configure clients to use Red Hat Storage Server appliance volumes using SMB_

# Configure quotas and ACLs <a name="obj9"></a>
_Configure Red Hat Storage Server features including disk quotas and POSIX access control lists (ACLs)_

# Configure IP failover for NFS-and SMB-based cluster services <a name="obj10"></a>

# Configure geo-replication services <a name="obj11"></a>

# Configure unified object storage <a name="obj12"></a>

http://blog.gluster.org/2012/09/howto-using-ufo-swift-a-quick-and-dirty-setup-guide/


# Troubleshoot Red Hat Storage Server problems <a name="obj13"></a>

# Monitor Red Hat Storage Server workloads <a name="obj14"></a>

# Perform management tasks <a name="obj15"></a>
_Perform Red Hat Storage Server management tasks such as tuning volume options, volume migration, stopping and deleting volumes, and configuring server-side quorum_

[1]: http://blog.gluster.org/2014/07/wait-what-no-glusterfs-server-in-centos-7/
[2]: http://www.server-world.info/en/note?os=CentOS_7&p=glusterfs
[3]: http://www.gluster.org/documentation/Getting_started_common_criteria/
[4]: http://www.gluster.org/community/documentation/index.php/Getting_started_setup_baremetal
[5]: https://access.redhat.com/documentation/en-US/Red_Hat_Storage/2.0/html/Administration_Guide/chap-User_Guide-Setting_Volumes.html#idp10244912
[6]: https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_setting_volumes.md
