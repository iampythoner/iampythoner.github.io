---
permalink: /misc/nginx_config
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

使用brew安装，常用目录为:

```
#### 日志目录
/usr/local/Cellar/nginx/1.12.2_1/logs
#### 配置文件目录
/usr/local/etc/nginx
#### 默认页面目录
/usr/local/var/www/
#### bin
/usr/local/Cellar/nginx/1.12.2_1/bin/nginx
```

最建议编译安装：无论是ubuntu、mac还是CentOS或者其他平台，都可以使用这种方式<br>

##### ubuntu、CentOS 编译安装:

先卸载并清理apt安装的nginx：

```
sudo nginx -s stop
sudo apt remove nginx

sudo rm -rf /usr/sbin/nginx
sudo rm -rf /etc/nginx
sudo rm -rf /var/log/nginx
sudo rm -rf /usr/sbin/nginx
sudo rm -rf /var/www/html
```

安装过程中可能遇到的一些<a href="/misc/nginx_error" target='blank'>依赖错误信息</a>

开始安装：

```
# 安装依赖项
# ubuntu
sudo apt install libpcre3 libpcre3-dev
sudo apt install zlib1g-dev
sudo apt install libssl-dev

# CentOS
sudo yum install pcre pcre-devel -y
sudo yum install zlib-devel
sudo yum install openssl-devel
```
安装

```
cd ~/Documents/lib
wget -nd http://124.205.69.131/files/91820000052E0119/nginx.org/download/nginx-1.10.2.tar.gz
tar zxf nginx-1.10.2.tar.gz
cd nginx-1.10.3

sudo ./configure --prefix=/usr/local/nginx --with-http_ssl_module

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
# 注意不要下载pcre第二版
./configure --prefix=/usr/local/nginx --with-pcre=/Users/Mike/Documents/lib/nginx/pcre-8.41 --with-openssl=/Users/Mike/Documents/lib/openssl/openssl-1.0.2n --with-http_ssl_module --with-zlib=/Users/Mike/Documents/lib/nginx/zlib-1.2.11 --with-http_stub_status_module

# make 时遇到了一个openssl的错误一直没有解决_ngx_ssl_check_host in ngx_event_openssl.o ld: symbol(s) not found for architecture x86_64
# 最后还是参照https://www.widlabs.com/article/mac-os-x-nginx-compile-symbol-not-found-for-architecture-x86_64 解决了
# 一定要先按照这篇文章修改，再make
sudo make 

sudo make install
```

编译安装的好处在于可以再次编译增加模块，同时如果想卸载只需删除`/usr/local/nginx`文件夹即可。

### 基本操作

```
#启动
sudo nginx
```
mac如果使用brew安装，初次启动的时候会有这个错误：

```
nginx: [emerg] open() "/usr/local/Cellar/nginx/1.12.2_1/logs/error.log" failed (2: No such file or directory)
```

只需到`/usr/local/Cellar/nginx/1.12.2_1/`目录新建logs目录和error.log即可

```
cd /usr/local/Cellar/nginx/1.12.2_1
mkdir logs && cd logs
touch error.log
```

##### 请求转发

```
# 将匹配的请求转发到ip为`xxx.xxx.xxx.xxx`主机上上的`port`端口处理
# 注意这里需要填写协议
location = / {
	proxy_pass http://xxx.xxx.xxx.xxx:port;
}
```

##### 二级域名配置

正常情况下不修改默认nginx.conf，而是在其他文件中修改，在nginx.conf以include的方式引用

如，nginx.conf

```
worker_processes  2;

error_log  logs/error.log;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    client_max_body_size 20M;
    keepalive_timeout  65;

    ### ..

    include servers/*;
    include mike/*;
}
```

`mike/main.conf`中配置`mike.com`, 是一个前后端分离的项目的静态页面，开发过程中使用的是8080端口

```
server {
    listen       80;
    server_name  mike.com;

    charset      utf-8;

#   location / {
#       proxy_pass http://127.0.0.1:8080;
#   }
    location / {
        alias /Users/Mike/Deploy/vue_pro/;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

`mike/api_1_0.conf`中配置`api.mike.com`, 是1.0版api的地址

```
server {
    listen       80;
    server_name  api.mike.com;
    charset      utf-8;

    location / {
        proxy_pass http://127.0.0.1:5000;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```




