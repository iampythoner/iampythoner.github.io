---
layout: cnblog_post
title:  "supervisor"
permalink: '/misc/supervisor'
date:   2018-12-19 07:34:39
categories: misc
---


[http://www.supervisord.org/](http://www.supervisord.org/)

#### install

```
sudo pip install supervisor

# which supervisorctl # /usr/local/bin/supervisorctl
# which echo_supervisord_conf
# which supervisord
# pip list
```

#### Creating a Configuration File

```
mkdir supervisor_conf
cd supervisor_conf
echo_supervisord_conf > supervisord.conf
vim supervisord.conf

```

启动

```
supervisord -c 
supervisorctl start all
```

#### 修改配置文件

supervisorctl reload


### 问题


#### 问题1

```
couldn't setuid to 0: Can't drop privilege as nonroot user
supervisor: child process was not spawned
```

修改配置文件

```
[supervisord]
user=root                 ; default is current user, required if root
```

然后,以root用户启动

```
sudo supervisord -c
sudo supervisorctl start all
```


###### 如果不想以root用户运行

首先保证

```
[supervisord]
; 不是以root运行，可以不用填user，默认使用当前user
;user=root                 ; default is current user, required if root
```

另外`[program]`中的user也不填


```
# 当前用户
python /usr/local/bin/supervisord -c supervisord.conf
# 当前用户
python /usr/local/bin/supervisorctl -c supervisord.conf start all
```

#### 问题2

程序在启动的时候用到了django的log handler, 比如启动一个celery的beat，中间用到了log，这个log是django配置的handler，出现了这种错误：

```
File "/usr/local/lib/python3.6/logging/config.py", line 566, in configure
    '%r: %s' % (name, e))
ValueError: Unable to configure handler 'error': [Errno 13] Permission denied: '/zzz/xxx/yyy.log'
```

解决: 将yyy.log所在目录(xxx)的权限更改为777, 并且保证到达log路径的所有文件夹ownner都是执行的用户


### 开机启动

[http://www.supervisord.org/running.html#running-supervisord-automatically-on-startup](http://www.supervisord.org/running.html#running-supervisord-automatically-on-startup)

```
wget https://raw.githubusercontent.com/Supervisor/initscripts/master/ubuntu -O supervisord
sudo cp supervisord /etc/init.d/supervisord
vim /etc/init.d/supervisord
    xxxx
sudo chmod +x /etc/init.d/supervisord
sudo update-rc.d supervisord defaults
sudo systemctl daemon-reload
# test
service supervisord start/stop
# auto start
sudo sysv-rc-conf redis on
```





