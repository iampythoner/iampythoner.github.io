---
layout: cnblog_post
title:  "systemd_sysv-rc-conf"
permalink: '/misc/systemd_sysv-rc-conf'
date:   2019-01-31 07:34:39
categories: misc
---

使用 `sudo apt-get install mysql-server` 安装的mysql是使用systemd 管理开机自启动的，可以在`/lib/systemd/system`这个目录查看到`mysql.service`文件，另外可以通过以下命令查看状态:

```
systemctl list-unit-files --type=service | grep mysql
sudo systemctl is-enabled mysql.service # 是否开机自启动
sudo systemctl is-active mysql.service # 是否是启动的状态
```

因为另外购买了MySQL服务，不想使用自己在服务器搭建的MySQL，所以想关闭MySQL开机自动启动:

```
sudo systemctl disable mysql.service
```

结果报了一堆错误:

```
sudo systemctl disable mysql
Synchronizing state of mysql.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install disable mysql
insserv: warning: script 'S20redis' missing LSB tags and overrides
insserv: warning: script 'redis' missing LSB tags and overrides
insserv: There is a loop between service plymouth and procps if started
insserv:  loop involving service procps at depth 2
insserv:  loop involving service udev at depth 1
insserv: Starting redis depends on plymouth and therefore on system facility `$all' which can not be true!
insserv: Starting redis depends on plymouth and therefore on system facility `$all' which can not be true!
....
insserv: Max recursions depth 99 reached
insserv: There is a loop at service redis if started
insserv: There is a loop at service plymouth if started
insserv: There is a loop between service redis and hwclock if started
insserv:  loop involving service hwclock at depth 1
insserv: There is a loop between service plymouth and urandom if started
insserv:  loop involving service urandom at depth 3
insserv:  loop involving service mountdevsubfs at depth 1
insserv:  loop involving service checkroot at depth 4
insserv: There is a loop between service redis and udev if started
insserv:  loop involving service mountkernfs at depth 1
insserv:  loop involving service redis at depth 1
insserv: exiting now without changing boot order!
update-rc.d: error: insserv rejected the script header
```

于是挨个解决, 首先是这个：

```
insserv: warning: script 'S20redis' missing LSB tags and overrides
```

这个redis服务是之前自己设置了开机自启，当时采用的方式是:

```
①在/etct/init.d 目录下添加redis官方提供的自启和stop功能的脚本
②通过sysv-rc-conf 将在不同runlevel下的启动开关打开，目前打开的是2、3、4、5
```

多方查找答案：找到了解决方法:[https://blog.csdn.net/yinhuan1649/article/details/85625044](https://blog.csdn.net/yinhuan1649/article/details/85625044)，
原来是要提供给systemd一个标准的服务注释(这个绝对是redis官方偷懒，对比查看supervisor:vim /etc/init.d/supervisord已经写好了这部分注释),

于是`vim /etc/init.d/redis`:

```
# Start/stop the redis daemon.
#
### BEGIN INIT INFO
# Provides:          redis
# Required-Start:    $remote_fs $syslog $time
# Required-Stop:     $remote_fs $syslog $time
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Regular background program processing daemon
# Description:       redis
### END INIT INFO
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.
```

然后使用`sudo sysv-rc-conf`关闭再打开2、3、4、5几个runlevel下的开关，这样`/etc/rc2.d/`、`/etc/rc3.d/`、`/etc/rc4.d/`、`/etc/rc5.d/`几个目下都拷贝了新的redis服务启动文件(和`/etc/init.d/redis`一致)

解决后，再次关闭MySQL自动启动`sudo systemctl disable mysql.service`:

```
Synchronizing state of mysql.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install disable mysql
insserv: warning: current start runlevel(s) (empty) of script `mysql' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `mysql' overrides LSB defaults (0 1 6).
insserv: warning: current start runlevel(s) (empty) of script `mysql' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `mysql' overrides LSB defaults (0 1 6).
```

查看状态`sudo systemctl is-enabled mysql.service`: `disabled`,已经关闭开机自启。


下面是两种设置自启方式查看状态的区别：

```
systemctl status redis.service
● redis.service - LSB: Regular background program processing daemon
   Loaded: loaded (/etc/init.d/redis; bad; vendor preset: enabled)
   Active: active (running) since Tue 2019-01-29 17:31:36 CST; 1 day 22h ago
     Docs: man:systemd-sysv-generator(8)
   CGroup: /system.slice/redis.service
           └─1007 /usr/local/redis/bin/redis-server 0.0.0.0:6379

Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailabl
```


```
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-01-31 15:47:44 CST; 3s ago
  Process: 13751 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid (c
  Process: 13699 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SU
 Main PID: 13755 (mysqld)
   CGroup: /system.slice/mysql.service
           └─13755 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```



参考：
\[Systemd 教程，systemctl常用命令\][https://blog.csdn.net/weixin_37766296/article/details/80192633](https://blog.csdn.net/weixin_37766296/article/details/80192633)