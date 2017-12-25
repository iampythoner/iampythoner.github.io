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

最建议编译安装：ubuntu和mac都可以使用这种方式<br>

##### ubuntu、CentOS 编译安装:

<a href="/misc/nginx_error" target='blank'>依赖错误信息</a>

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

