---
layout: post
title: "RHCE - RHEL7 - Part 1"
published: true
---

# System Configuration and management

This is the first in a three part series covering the objectives for the Red Hat Certified Engineer (RCHE, EX300) exam.

Each part will follow the objectives as they have been outlined by [Red Hat][1].

Here are the objectives that will be covered in this document:

* [Use network teaming or bonding to configure aggregated network links between two Red Hat Enterprise Linux systems](#obj1)
* [Configure IPv6 addresses and perform basic IPv6 troubleshooting](#obj2)
* [Route IP traffic and create static routes](#obj3)
* [Use firewalld and associated mechanisms such as rich rules, zones and custom rules, to implement packet filtering and configure network address translation (NAT)](#obj4)
* [Use /proc/sys and sysctl to modify and set kernel runtime parameters](#obj5)
* [Configure a system to authenticate using Kerberos](#obj6)
* [Configure a system as either an iSCSI target or initiator that persistently mounts an iSCSI target](#obj7)
* [Produce and deliver reports on system utilization (processor, memory, disk, and network)](#obj8)
* [Use shell scripting to automate system maintenance tasks](#obj9)

# Aggregated network links <a name=obj1></a>

The full objective is: _Use network teaming or bonding to configure aggregated network links between two Red Hat Enterprise Linux systems_

[Official Red Hat Documentation][2]

Using VirtualBox, we can configure two machines to connect to each other with several network configurations. In this setup I'll be using a 'host-only network'. With two NICs in the VM they are named `enp0s3` and `enp0s8` which will be the "ports" which will be teamed.

    nmcli connection add type team ifname Team0
    nmcli connection add type team-slave con-name Team0-port1 ifname enp0s3 master Team0
    nmcli connection add type team-slave con-name Team0-port2 ifname enp0s8 master Team0
    nmcli connection up team-Team0

# Configure IPv6 addresses and perform basic IPv6 troubleshooting <a name=obj2></a>

/etc/sysconfig/network-scripts/ifcfg-enp0s3

# Route IP traffic and create static routes

There is a section in the [documentation][3] on configuring static routed using the command line. The relevant man page to get information on configuring static routes will be `ip-route`.

    man ip-route

An important thing to note is how to make changes persistant:

>Static routes set using ip commands at the command prompt will be lost if the system is shutdown or restarted. To configure static routes to be persistent after a system restart, they must be placed in per-interface configuration files in the /etc/sysconfig/network-scripts/ directory.     



# Use firewalld 

The full objective is: _Use firewalld and associated mechanisms such as rich rules, zones and custom rules, to implement packet filtering and configure network address translation (NAT)_

# Use /proc/sys and sysctl to modify and set kernel runtime parameters

We can use /proc/sys or /etc/sysctl.conf to make changes to the kernel runtime parameters. 

For example, if we `cat` out the contents of the `/proc/sys/net/ipv4/ip_forward` file we can see what forwarding for this system is set to `0` (off) or `1` (on).

We can change the value to what we want in this file or we can set it in the /etc/sysctl.conf file this way:

    net.ipv4.ip_forward = 1

Notice that the name follows the directory structure within /proc/sys. 

# Configure a system to authenticate using Kerberos



# Configure a system as either an iSCSI target or initiator

The full objective is: _Configure a system as either an iSCSI target or initiator that persistently mounts an iSCSI target_



# Produce and deliver reports on system utilization (processor, memory, disk, and network)

# Use shell scripting to automate system maintenance tasks



[1]: http://www.redhat.com/en/services/training/ex300-red-hat-certified-engineer-rhce-exam
[2]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Configure_Network_Teaming.html
[3]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Using_the_Command_Line_Interface.html#sec-Static-Routes_and_the_Default_Gateway
