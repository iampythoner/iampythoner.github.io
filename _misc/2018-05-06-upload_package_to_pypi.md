---
layout: cnblog_post
title:  "upload_package_to_pypi"
permalink: '/misc/upload_package_to_pypi'
date:   2018-05-06 07:34:39
categories: misc
---

真的，先了解一下各种开源协议的区别

[https://www.zhihu.com/question/19568896](https://www.zhihu.com/question/19568896)

[http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html)

```
向PYPI上传包

1.项目配置
  ①setup.py 
    命令：分为标准命令(distutils提供)、额外命令(setuptools、wheel提供)
    常用命令：
        标准：build、clean、install、sdist、register、bdist、check、upload
        额外：develop、alias、test、bdist_wheel
    重要的metadata:看下面的setup.py源码
  ②setup.cfg 包含setup.py脚本命令的默认选项，如果构建和分发包的过程更加复杂，并且需要向
    setup.py命令中传入许多可选参数，那么这个文件非常有用。
  ③MANIFAST.in 文件导出清单

2.在开发期间使用包
    安装包
    setup.py install 或 pip install project_path
    卸载包
    pip uninstall package-name
    开发版
    setup.py develop 或 pip -e

3.上传包
    python setup.py <dist-command> upload

    如果想上传源代码发行版、构建发行版和wheel包
    python setup.py sdist bdist bdist_wheel upload

    ** 先配置~/.pypirc
        [distutils]
        index-servers =
            pypi

        [pypi]
        repository:https://upload.pypi.org/legacy/
        username:mike_chang
        password: ******
    说明:
        sdist 源码发布版 ：生成.tar.gz 源码包
        bdist 预构建发行版：通过4个步骤编译包：
            build_py: 通过字节编译并将其复制到构建文件夹中来构建纯Python模块。
            build_clib:如果包中包含任何C库，它会利用C编译器在构建文件夹中创建一个静态库来构建C库。
            build_ext: 构建C扩展，并像build_clib一样将结果放在构建文件夹中。
            build_scripts: 构建被标记为脚本的模块。如果第一行被设置为`!#`的话，
            它还会修改解释器路径并修改文件模式使其变为可执行文件。
        bdist_wheel 另一种构建发行版：为了替代egg而产生的，这个命令是wheel包对setuptools扩展而提供的，有很多优点：
            更快地安装纯Python包和本地C扩展包
            避免setup.py代码执行
            安装的C扩展不需要windows或OSX上的编译器
            允许更好的缓存，用户测试和持续集成
            创建.pyc文件作为安装的一部分，以确保它们匹配所使用的Python解释器
            在跨平台和跨机器上更一致的安装
        根据PYPA的推荐，wheel应该是默认的分发格式。不行的是，Linux平台特点的wheel还不可用，
        因此如果必须分发带有C扩展的包,那么需要为Linux用户创建sdist发行版。

        几个命令参数的区别：
        python setup.py sdist build
        在build目录生成构建的文件、在dist生成发布的tar.gz

        python setup.py sdist
        在dist生成发布的tar.gz

        python setup.py sdist upload
        在dist生成发布的tar.gz，并上传



    通常这样使用
    python setup.py sdist build
    python setup.py sdist upload

    python setup.py bdist_wheel build
    python setup.py bdist_wheel upload
```


setup.py


```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import io
import re
from collections import OrderedDict
from setuptools import setup

with io.open('README.rst', 'rt', encoding='utf8') as f:
    readme = f.read()

with io.open('flask_session_imp/__init__.py', 'rt', encoding='utf8') as f:
    version = re.search(r'__version__ = \'(.*?)\'', f.read()).group(1)


setup(
    name='Flask-Session-Imp',
    version=version,
    url='https://github.com/mikezone/flask-session',
    project_urls=OrderedDict((
        ('Documentation', 'https://github.com/mikezone/flask-session'),
        ('Code', 'https://github.com/mikezone/flask-session'),
        ('Issue tracker', 'https://github.com/pallets/flask/issues'),
    )),
    license='BSD',
    author='mike_chang',
    author_email='82643885@qq.com',
    maintainer='mike_chang',
    maintainer_email='82643885@qq.com',
    description='Adds server-side session support to your Flask application',
    long_description=readme,
    packages=['flask_session_imp'],
    include_package_data=True,
    zip_safe=False,
    platforms='any',
    # python_requires='>=2.7,!=3.0.*,!=3.1.*,!=3.2.*,!=3.3.*',
    install_requires=[
        'Flask>=0.8'
    ],
    test_suite='test_session',
    classifiers=[
        'Environment :: Web Environment',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License',
        'Operating System :: OS Independent',
        'Programming Language :: Python :: 2',
        'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
        'Topic :: Software Development :: Libraries :: Python Modules'
    ]
)
```