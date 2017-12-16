---
permalink: /misc/mysql_config
layout: cnblog_post
title:  'MySQL常用配置'
date:   2017-11-10 06:34:39
categories: misc
---

如果MySQL服务器没有绑定ip，而Web程序连接它时，可能出现以下错误，django为例：

```
mysql.connector.errors.InterfaceError: 2003: Can't connect to MySQL server on '172.16.134.147:3306' (61 Connection refused)
```

bind ip 之后，django如果仍然连接不上，如出现以下错误：

```
mysql.connector.errors.DatabaseError: 1130: Host '172.16.134.1'
 is not allowed to connect to this MySQL server
```
此时，在MySQL服务器上的终端客户端连接也是这个错误，这证明连接可以成功，只是没有授权，

先将bind ip更改回127.0.0.1以便进行权限管理操作，接着连接MySQL server：，进入mysql交互命令行：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```
然后再更改ip为实际ip，这样远程的Web程序就可以访问了。

##### 其他操作

你想myuser使用mypassword从任何主机连接到mysql服务器的话。

```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```

如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器，并使用mypassword作为密码

```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```

如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器的dk数据库，并使用mypassword作为密码

```
GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```
