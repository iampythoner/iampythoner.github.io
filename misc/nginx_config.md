---
permalink: /misc/nginx_cnfig
layout: cnblog_post
title:  'nginx安装和配置'
date:   2017-10-19 06:34:39
categories: misc
---

### 1.安装


ubuntu使用apt安装

```
sudo apt install nginx
```

安装完成之后：

```
配置文件目录：
/etc/nginx
默认的日志文件存放目录
/var/log/nginx/access.log
/var/log/nginx/error.log
默认可执行程序位置
/usr/sbin/nginx
默认html存放的目录
/var/www/html
```

mac 使用brew安装：

```
brew install nginx
```

最建议编译安装：ubuntu和mac都可以使用这种方式<br>

##### ubuntu编译安装:

```
# 安装依赖项
sudo apt install libtool
sudo apt install libpcre3 libpcre3-dev
sudo apt install zlib1g-dev
sudo apt install openssl libssl-dev
```
安装

```
cd ~/Documents/lib
wget -nd http://124.205.69.131/files/91820000052E0119/nginx.org/download/nginx-1.10.2.tar.gz
tar zxf nginx-1.10.3.tar.gz
cd nginx-1.10.3


sudo ./configure --prefix=/usr/local/nginx 

sudo make

sudo make install 
```

##### mac编译安装:

准备三方库源码

```
# pcre: http://www.pcre.org/  下载地址 https://sourceforge.net/projects/pcre/files/
# openssl: https://www.openssl.org/source/
# zlib: http://www.zlib.net/
```



```
./configure --prefix=/usr/local/nginx --with-pcre=/Users/Mike/Documents/lib/nginx/pcre2-10.30 --with-openssl=/Users/Mike/Documents/lib/openssl/openssl-1.0.2n --with-http_ssl_module --with-zlib=/Users/Mike/Documents/lib/nginx/zlib-1.2.11 --with-http_stub_status_module

sudo make 

sudo make install
```

mac 上的pcre老是编译失败，最终还是使用了brew安装。


编译安装的好处在于可以再次编译增加模块，同时如果想卸载只需删除`/usr/local/nginx`文件夹即可。

brew 安装之后的常用目录为:

```
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
```

