---
permalink: /misc/uwsgi_cnfig
layout: cnblog_post
title:  'uwsgi安装和配置'
date:   2017-10-19 06:34:39
categories: misc
---

<!--Category-->
<div id="navCategory">
    <strong>本文目录</strong>
    <ul>
        <li><a href="#anchor1_0">安装</a>
            <ul>
                <li><a href="#anchor1_1">mac</a></li>
                <li><a href="#anchor1_2">ubuntu</a></li>
            </ul>
        </li>
        <li><a href="#anchor2_0">基本配置和使用</a>
            <ul>
                <li><a href="#anchor2_1">mac 下的配置项</a></li>
                <li><a href="#anchor2_2">ubuntu 下的配置项</a></li>
            </ul>
        </li>
    </ul>
</div>
<!--Category结束-->

<h3 id="anchor1_0">安装</h3>
<h5 id="anchor1_1">mac</h5>
编译安装

```
wget https://projects.unbit.it/downloads/uwsgi-latest.tar.gz
tar zxvf uwsgi-latest.tar.gz
cd <dir>
make
```
出现这个问题:

```
ld: file not found: /usr/lib/system/libsystem_darwin.dylib for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
*** error linking uWSGI ***
```

可以添加编译连接库，再执行make

```
export LDFLAGS="-isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk

make
```

之后出现了链接ssl库的错误，可以这样执行

```
export LDFLAGS="-isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.13.sdk -L /usr/local/Cellar/openssl@1.1/1.1.0g/lib"

make
```

这样又出现了x86_64找不到符号的问题，而无论是使用pip安装，还是下载uwsgi源码编译安装都有类似的问题， 最后是使用brew安装的:

```
brew install uwsgi
```

<h5 id="anchor1_2">ubuntu</h5>
ubuntu上安装非常简单，进入虚拟环境使用pip安装即可

```
workon xxx
pip install uwsgi
```

<h3 id="anchor2_0">基本配置和使用</h3>

使用配置文件启动

```
uwsgi --ini uwsgi.ini # 或 uwsgi uwsgi.ini
```

关闭命令

```
uwsgi --stop uwsgi.pid
```

<h4 id="anchor2_1">mac 下的配置项，这里使用brew安装的</h4>
大部分配置项都没有太多注意的点，需要注意的是`plugin`选项和请求处理相关的选项如`http`、`http-socket`、`protocol`
不使用nginx，uwsgi直接负责处理Web请求：

```
[uwsgi]
# 方法①http
# http=127.0.0.1:8080

# 方法②http-socket
# http-socket=127.0.0.1:8080

# 方法③ scoket + protocol
socket=127.0.0.1:8080
protocol=http

# plugin 是通过编译产生，下面会说这个问题
#plugin = ./python_plugin.so

# 项目目录,这里以django项目为例
chdir=/path/to/project_dir
wsgi-file=project_name/wsgi.py
processes=4
threads=2
master=True
pidfile=uwsgi.pid
# 启用日志文件，如果不启用，则在命令行直接展示所有的请求
daemonize=uwsgi.log
virtualenv=/Users/Mike/.virtualenvs/py3_django
```

方法①`http=127.0.0.1:8080`启动，启动时报错，启动失败:

```
The -s/--socket option is missing and stdin is not a socket.
```

方法②`http-socket=127.0.0.1:8080`启动产生warning,能够启动成功:

```
!!!!!!!!!!!!!! WARNING !!!!!!!!!!!!!!
no request plugin is loaded, you will not be able to manage requests.
you may need to install the package for your language of choice, or simply load it with --plugin.
!!!!!!!!!!! END OF WARNING !!!!!!!!!!
```
发起请求无法正确被处理，出现

```
-- unavailable modifier requested: 0 --
```

方法③`socket=127.0.0.1:8080`+`protocol=http`和方法②一模一样，都是缺少python对应的plugin，可以通过源码编译一个python的plugin

```
#cd 到源码目录，注意版本和brew安装的uwsgi版本相同
cd /Users/Mike/Documents/lib/uwsgi/uwsgi-2.0.15
workon py3_django
python uwsgiconfig.py --plugin plugins/python core
# 该目录下多出4个文件：a.out  python_plugin.so  uwsgibuild.lastprofile uwsgibuild.log

cp python_plugin.so /path/to/project # uwsgi.ini同级目录
```
再次编辑uwsgi.ini, 添加如下一行：

```
plugin = ./python_plugin.so
```

这样uwsgi可以正常工作了。

如果配合nginx工作的话，就不能使用上面的方法②和方法③的配置，而要改成使用socket配置项:

```
[uwsgi]
# 配合nginx使用
socket=127.0.0.1:8080

plugin = ./python_plugin.so

# 项目目录,这里以django项目为例
chdir=/path/to/project_dir
wsgi-file=project_name/wsgi.py
processes=4
threads=2
master=True
pidfile=uwsgi.pid
# 启用日志文件，如果不启用，则在命令行直接展示所有的请求
daemonize=uwsgi.log
virtualenv=/Users/Mike/.virtualenvs/py3_django
```

修改nginx配置项：


```
http {
    # .....
    server {
        listen       80;
        # ......
        location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:8080;
        }
    }
}
```
现在可以直接使用127.0.0.1访问了。

<h5 id="anchor2_1_1">django 静态文件搜集</h5>
```
sudo ./manage.py collectstatic
```
django项目通常使用这个命令搜集静态文件，会将所有的静态文件搜集到`settings`里面配置的`STATIC_ROOT`参数指定的目录中，如:

```
STATIC_ROOT = '/var/www/project_name/static'
```

静态文件搜集之后，在nginx配置项中添加对静态文件的访问配置：

```
# ....
location /static {
    alias /var/www/project_name/static/;
}
# ...
```

<h5 id="anchor2_1_2">nginx 负载均衡</h5>
可以在本机或者其他主机多启动几个uwsgi，对处理的请求进行分别处理，如果想以本机进行测试，可以使用多个端口，新增uwsgi服务的配置项为：

```
[uwsgi]
socket=127.0.0.1:8081
plugin = ./python_plugin.so
chdir=/path/to/project_dir
wsgi-file=project_name/wsgi.py
processes=4
threads=2
master=True
pidfile=uwsgi1.pid
daemonize=uwsgi1.log
virtualenv=/Users/Mike/.virtualenvs/py3_django
```

这样，就可以在nginx中增加配置了：

```
http {
    # ...
    upstream project_name {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
    }

    server {
        # ...
        location / {
            include uwsgi_params;
            uwsgi_pass project_name;
        }
    }
}
```

<h4 id="anchor2_2">ubuntu 下的配置项，在虚拟环境中使用pip安装的uwsgi</h4>

在不使用nginx的情况下：
上面提到的方法①②③都是可以直接使用的，而且不要额外添加python plugin,
但是要注意的是在配合使用nginx的时候还是只有`socket`选项即可。

另外一些注意的点：
ubuntu搜集静态文件的时候不要使用sudo

```
./manage.py collectstatic
```

如果出现对目标文件夹访问权限不足的问题,如

```
Permission denied: '/var/www/project_name'
```
可以直接修改对`/var/www`的操作权限

```
sudo chmod -R 777 /var/www
```
再次执行搜集即可。

ubuntu上的安装和配置比mac上简单不少，较少出现问题。


