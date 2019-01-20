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
supervisord -c supervisord.conf
supervisorctl start all
```

#### 修改配置文件

```
supervisorctl reload # 强制重新加载配置文件和重新启动program
supervisorctl update # 加载配置文件，只更新受影响的程序
```

#### 重要配置项

有子程序的程序 如celery, gunicorn

```
;stopasgroup=false             ; 这个东西主要用于，supervisord管理的子进程，这个子进程本身还有
                                 子进程。那么我们如果仅仅干掉supervisord的子进程的话，子进程的子进程
                                 有可能会变成孤儿进程。所以咱们可以设置可个选项，把整个该子进程的
                                 整个进程组都干掉。 设置为true的话，一般killasgroup也会被设置为true。
                                 需要注意的是，该选项发送的是stop信号
                                 默认为false。。非必须设置。。
;killasgroup=false             ; 这个和上面的stopasgroup类似，不过发送的是kill信号
```

log

```
logfile=/tmp/supervisord.log ; 这个是supervisord这个主进程的日志路径，注意和子进程的日志不搭嘎。
                               默认路径$CWD/supervisord.log，$CWD是当前目录。。非必须设置
logfile_maxbytes=50MB        ; 这个是上面那个日志文件的最大的大小，当超过50M的时候，会生成一个新的日 
                               志文件。当设置为0时，表示不限制文件大小
                               默认值是50M，非必须设置。              
logfile_backups=10           ; 日志文件保持的数量，supervisor在启动程序时，会自动创建10个buckup文件，用于log rotate
                               当设置为0时，表示不限制文件的数量。
                               默认情况下为10。。。非必须设置

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
sudo sysv-rc-conf supervisord on
```

### supervisor 日志相关问题
程序的日志应该由程序自己控制，而不要要直接将日志导向supervisor的日志中，
supervisor的日志应该只记录程序启动、重启等相关的日志信息，而程序中的具体业务日志应该由程序本身自己处理，如django使用自己的logger模块，普通脚本使用系统的logging模块。

无论怎么对supervisor启动日志和业务日志做区分，总有混在一起的情况，注意以下原则：<br>
①标准输出流中的信息，总是会打印到supervisor的日志文件中，如print、hander为StreamHandler的logger(打印不打印还要看level)<br>
②supervisor只接受标准输出流的打印，对于handler为file的logger不会打印到supervisor日志中。<br>
③**重要：**当使用了handler为file的时候，依然有log输出到supervisor的日志文件中，这是因为这个logger默认的propagate属性为True：<br>
这样的logger使用自身的handler处理完之后，会冒泡到上级logger处理，如(getLogger('a.b')这个logger的上级是getLogger('a'))，从而最终会交给getLogger()这个root loger处理（getLogger('')和getLogger()其实是一个logger）。<br>
对于getLogger(),它默认的handler为`[]`但是当系统调用`logging.basicConfig()`这个方法之后，如果没有给root logger 指定handler，那么它会自己创建一个StreamHandler，这样就会输出到标准输出流中。<br>
因此有时候指定了一个logger,如logger=getLogger('web_log'),并且指定了一个file handler，那么它在文件输出后，同时也在supervisor的日志文件中输出了一份，这是设置这个logger.propagate = False 即可，这样只会在文件中输出，而不再输出到supervisor的日志文件中了。<br>
这个问题在django和flask项目中尤为常见，这是因为django和flask调用过basicConfig,并且使用的logger没有设置propagate为False。<br>
④标准输出流print的内容会输出到stdout日志文件，而logger的内容会输出到stderr日志文件





