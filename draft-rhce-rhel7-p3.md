---
layout: post
title: "RHCE - RHEL7 - Part 3"
published: true
---

# Database services

This is the final in a three part series covering the objectives for the Red Hat Certified Engineer (RCHE, EX300) exam. This one is pretty short since it only covers the basic use of MariaDB which is identical to MySQL. There is a good resource made by Randy Hamilton that covers the basics of [MySQL][4], knowing everything from that presentation shoudl be more than enough to prepare for these objectives.

Each part will follow the objectives as they have been outlined by [Red Hat][1].

Here are the objectives that will be covered in this document:

* [Install and configure MariaDB](#obj1)
* [Backup and restore a database](#obj2)
* [Create a simple database schema](#obj3)
* [Perform simple SQL queries against a database](#obj4)

# Install and configure MariaDB <a name=obj1></a>

The first step is fairly easy:

    [root@db ~]# yum install mariadb-server
    [root@db ~]# mysql --version
    mysql  Ver 15.1 Distrib 5.5.40-MariaDB, for Linux (x86_64) using readline 5.1
    [root@db ~]# systemctl enable mariadb.service 
    ln -s '/usr/lib/systemd/system/mariadb.service' '/etc/systemd/system/multi-user.target.wants/mariadb.service'
    [root@db ~]# systemctl start mariadb.service 

We can configure MariaDB as we would with MySQL, by editing `my.cnf` files. The MariaDB site has a page ([Configuring MariaDB with my.cnf][2]) that discusses where the configuration files can be found. There is also another page ([Server System Variables][3]) which details the available configurations.

# Backup and restore a database <a name=obj2></a>

We can still use mysqldump as we would with mysql:

    [root@db ~]# mysqldump -A > all.sql
    [root@db ~]# ls
    all.sql

And let's clear out our data directory:

    [root@db ~]# systemctl stop mariadb.service
    [root@db ~]# rm -rf /var/lib/mysql/
    [root@db ~]# mkdir /var/lib/mysql
    [root@db ~]# chown mysql:mysql /var/lib/mysql/
    [root@db ~]# systemctl start mariadb.service

    [root@db ~]# mysql
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 3
    Server version: 5.5.40-MariaDB MariaDB Server
    
    Copyright (c) 2000, 2014, Oracle, Monty Program Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | test               |
    +--------------------+
    4 rows in set (0.00 sec)

Now we can restore from the backup we created earlier:

    [root@db ~]# mysql < all.sql 
    [root@db ~]# mysql
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 5
    Server version: 5.5.40-MariaDB MariaDB Server
    
    Copyright (c) 2000, 2014, Oracle, Monty Program Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | myNewDB            |
    | mysql              |
    | performance_schema |
    | test               |
    +--------------------+
    5 rows in set (0.00 sec)

    MariaDB [(none)]> use myNewDB;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A
    
    Database changed
    
    MariaDB [myNewDB]> explain animals;
    +-------+------------------+------+-----+---------+----------------+
    | Field | Type             | Null | Key | Default | Extra          |
    +-------+------------------+------+-----+---------+----------------+
    | id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | name  | varchar(20)      | NO   |     | NULL    |                |
    | age   | int(11)          | NO   |     | NULL    |                |
    +-------+------------------+------+-----+---------+----------------+
    3 rows in set (0.00 sec)

    MariaDB [myNewDB]> select * from animals;
    +----+------+-----+
    | id | name | age |
    +----+------+-----+
    |  1 | cat  |   3 |
    |  2 | dog  |   2 |
    |  3 | cow  |  10 |
    +----+------+-----+
    3 rows in set (0.00 sec)


We can see that the database `myNewDB` and the data that we created are available.

# Create a simple database schema <a name=obj3></a>

First, I'll open the console note that we start MariaDB by using `mysql`. 

    [root@db ~]# mysql
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 3
    Server version: 5.5.40-MariaDB MariaDB Server
    
    Copyright (c) 2000, 2014, Oracle, Monty Program Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]> 

Next, create a database and table:

    MariaDB [(none)]> create database myNewDB;
    Query OK, 1 row affected (0.00 sec)
    
    MariaDB [(none)]> use myNewDB;
    Database changed
    
    MariaDB [myNewDB]> CREATE TABLE animals (
    -> id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    -> PRIMARY KEY ( id ),
    -> name VARCHAR(20) NOT NULL,
    -> age INT NOT NULL);
    Query OK, 0 rows affected (0.00 sec)
    
    MariaDB [myNewDB]> explain animals;
    +-------+------------------+------+-----+---------+----------------+
    | Field | Type             | Null | Key | Default | Extra          |
    +-------+------------------+------+-----+---------+----------------+
    | id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | name  | varchar(20)      | NO   |     | NULL    |                |
    | age   | int(11)          | NO   |     | NULL    |                |
    +-------+------------------+------+-----+---------+----------------+
    3 rows in set (0.00 sec)

# Perform simple SQL queries against a database<a name=obj4></a>

Lets's add some data:

    MariaDB [myNewDB]> INSERT INTO animals VALUES (1, 'cat', 3);
    Query OK, 1 row affected (0.00 sec)
    
    MariaDB [myNewDB]> INSERT INTO animals VALUES (2, 'dog', 2);
    Query OK, 1 row affected (0.00 sec)
    
    MariaDB [myNewDB]> INSERT INTO animals VALUES (3, 'cow', 10);
    Query OK, 1 row affected (0.00 sec)

And now we can start performing some SQL queries:

    MariaDB [myNewDB]> select * from animals;
    +----+------+-----+
    | id | name | age |
    +----+------+-----+
    |  1 | cat  |   3 |
    |  2 | dog  |   2 |
    |  3 | cow  |  10 |
    +----+------+-----+
    3 rows in set (0.00 sec)
    
    MariaDB [myNewDB]> select * from animals where name = 'dog';
    +----+------+-----+
    | id | name | age |
    +----+------+-----+
    |  2 | dog  |   2 |
    +----+------+-----+
    1 row in set (0.00 sec)
    
    MariaDB [myNewDB]> select * from animals where age > 2;
    +----+------+-----+
    | id | name | age |
    +----+------+-----+
    |  1 | cat  |   3 |
    |  3 | cow  |  10 |
    +----+------+-----+
    2 rows in set (0.00 sec)
    
    MariaDB [myNewDB]> select * from animals where name like "%w";
    +----+------+-----+
    | id | name | age |
    +----+------+-----+
    |  3 | cow  |  10 |
    +----+------+-----+
    1 row in set (0.00 sec)

That's about it. I haven't taken the exam so I don't really know exactly how much about MariaDB they expect us to know but this should be a good start.




[1]: http://www.redhat.com/en/services/training/ex300-red-hat-certified-engineer-rhce-exam
[2]: https://mariadb.com/kb/en/mariadb/documentation/getting-started/configuring-mariadb-with-mycnf/
[3]: https://mariadb.com/kb/en/mariadb/documentation/optimization-and-tuning/system-variables/server-system-variables/
[4]: http://nitedog.net/mysql/#/