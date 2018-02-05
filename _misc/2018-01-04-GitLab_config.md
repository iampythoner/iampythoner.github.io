---
layout: cnblog_post
title:  "GitLab配置"
permalink: '/misc/GitLab_config'
date:   2018-01-04 07:34:39
categories: misc
---

Ubuntu下配置：

1.安装依赖

```sh
sudo apt update
sudo apt install -y curl openssh-server ca-certificate


sudo apt install -y postfix
```

2.

```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
apt-get install gitlab-ce
```

3.配置并启动服务

```
gitlab-ctl reconfigure # 根据修改后的/etc/gitlab/gitlab.rb文件重新配置
gitlab-ctl restart # 重启服务，包括自带的nginx，unicorn，web项目本省和git服务
```
可以查看是否可以使用了：http://127.0.0.1:8080(unicorn) http://127.0.0.1:80(nginx)
gitlab本身是一个ROR项目，它是通过unicorn服务器部署的，
所以，你的任何请求传递过程是：client-->nginx-->unicorn-->ROR， unicorn默认跑在8080端口，如果想更改unicorn的端口，可以这样更改配置项`unicorn['port']`,如果想通过域名直接访问web页面可以更改配置项`external_url`, 下面是几个更改后的主要配置：

```
sudo vim /etc/gitlab/gitlab.rb
    external_url 'http://code.iampythoner.com'
    gitlab_rails['time_zone'] = 'PRC'
    unicorn['port'] = 18080
```

4.使用自己的nginx

gitlab使用的是自带的nginx服务器，可执行程序放到了`/opt/gitlab/embedded/sbin/`下面，从`embedded`这个词就可以知道这些内容是gitlab内嵌的。如果希望使用我们自己的nginx服务器(讲真的，这个没有必要，很多配置要重新搞)：

```
sudo vim /etc/gitlab/gitlab.rb
    nginx['enable'] = false # 不使用自带的nginx服务器
```

重新加载配置

```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

使用自己的nginx代理：

```
server {
    listen   80;
    server_name   code.iampythoner.com;

    location / {
        proxy_pass  http://127.0.0.1:18080;
    }
}
```

首页js、css加载不出来的解决方法：
nginx配置添加静态文件路由

```
server {
    # ...
    server_name   code.iampythoner.com;

    # ...

    location /assets {
        root /opt/gitlab/embedded/service/gitlab-rails/public;
        index index.html;
    }
}
```

5.基本使用和权限管理
使用：注册必须通过邮箱验证、权限管理




本文主要参考：

```
https://about.gitlab.com/installation/#ubuntu
https://www.jianshu.com/p/808fbf9d972f
```

