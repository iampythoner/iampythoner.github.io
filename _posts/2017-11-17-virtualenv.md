---
layout: cnblog_post
title:  "pyenv、virtualenv、virtualenvwrapper"
date:   2017-10-18 06:34:39
categories: Python
---

<h3 id="anchor1_0">pyenv</h3>
pyenv最大的优势是：可以在"全局"管理不同版本的Python,
可以随时配置当前的使用的Python版本，并对其他使用Python解释器的程序生效。<br>
当系统安装多个版本的Python，使用pyenv切换是相当方便的。

<h3 id="anchor2_0">virtualenv和virtualwrapper</h3>
virtualenv创建的不是全局的Python环境，只是一个虚拟的工作环境，
通常用来为一个应用创建一套“隔离”的Python运行环境，
而且只有在激活某个自定义环境时，这个环境对应的Python版本和三方库才会生效。
相比pyenv它是更轻量级的，类似于容器的工作方式。
而pyenv只是对Python版本它的三方库进行分别管理，不能对每个项目进行单独配置。

如果开发者没有经常更改系统默认Python版本的需求，pyenv是不需要安装的。
但是对于项目开发而言，virtualenv基本上是必须的。
而且virtualenv可以创建任意版本的Python和三方库组合的环境，在大部分情况下使用起来完全可以代替pyenv。

virtualwrapper是对virtualenv的提升，对virtualenv指令进行的封装，使得使用更方便。

<h5 id=""></h5>
virtualenv 和 virtualwrapper需要使用pip安装，
因此本身就依赖于系统设定的Python解释器运行，而virtualwrapper更是依赖
VIRTUALENVWRAPPER_PYTHON这个环境变量指定python解释器的路径。


<h4 id="anchor2_1">virtualenv常用操作</h4>
1.使用前创建一个项目的独立目录

```sh
mkdir projecttest
cd projecttest
```
2.创建一个纯净的3.5版本python环境

```sh
➜  projecttest $: virtualenv pure_3_5 --no-site-packages --python=python3.5
```

下面是一些创建成功的输出信息，它拷贝了系统存在的3.5解释器，并安装了setuptools, pip, wheel这三个基本工具。

```txt
Running virtualenv with interpreter /Library/Frameworks/Python.framework/Versions/3.5/bin/python3.5
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.5'
New python executable in /Users/Mike/Documents/projecttest/pure_3_5/bin/python3.5
Also creating executable in /Users/Mike/Documents/projecttest/pure_3_5/bin/python
Installing setuptools, pip, wheel...done.
```

此时当前目录下出现pure_3_5目录

```
➜  projecttest $: ls
➜  projecttest $: pure_3_5
```
这个目录下的bin、include、lib分别对环境操作工具、依赖库、安装的库进行分别管理，其中bin目录下的activate脚本是对虚拟环境的激活程序，
它里面包含了"激活"和"退出指令的实现"并将当前的虚拟环境添加到系统环境变量中，可以通过执行这个脚本来激活当前的虚拟环境：

```sh
➜  projecttest $: source pure_3_5/bin/activate
(pure_3_5) ➜  projecttest $:
```

这个时候可以按照项目需求安装需要的库：

```sh
(pure_3_5) ➜  projecttest $: pip install greenlet
```
安装的任何库都只会和当前的虚拟环境绑定，而不会影响到系统的Python环境。

查看一下当前已经安装的库：(pip、setuptools、wheel这三个是默认安装的)

```txt
(pure_3_5) ➜  projecttest $: pip list
greenlet (0.4.12)
pip (9.0.1)
setuptools (36.7.2)
wheel (0.30.0)
```

3.退出环境
可以使用deactive退出当前环境

```sh
(pure_3_5) ➜  projecttest $: deactivate
➜  projecttest $:
```


<h3 id="anchor3_0">virtaulenvwrapper</h3>
virtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境,它的主要亮点有：<br>
1.可以将所有的虚拟环境整合到一个目录下，方便管理，而不是像virtualenv那样将虚拟环境放置在和项目同一个目录，
这样将每个环境与项目剥离开，使得每个环境的多次使用非常方便。<br>
2.可以非常方便地对虚拟环境进行添加、删除、复制等管理操作，还可以很方便地对使用的环境进行切换。

安装virtaulenvwrapper：(使用了系统默认pip)

```sh
pip3 install virtaulenvwrapper
```

在使用virtualenvwrapper之前，需要进行以下操作：

1.创建用来存放虚拟环境的目录

```sh
mkdir ~/.virtualenvs
```

2.添加以下环境变量(Linux可以在~/.bashrc添加 Mac可以在 ~/.bash_profile添加)，以Mac为例:

```sh
export WORKON_HOME=~/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/Library/Frameworks/Python.framework/Versions/3.6/bin/python3
source /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh
```
如果使用的是Linux，~/.bashrc应该是这样配置的：

```sh
export WORKON_HOME=~/.virtualenvs
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source ~/.local/bin/virtualenvwrapper.sh
```

3.载入刚刚添加的环境

```sh
source ~/.bash_profile
```

这样virtualenvwrapper便可以使用了。下面是几个常用的操作：

```
mkvirtualenv # 新建虚拟环境
lsvirtualenv # 列出所有的虚拟环境
workon # 进入或者切换虚拟环境
deactivate # 退出当前虚拟环境
rmvirtualenv # 删除虚拟环境
```

例如，和刚才一样创建一个纯净的Python3.5环境:

```sh
➜  ~ $: mkvirtualenv --no-site-packages --python=python3.5 pure3_5
(pure3_5) ➜  ~ $:
```
这个命令会在设置的虚拟环境目录下(~/.virtualenvs)创建环境,并执行这个环境的activate脚本。

使用workon切换虚拟环境

```
workon pure3_5
```
