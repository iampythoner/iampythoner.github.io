---
layout: cnblog_post
title:  "HTTPS配置"
permalink: '/misc/HTTPS_config'
date:   2018-01-07 07:34:39
categories: misc
---


### 购买SSL证书

有免费的、有收费的，我现在使用的是腾讯云提供的免费一年的域名型免费版(DV)证书，在<a href="https://console.qcloud.com/ssl" target='blank'>https://console.qcloud.com/ssl</a> 申请即可，点击购买证书-->域名型免费版(DV)


### 验证证书

其实就是将申请的证书和域名绑定，参考<a href="https://cloud.tencent.com/document/product/400/4142" target='blank'>https://cloud.tencent.com/document/product/400/4142</a>，


### nginx/Apache/IIS 安装证书

参考 <a href="https://cloud.tencent.com/document/product/400/4143" target='blank'>https://cloud.tencent.com/document/product/400/4143</a>
我使用的是nginx，

先上传证书到服务器，在证书申请的网站的管理页面下载即可，一般有nginx/Apache/IIS三份，然后上传到服务器，这里我将nginx证书(.crt或.pem)和私钥(.key)上传到了`/usr/local/nginx/ssl_file/iampythoner.com/`目录下，下面的nginx配置会用到这两个文件。

然后一下nginx的配置，让所有的HTTP请求重定向给HTTPS

```
server {
    listen        80;
    server_name   iampythoner.com www.iampythoner.com;
    return 302    https://$host$request_uri;
}

server {
    listen       443 ssl;
    server_name  iampythoner.com www.iampythoner.com;

    ssl_certificate /usr/local/nginx/ssl_file/iampythoner.com/1_iampythoner.com_bundle.crt;
    ssl_certificate_key /usr/local/nginx/ssl_file/iampythoner.com/2_iampythoner.com.key;

    location / {
        # ......
    }
}
```