---
layout: post
title: "RHCE - RHEL7 - Part 1"
published: true
---

# System Configureation and management

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
    ip link set dev Team0 up


---
[1]: http://www.redhat.com/en/services/training/ex300-red-hat-certified-engineer-rhce-exam
[2]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Configure_Network_Teaming.html
