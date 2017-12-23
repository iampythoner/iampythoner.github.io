---
permalink: /misc/nginx_error
layout: cnblog_post
title:  'nginx错误信息'
date:   2017-10-19 06:34:39
categories: misc
---

安装依赖

```
./configure --prefix=/usr/local/nginx

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

sudo apt install libpcre

./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using --without-http_gzip_module
option, or install the zlib library into the system, or build the zlib library
statically from the source with nginx by using --with-zlib=<path> option.

sudo apt install zlib1g-dev

./configure --prefix=/usr/local/nginx/

+ using system PCRE library
+ OpenSSL library is not used
+ using builtin md5 code
+ sha1 library is not found

sudo apt install libssl-dev

./configure --prefix=/usr/local/nginx/

+ using system PCRE library
+ OpenSSL library is not used
+ md5: using system crypto library
+ sha1: using system crypto library
+ using system zlib library

./configure --prefix=/usr/local/nginx/ --with-http_ssl_module

+ using system PCRE library
+ using system OpenSSL library
+ md5: using OpenSSL library
+ sha1: using OpenSSL library
+ using system zlib library
```