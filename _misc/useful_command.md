---
permalink: /misc/useful_command
layout: cnblog_post
title:  '非常有用的命令'
date:   2017-10-22 06:34:39
categories: misc
---

常看端口占用情况：

```
sudo lsof -i:80
netstat -tunlp|grep
```
这是最服百度的一次：<a href="https://jingyan.baidu.com/article/546ae1853947b71149f28cb7.html" target='blank'>linux如何查看端口被哪个进程占用？</a>


vim 修改了只读文件，但是想保存住呀

```
:w !sudo tee %
```
:w : Write a file.可以将文件写入，文件仍然是只读模式，通过 :q! 退出<br>
!sudo : Call shell sudo command.<br>
tee : The output of the vi/vim write command is redirected using tee.
% : Triggers the use of the current filename.<br>
Simply put, the ‘tee’ command is run as sudo and follows the vi/vim command on the current filename given.<br>


查看发行版信息

```
cat /proc/version

lsb_release -a
```

查看文件的16进制数据

```
xxd
``` 

ubuntu桌面版拨号联网

```
nm-connection-editor
```