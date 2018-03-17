---
layout: cnblog_post
title:  "flask_download"
permalink: '/misc/flask_download'
date:   2018-03-17 07:34:39
categories: misc
---

问题描述: 已知一个下载链接 download_url，直接下载下来的话，文件名是 xxx.xlsx, 根据产品要求，文件名必须是filename.xlsx.

解决问题：

方案1: 前端js去修改, 网上看了下， <a href="http://somehost/somefile.zip" download="filename.zip">Download file</a>

在html标签<a>加上download属性，但是好像并没有什么用，具体可以查看原文地址:

https://scarletsky.github.io/2016/07/03/download-file-using-javascript/ 

方案2: 后台先去下载保存在服务器，然后再下载给前端页面， 这样是可以，但是比较方案比较垃圾，下载时间翻倍，这里不做介绍

方案3: 使用python flask框架的stream流，相当于一个管道一样，将第三方地址的下载流转换到当前页面，下面是代码的实现


```
import requests
from flask import request, stream_with_context, Response

def file_downlad(url, file_name):
    # 首先定义一个生成器，每次读取512个字节
    def generate():
        r = requests.get(url, cookies=request.cookies, stream=True)
        for chunk in r.iter_content(chunk_size=512):
            if chunk:
                yield chunk

    response = Response(stream_with_context(generate()))
    content_disposition = "attachment; filename={}".format(file_name)
    response.headers['Content-Disposition'] = content_disposition
    return response
```


### send_from_directory

https://www.cnblogs.com/we8fans/p/7107353.html

```
from flask import send_file, send_from_directory
import os

@app.route("/download/<filename>", methods=['GET'])
def download_file(filename):
    # 需要知道2个参数, 第1个参数是本地目录的path, 第2个参数是文件名(带扩展名)
    directory = os.getcwd()  # 假设在当前目录
    return send_from_directory(directory, filename, as_attachment=True)
```

后边那个as_attachment参数需要赋值为True，不过此种办法有个问题，就是当filename里边出现中文的时候，会报如下错误:



解决办法:
使用flask自带的make_response
代码修改如下



```
from flask import send_file, send_from_directory
import os
from flask import make_response

@app.route("/download/<filename>", methods=['GET'])
def download_file(filename):
    # 需要知道2个参数, 第1个参数是本地目录的path, 第2个参数是文件名(带扩展名)
    directory = os.getcwd()  # 假设在当前目录
    response = make_response(send_from_directory(directory, filename, as_attachment=True))
    response.headers["Content-Disposition"] = "attachment; filename={}".format(file_name.encode().decode('latin-1'))
    return response
```

根据文件名猜测MIME
mime_type = mimetypes.guess_type(filename)[0]


#### 想浏览器推静态文件

```
from flask import app
import os


@app.route("/download/<filepath>", methods=['GET'])
def download_file(filepath):
    # 此处的filepath是文件的路径，但是文件必须存储在static文件夹下， 比如images\test.jpg
    return app.send_static_file(filepath) 
```
