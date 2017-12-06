---
permalink: /misc/change_username
layout: cnblog_post
title:  '更改Ubuntu用户名、登录名、组名'
date:   2017-10-18 06:34:39
categories: misc
---

```sh
jack:x:1000:1000:jack,,,:/home/jack/:/bin/bash
mike:x:1000:

将ubuntu用户名和home目录都更改：
如上面的user为jack group也为jack，用户id和组id都是1000，桌面登录名为jack，home目录是/home/jack,
现在要达到的目标是更改后，用户id和组id不变，user变为mike，group变为mike，桌面登录名为mike，home目录变为/home/mike

1.新建一个test用户用于操作这个账户
sudo useradd -m test # 创建用户并创建home目录
sudo passwd test # 为test用户设置密码
sudo usermod -G sudo test # 将test用户添加到sudoer组中

2.重启登陆test，修改用户名和组名
①sudo usermod -l mike jack # 将jack用户名更改为mike
更改之后/etc/passwd文件内容变为：
mike:x:1000:1000:jack,,,:/home/jack/:/bin/bash
②手动更改桌面登录名
vi /etc/passwd # 将用户信息更改为下面一行的内容
mike:x:1000:1000:mike,,,:/home/jack/:/bin/bash
③修改用户组名
sudo groupmod -n mike jack
也可以手动更改用户组名(不推荐)：
vi /etc/group # 讲jack变更为mike，主要是修改下面一行
jack:x:1000: # 更改为 mike:x:1000:


这样就修改完用户名，登录名，组名，可以reboot查看一下效果

3.reboot，登录test，更改home目录
①对home目录进行copy
sudo mkdir -p /home/mike
sudo cp -r /home/jack/. /home/mike
②更改新的home目录的所有者和组
sudo chown -R mike:mike /home/mike/
③更改用户的home目录
sudo usermod -d /home/mike -m mike

此时/etc/passwd中的用户信息已经变成：
mike:x:1000:1000:mike,,,:/home/mike/:/bin/bash
```