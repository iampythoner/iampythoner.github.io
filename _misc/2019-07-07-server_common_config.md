---
layout: cnblog_post
title:  "server_common_config"
permalink: '/misc/server_common_config'
date:   2019-07-07 07:34:39
categories: misc
---

```
vim /etc/ssh/ssh_config
  # Mike add for keep long connection
  ClientAliveInterval 30
  ClientAliveCountMax 86400

service sshd restart


CDPATH=.:/var/abc/

# MySQL 使用repo安装
https://dev.mysql.com/downloads/repo/yum/
找到 A Quick Guide to Using the MySQL Yum Repository

yum 安装MySQL https://www.cnblogs.com/caoxb/p/9405323.html


django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
Did you install mysqlclient?

yum search mysqlclient
  Loaded plugins: langpacks, versionlock
  Excluding 1 update due to versionlock (use "yum versionlock status" to show it)
  ================================= Matched: mysqlclient =================================
  perl-Crypt-MySQL.x86_64 : Emulate MySQL PASSWORD() function
  python36-mysql.x86_64 : An interface to MySQL
  python36-mysql-debug.x86_64 : An interface to MySQL, built for the CPython debug runtime
yum install python36-mysql.x86_64

#### redis
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar xzf redis-5.0.5.tar.gz
cd redis-5.0.5
make

sudo make install
mkdir /etc/redis
cp redis.conf /etc/redis/6379.conf
cp utils/redis_init_script /etc/init.d/redis
vim /etc/redis/6379.conf
  bind 0.0.0.0
  daemonize yes
  requirepass xxx
chkconfig redis on
service redis start 
```


