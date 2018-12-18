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




