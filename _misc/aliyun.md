---
layout: cnblog_post
permalink: '/misc/aliyun'
title:  "aliyun"
date:   2017-12-02 06:34:39
show: false
---
```
##### ECS
# 用root用户操作
useradd -m mike
usermod -G root mike # centOS
vim /etc/sudoers
# 在 root ALL=(ALL) ALL 下面添加
mike ALL=(ALL) ALL
# mike用户 操作

##### redis
1.创建实例的时候就设置好密码了，如果想重置，点击右上角的重置
2.在安全设置中添加白名单
3.分配公网ip
4.可以在ecs中连接了，redis-cli -h xxxxxxx.redis.rds.aliyuncs.com -p 6379 -a 'xxxxxxxx'
如果出现以下问题都是密码错误

(error) ERR invalid password
(error) NOAUTH Authentication required.
redis 忘记密码 点击重置即可

##### MySQL
1.创建实例的时候就设置好密码了
2.管理->基本信息->设置白名单
3.分配公网ip，即外网地址
4.管理->账号管理->为root用户设置密码
可以连接了：
mysql -h xxxxxxxxxxx.mysql.rds.aliyuncs.com -P 3306 -u root -p

##### Mongodb
1.创建实例的时候就设置了密码
2.设置白名单
3.分配外网地址
4.连接
```