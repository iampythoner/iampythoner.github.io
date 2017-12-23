---
permalink: /misc/interview
layout: cnblog_post
title:  'interview'
date:   2017-12-20 06:34:39
categories: misc
---

----python 语言相关
1.可迭代对象、迭代器、生成器、itertools模块
2.装饰器,常见装饰器@classmethod, @staticmethod, @propert--Python高级编程
3.上下文管理协议 contextlib--Python高级编程
4.元类开发--Python高级编程 PythonCookbook 参数化编程 hasattr getattr, dynamic load: imp模块，
 https://docs.python.org/3/library/modules.html， dynamic run: execute
 内省：https://docs.python.org/3/library/inspect.html
 mop
5.魔法方法、特殊属性  https://docs.python.org/3/reference/datamodel.html#special-method-names
https://docs.python.org/3/library/stdtypes.html#special-attributes
抽象数据类型对应的接口和魔法方法：https://docs.python.org/3/library/collections.abc.html
6.类相关问题 抽象类 新式旧式类 slot 继承、多态 isinstance issubclass super关键字 MRO， 多继承Mixin
7.Python3.3 3.4 3.5 3.6 新特性
8.函数式编程 functools 柯里化 偏函数
9.深拷贝、浅拷贝
10.字符操作、格式化、常用时间相关类和方法
11.mock
12.加密 https://docs.python.org/3/library/crypto.html
--
内存管理tracemalloc 垃圾回收 gc模块https://docs.python.org/3/library/gc.html
GIL锁
2和3 重要区别 http://python.jobbole.com/80006/


----操作系统、主要是多线程问题
并发模型
进程间通信、进程池
线程通信工具、同步工具
协程的发展史 yield、yield from、greenlet、gevent、asyncio

----计算机网络
ARP TCP HTTP WebSocket WebService SOAP RPC
HTTP长连接Transfer-Encoding: chunked TCP粘包
HTTPS
XSRF和XSS
Python常用的网络库

----项目相关
redis
celery
MySQL  悲观锁 乐观锁 优化
文件系统
uwsgi gunicorn
nginx
fabric自动化部署

shell编程
---偏门问题
cgi部署
apache + mod_python部署

