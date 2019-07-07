---
layout: cnblog_post
title:  "async_request_lib_compare"
permalink: '/misc/async_request_lib_compare'
date:   2019-07-08 07:34:39
categories: misc
---
https://stackoverflow.com/questions/39435443/why-is-python-3-http-client-so-much-faster-than-python-requests

https://github.com/svanoort/python-client-benchmarks

https://stackoverflow.com/questions/15461995/python-requests-vs-pycurl-performance/32899936#32899936

https://stackoverflow.com/questions/9110593/asynchronous-requests-with-python-requests

https://stackoverflow.com/questions/10555292/ideal-method-for-sending-multiple-http-requests-over-python

client:
http://pycurl.io
https://github.com/pycurl/pycurl

https://github.com/gwik/geventhttpclient

web:
https://www.freecodecamp.org/news/million-requests-per-second-with-python-95c137af319/

```
https://github.com/encode/http3
https://www.encode.io/http3/
完备的API， parallel处于beta阶段，暂时不敢使用
https://github.com/encode/http3/pull/52
```

```
https://github.com/encode/requests-async
自己承认已经被http3超越

推荐
The http3 package both sync and async HTTP clients, with a requests-compatible API.
The aiohttp package provides an alternative client for making async HTTP requests.
```

https://github.com/encode


```
https://github.com/kennethreitz/grequests

used by 1190 stars 3060
```


```
https://github.com/ross/requests-futures
```


异常时，一组请求中的其他请求取消，并释放连接
client性能好点

requests文档https://2.python-requests.org//zh_CN/latest/user/advanced.html
推荐非阻塞使用grequests 和 requests-futures



pip install http3
pip install requests
pip install pycurl
pip install grequests