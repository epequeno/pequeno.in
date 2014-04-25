---
layout: post
title: "octopress and ubuntu"
date: 2014-04-25 13:24:10 -0500
comments: true
categories: [tutorial, ubuntu, octopress] 
---

I've been a big fan of [octopress](http://octopress.org/) and today I'm going to walkthrough getting a blog setup using a RackSpace cloud instance and the latest version of [Ubuntu](https://wiki.ubuntu.com/TrustyTahr/ReleaseNotes) (14.04, Trusty Tahr).

<!-- more -->

We'll begin by signing in to our cloud [management dashboard](https://mycloud.rackspace.com) and click on 'Create Server." 

[dashboard]()

We'll go ahead and select the Ubuntu 14.04 image.

[distro choices]()

We have some options to make a pretty powerful instance (32 vCPUs, 120GB RAM) but just for demonstration we'll just create a standard instance which runs at 2 cents an hour or about $15 per month before tax.

[standard instance]()

As a security measure, RackSpace allows us to add an SSH key so we can set our server up to be able to log in without a password. By default, the only user on the system will be `root` and so the public key we suppy it will be placed in `root`'s `.ssh` folder. When we connect to the server using the IP address supplied to us, we shouldn't be asked for a password. In our example case we connect using `ssh root@166.78.9.94`

We should create a user so we can disable root logins and password authentication (making it ssh only). We also add give the new user admin rights using visudo.

```
root@octopress:~# useradd -m -s /bin/bash octopress
root@octopress:~# passwd octopress
root@octopress:~# visudo
```

Adding the following line `octopress ALL=(ALL:ALL) ALL` in the 
`# User privilege specification` section. This will allow `octopress` to use `sudo`.

Change to the newly created user and mv the public key from `root`'s `.ssh` folder to `octopress`' Change the permissions on the `authorized_keys` file and when we connect to the server we aren't aksed for the password. Only after we can get that working can we begin to modify the `sshd_config` file.

```
root@octopress:~# su octopress
octopress@octopress:~$ mkdir .ssh
octopress@octopress:~$ sudo mv /root/.ssh/authorized_keys .ssh/
octopress@octopress:~$ sudo chown octopress:octopress .ssh/authorized_keys
octopress@octopress:~$ sudo vi /etc/ssh/sshd_config
```

Set the following directives `PermitRootLogin no` and `PasswordAuthentication no`. Finally, restart the ssh service. Now we've locked down the server, root can't log in at all and no paswords are even asked for leaving only our private key as our means of entering the system.

```
octopress@octopress:~$ sudo service ssh restart
```

Now we can get to the good stuff. We'll start with [nginx](http://wiki.nginx.org/Main). We're going to take it easy this time and install from the repositories (instead of building from source.) It gets installed to `/etc/nginx` note that if you build from source the default location would be `/usr/local/nginx`. But our public documents are in `/usr/share/nginx/html` Don't forget to actually start the service.

```
octopress@octopress:~$ sudo apt-get install nginx
octopress@octopress:~$ whereis nginx
nginx: /usr/sbin/nginx /etc/nginx /usr/share/nginx /usr/share/man/man1/nginx.1.gz
octopress@octopress:~$ sudo service nginx start
```

Now that nginx is up and running we can start with octopress but we'll need [git](http://git-scm.com/), [curl](http://curl.haxx.se/) and [rvm](https://rvm.io/)

```
octopress@octopress:~$ sudo apt-get install git curl
octopress@octopress:~$ curl -L https://get.rvm.io | bash -s stable --ruby
octopress@octopress:~$ source .rvm/scripts/rvm
octopress@octopress:~$ rvm use 1.9.3
octopress@octopress:~$ rvm rubygems latest
```

Now actually get octopress. We need to adjust some ownership so that user `octopress` can write to the public html folder.

```
octopress@octopress:~$ cd /usr/share/nginx/
octopress@octopress:/usr/share/nginx$ sudo chown -R octopress:octopress html/
octopress@octopress:/usr/share/nginx$ cd html
octopress@octopress:/usr/share/nginx/html$ git clone git://github.com/imathis/octopress.git octopress
octopress@octopress:/usr/share/nginx/html$ cd octopress
octopress@octopress:/usr/share/nginx/html/octopress$ gem install bundler
octopress@octopress:/usr/share/nginx/html/octopress$ bundle install
octopress@octopress:/usr/share/nginx/html/octopress$ rake install
octopress@octopress:/usr/share/nginx/html/octopress$ rake generate

```

Now let's tell nginx which files to serve. Edit the `/etc/nginx/sites-enabled/default` file.

```
octopress@octopress:/etc/nginx/sites-enabled$ vim default
```

Change the `root` directive to the following: `root /usr/share/nginx/html/octopress/public;` Now, if you point your browser to your IP address you should see the default octopress theme. Happy blogging, hacker!


