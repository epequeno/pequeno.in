---
layout: post
title: "RHCE - RHEL7 - Part 2"
published: true
---

# Network Services

This is the second in a three part series covering the objectives for the Red Hat Certified Engineer (RCHE, EX300) exam.

To focus only on the topics at hand I'm disabling firewalls. If you would like to get more information about how to configure the firewall in RHEL 7 please see [Part 1][3] of this series.

Each part will follow the objectives as they have been outlined by [Red Hat][1].

* [Install the packages needed to provide the service](#obj1)
* [Configure SELinux to support the service](#obj2)
* [Use SELinux port labeling to allow services to use non-standard ports](#obj3)
* [Configure the service to start when the system is booted](#obj4)
* [Configure the service for basic operation](#obj5)
* [Configure host-based and user-based security for the service](#obj6)
* [HTTP/HTTPS](#obj7)
    * [Configure a virtual host](#obj7-1)
    * [Configure private directories](#obj7-2)
    * [Deploy a basic CGI application](#obj7-3)
    * [Configure group-managed content](#obj7-4)
    * [Configure TLS security](#obj7-5)
* [DNS](#obj8)
    * [Configure a caching-only name server](#obj8-1)
    * [Troubleshoot DNS client issues](#obj8-1)
* [NFS](#obj9)
    * [Provide network shares to specific clients](#obj9-1)
    * [Provide network shares suitable for group collaboration](#obj9-2)
    * [Use Kerberos to control access to NFS network shares](#obj9-3)
* [SMB](#obj10)
    * [Provide network shares to specific clients](#obj10-1)
    * [Provide network shares suitable for group collaboration](#obj10-2)
    * [Use Kerberos to authenticate access to shared directories](#obj10-3)
* [SMTP](#obj11)
    * [Configure a system to forward all email to a central mail server](#obj11-1)
* [SSH](#obj12)
    * [Configure key-based authentication](#obj12-1)
    * [Configure additional options described in documentation](#obj12-2)
* [NTP](#obj13)
    * [Synchronize time using other NTP peers](#obj13-1)


# Install the packages <a name=obj1></a>

The full objective is: _Install the packages needed to provide the service._

This seems like a very generic objective. I'll outline a general approach to installing the packages and ensuring they are started on boot. These steps will be simliar for any service you wish to install on a given system.

    [root@netserv ~]# yum search ntp
    [root@netserv ~]# yum install ntp
    
    [root@netserv ~]# systemctl list-unit-files | grep ntp
    ntpd.service                              disabled
    ntpdate.service                           disabled     
    
    [root@netserv ~]# systemctl enable ntpd.service 
    ln -s '/usr/lib/systemd/system/ntpd.service' '/etc/systemd/system/multi-user.target.wants/ntpd.service'
    [root@netserv ~]# systemctl start ntpd.service 

    [root@netserv ~]# systemctl list-unit-files | grep ntp
    ntpd.service                              enabled 
    ntpdate.service                           disabled

# Configure SELinux to support the service <a name=obj2></a>

Everybody's favorite thing in the world, SELinux.  First, let's make sure it's on and doing its thing:

    [root@netserv ~]# getenforce
    Permissive

We need to change it to `Enforcing` so we'll edit the SELinux config file to make sure we boot into `Enforcing` and we'll use `setenforce 1` to make SELinux enforce rules during this session:

    [root@netserv ~]# vim /etc/selinux/config 
    SELINUX=enforcing
    [root@netserv ~]# setenforce 1

Rather than put all the particular SELinux configurations here, I'll note any needed configs in the section that pertains to that service.

# Use SELinux port labeling to allow services to use non-standard ports <a name=obj3></a>

See above

# Configure the service to start when the system is booted <a name=obj4></a>

See the [first section](obj1)

# Configure the service for basic operation <a name=obj5></a>

Typically Red Hat only expects you to have the service installed and running with very little configuration. "Basic operation" also implies that the service should not be blocked by SELinux or firewall rules to be sure to understand all the aspects of what makes the service function.

# Configure host-based and user-based security for the service <a name=obj6></a>

RHEL 7, as with previous versions can use either firewalls or [TCP Wrapper][2] to configure host-based security. Some services like SMB and HTTPD can make restrictions based on users. This objective basically means that you should be aware of the various ways that you can restrict the given service.

# HTTP/HTTPS <a name=obj7></a>

This objective covers the [Apache][3] web server. Fortunaly there isn't much difference in how apache operates from RHEL 6. An important thing to be aware of is the major version has changed to httpd 2.4.x in RHEL 7. 

    [root@netserv ~]# httpd -v
    Server version: Apache/2.4.6 (CentOS)
    Server built:   Jul 23 2014 14:48:00

First, lets get it installed. Note that we also install the httpd-manual so we can have reference material available to us when needed:

    [root@netserv ~]# yum install httpd httpd-manual -y
    [root@netserv ~]# systemctl enable httpd.service 
    ln -s '/usr/lib/systemd/system/httpd.service' '/etc/systemd/system/multi-user.target.wants/httpd.service'
    [root@netserv ~]# systemctl start httpd.service 


## Configure a VirtualHost <a name=obj7-1></a>

I'll be using my domain powertra.in for these examples.

First things first, we should look at the docs that were istalled with `httpd-manual`. In this case I'll go the url http://powertrain/manual The instructions that we need are at the bottom of the middle column of links. We'll be setting up named based VirtualHosts.

With Apache 2.2 we had to use the [NameVirtualHost][4] directive. It looks like that directive no longer has any [effect][5] in Apache 2.4. 

> The NameVirtualHost directive no longer has any effect, other than to emit a warning. Any address/port combination appearing in multiple virtual hosts is implicitly treated as a name-based virtual host. 

Good news for us since this is less work. We can get straight to making our VirtualHost configuration files:

    [root@netserv ~]# cd /etc/httpd/conf.d/
    [root@netserv conf.d]# cat powertra.in.conf 
    <VirtualHost *:80>
	    ServerName powertra.in
	    DocumentRoot /var/www/vhosts/powertra.in
    </VirtualHost>
    
    [root@netserv conf.d]# cat beta.powertra.in.conf 
    <VirtualHost *:80>
	    ServerName beta.powertra.in
	    DocumentRoot /var/www/vhosts/beta.powertra.in	
    </VirtualHost>

    [root@netserv conf.d]# cd /var/www/vhosts/

    [root@netserv vhosts]# tree
    .
    ├── beta.powertra.in
    │   └── index.html
    └── powertra.in
        └── index.html
    
    2 directories, 2 files

Each of those index files has a few words to make sure they are distint. I can now go to powertra.in and beta.powertra.in both of which resolve to the same IP address and get different content for each.

## Configure private directories <a name=obj7-2></a>

By 'private' I believe they mean password protected directories. Again, we'll look to the manual. We are looking for the 'Authentication and Authorization' guide which is at the top of the 3rd column. 

Let's start by creating a directory with some super secret stuff inside:

    [root@netserv powertra.in]# pwd
    /var/www/vhosts/powertra.in
    [root@netserv powertra.in]# mkdir secret
    [root@netserv powertra.in]# cd secret/
    [root@netserv secret]# echo "secret stuff" > index.html

Now we can create (`-c`) a file named `passwords` for apache to check against.

    [root@netserv httpd]# pwd
    /etc/httpd
    [root@netserv httpd]# htpasswd -c passwords steven
    New password: 
    Re-type new password: 
    Adding password for user steven

Now we'll modify the powertra.in configuration file:

    [root@netserv httpd]# cd ../httpd/conf.d/
    [root@netserv conf.d]# vim powertra.in.conf 
    [root@netserv conf.d]# cat powertra.in.conf 
    <VirtualHost *:80>
	    ServerName powertra.in
	    DocumentRoot /var/www/vhosts/powertra.in
    </VirtualHost>
    
    <Directory /var/www/vhosts/powertra.in/secret>
	    AuthType Basic
	    AuthName "SUPER SECRET STUFF HERE"
	    AuthUserFile /etc/httpd/passwords
	    Require user steven
    </Directory>

Now, when I try to go to powertra.in/secret I'll be asked to provide my user name `steven` and password.

## Deploy a basic CGI application <a name=obj7-3></a>

Again, the manual gives us everything we need to complete this objective, this time the link that we're looking for (in my case) is http://powertra.in/manual/howto/cgi.html

Really, this is very simple since apache is already configured the way you need it to be to complete this objective. 

    [root@netserv ~]# cd /var/www/cgi-bin/
    [root@netserv cgi-bin]# ll
    total 4
    -rwxr-xr-x. 1 root root 76 Nov 25 04:04 first.pl
    [root@netserv cgi-bin]# cat first.pl 
    #!/usr/bin/perl
    print "Content-type: text/html\n\n";
    print "Hello, World.";

All I did was create a file in `/var/www/cgi-bin/` named `first.pl` and made it executable. From there I can go to powertra.in/cgi-bin/first.pl and see my output, meaning that my CGI "application" has worked.


## Configure group-managed content<a name=obj7-4></a>

This is essentially the same process we use in setting a password protected directory. This time around we're gonna create a group file which will allow us to use similar setting for a group of users.

First, let's add some users to the `passwords` file we created earlier and then add them to a `groups` file which defines the group name and who belongs to the group. In this case our group will be named `sales`:

    [root@netserv httpd]# pwd
    /etc/httpd
    [root@netserv httpd]# htpasswd passwords tom
    New password: 
    Re-type new password: 
    Adding password for user tom
    [root@netserv httpd]# htpasswd passwords sally
    New password: 
    Re-type new password: 
    Adding password for user sally
    [root@netserv httpd]# htpasswd passwords bill
    New password: 
    Re-type new password: 
    Adding password for user bill
    
    [root@netserv httpd]# vim groups
    [root@netserv httpd]# cat groups 
    sales: tom sally bill


And let's make some content the group should have access to:

    [root@netserv beta.powertra.in]# pwd
    /var/www/vhosts/beta.powertra.in
    [root@netserv beta.powertra.in]# mkdir group
    [root@netserv beta.powertra.in]# echo "group stuff" > group/index.html

And make our directory configuration in the beta.powertra.in conf file:

    [root@netserv conf.d]# pwd
    /etc/httpd/conf.d
    [root@netserv conf.d]# vim beta.powertra.in.conf 
    [root@netserv conf.d]# cat beta.powertra.in.conf 
    <VirtualHost *:80>
	    ServerName beta.powertra.in
	    DocumentRoot /var/www/vhosts/beta.powertra.in	
    </VirtualHost>
    
    <Directory /var/www/vhosts/beta.powertra.in/group>
	    AuthType Basic
	    AuthName "GROUP PEEPS ONLY"
	    AuthUserFile /etc/httpd/passwords
	    AuthGroupFile /etc/httpd/groups
	    Require group sales
    </Directory>

Now if we try going to beta.powertra.in/group/ we will be asked to provide a user name and password, we can use any that we defined in the `groups` file (`tom`, `sally` or `bill`) but the user `steven` would not be able to get in since he's not a member of the `sales` group.

## Configure TLS security<a name=obj7-5></a>

For this we'll need to install some extra packages `mod_ssl` will allow apache to use keys and certs. We will have to generate our own self-signed certs for this example. You could use the `openssl` tool but that is too complex for the purposes of this test. The `crypto-utils` package gives us the `genkey` tool which makes generating a key much easier and saves time (in my opinion).

    [root@netserv ~]# yum install mod_ssl crypto-utils -y
    [root@netserv ~]# genkey --days 365 powertra.in

Once we have our keys created we'll have to configure the VirtualHost to use TLS. I have removed the `ssl.conf` file that was provided with `mod_ssl` So I've had to put the `Listen 443 https` directive here. 

A side note: I originally made a cert with a key length of 512 bits, I wasn't able to test this in firefox 33 since keys smaller than 1024 bits are no longer accepted. I was able to test the site with elinks and everything worked from the server side. I made another key with length 2048 and firefox liked that one just fine.

    [root@netserv conf.d]# pwd
    /etc/httpd/conf.d
    [root@netserv conf.d]# cat powertra.in.conf 
    <VirtualHost *:80>
	    ServerName powertra.in
	    DocumentRoot /var/www/vhosts/powertra.in
    </VirtualHost>
    
    <Directory /var/www/vhosts/powertra.in/secret>
	    AuthType Basic
	    AuthName "SUPER SECRET STUFF HERE"
	    AuthUserFile /etc/httpd/passwords
	    Require user steven
    </Directory>
    
    Listen 443 https
    
    <VirtualHost *:443>
	    ServerName powertra.in
	    SSLEngine on
	    SSLCertificateFile /etc/pki/tls/certs/powertra.in.crt
	    SSLCertificateKeyFile /etc/pki/tls/private/powertra.in.key
    </VirtualHost>

# DNS <a name=obj8</a>

Fortunatly, the test doesn't require us to configure a DNS server to any great degree. 
    
## Configure a caching-only name server<a name=obj8-1></a>



## Troubleshoot DNS client issues<a name=obj8-1></a>


# NFS<a name=obj9></a>

## Provide network shares to specific clients<a name=obj9-1></a>

## Provide network shares suitable for group collaboration<a name=obj9-2></a>

## Use Kerberos to control access to NFS network shares<a name=obj9-3></a>


# SMB <a name=obj10></a>

## Provide network shares to specific clients<a name=bj10-1></a>

## Provide network shares suitable for group collaboration<a name=bj10-2></a>

## Use Kerberos to authenticate access to shared directories<a name=bj10-3></a>

# SMTP<a name=obj11></a>

##Configure a system to forward all email to a central mail server<a name=obj11-1></a>


# SSH<a name=obj12></a>

## Configure key-based authentication<a name=<obj12-1></a>

## Configure additional options described in documentation<a name=obj12-2></a>

# NTP<a name=obj13></a>

## Synchronize time using other NTP peers<a name=obj13-1></a>


[1]: http://www.redhat.com/en/services/training/ex300-red-hat-certified-engineer-rhce-exam
[2]: http://en.wikipedia.org/wiki/TCP_Wrapper
[3]: http://httpd.apache.org/
[4]: http://httpd.apache.org/docs/2.2/mod/core.html#namevirtualhost
[5]: http://httpd.apache.org/docs/2.4/upgrading.html#misc