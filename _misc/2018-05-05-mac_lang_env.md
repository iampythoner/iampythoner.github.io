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
系统自带的Python安装在这个目录/System/Library/Frameworks/Python.framework ,试用pip安装三方库带的二进制工具，也没有安装到这个目录(暂不确定，总之不太方便管理，即使想使用Python2也建议在官网下载安装包安装，而不是直接使用系统自带的)。
安装新的Python环境放在/Library/Frameworks/Python.framework目录(无论是2、3都会放在这个位置)

使用安装包安装Python2，会在写bashrc(Python3安装包会写)，因此python和python2依然是系统自带的，如果想试用自己的安装的Python2，还得手动修改一下bashrc
vim ~/.bash_profile
    PATH="/Library/Frameworks/Python.framework/Versions/2.7/bin:$PATH"
# 为Python2 安装ipython
# 先通过确认一下当前使用的Python和pip2
pip2 -V # pip 9.0.1 from /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages (python 2.7)
# 安装 
pip2 install ipython -i https://pypi.douban.com/simple/

为Python3安装ipython 同上，只不过将pip2 换成pip3
因为我在~/.bash_profile设置Python3的bin目录优先搜索，所以安装完ipython之后，启动的是python3的ipython， 可以直接将/Library/Frameworks/Python.framework/Versions/3.6/bin目录下的ipython直接删掉(这个目录下还有一个ipython3可执行文件),这样就可以做到，ipython启动2的ipython，ipython3启动3的ipython。


# 把上面的都安装完之后再安装pyenv

pyenv 查看当前python解释器路径
pyenv which python

如何快速正确的安装 Ruby, Rails 运行环境
https://ruby-china.org/wiki/install_ruby_guide/

```