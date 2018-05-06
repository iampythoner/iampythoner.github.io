---
layout: cnblog_post
title:  "mac_lang_env"
permalink: '/misc/mac_lang_env'
date:   2018-05-05 07:34:39
categories: misc
---

mac 上各种语言环境的配置

```
Java
$ /usr/libexec/java_home
Unable to find any JVMs matching version "(null)".
No Java runtime present, try --request to install.

$ ls /Library/Java/JavaVirtualMachines

安装完配置完
/Library/Java/JavaVirtualMachines 生成目录
/usr/libexec/java_home 结果为新生成的目录


Python 
安装之前/Library/Frameworks/Python.framework目录不存在
系统自带的Python安装在这个目录/System/Library/Frameworks/Python.framework
安装新的Python环境放在/Library/Frameworks/Python.framework目录

pyenv 查看当前python解释器路径
pyenv which python

如何快速正确的安装 Ruby, Rails 运行环境
https://ruby-china.org/wiki/install_ruby_guide/

```