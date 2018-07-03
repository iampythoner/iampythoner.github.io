---
layout: cnblog_post
title:  "Python 编码 中文问题"
permalink: '/misc/py_encoding'
date:   2017-12-01 07:34:39
categories: misc
---


### Python 2

##### 文件编码

如果不加文件编码注释 `#-*- coding=utf-8 -*-`，则只要出现了中文字面量，就会报，代码在加载阶段久报错，根本不会执行：

```
SyntaxError: Non-ASCII character '\xe4' in file /xxxx.py on line 2, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

注意这个只是说明了文件编码,而与程序执行过程中的编解码无关,文件编码只是保证了相应的字面量是否合法，如果非要使用汉字而不添加文件编码，需要使用相应的unicode转义字符 如`'\xe4\xb8\xad\xe6\x96\x87'`和`u'\u4e2d\u6587'`（都代表`中文`这两个汉字）。

##### 常见问题

添加了`#-*- coding=utf-8 -*-`之后，文件可以加载了，并且文件中出现的str和unicode字面量都会按照utf-8编码，但是在encode或者decode容易出现问题：

```py
#-*- coding: utf-8 -*-

s = '中文'
u = s.decode() # UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

这是因为str类型decode时会按照系统默认的编码进行decode，而系统默认的编码方式是`ascii`,另外encode也会相同的报错

```py
#-*- coding: utf-8 -*-

s = '中文'
s = s.encode() # UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

这里报的是相同的错误，这是因为s.encode()实际上是相当于调用了`s.decode().encode()`先进行一步`decode()`将`str`转为`unicode`类型,然后再调用`unicode`类型的encode()方法将`unicode`类型转为`str`,因此在执行`decode()`的时候就报错了。

解决上面的两个问题应该在文件开始设置系统默认的编码方式，而不再使用`ascii`(可以通过sys.getdefaultencoding()查看默认编码),设置的方法是:

```py
reload(sys) # Python2.5 初始化后会删除 sys.setdefaultencoding 这个方法，我们需要重新载入
sys.setdefaultencoding('utf-8')
```

所以整个文件这样就没有什么问题了:

```py
#-*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

s = '中文'
u = s.decode()
print(u) # 中文
print(type(u)) # <type 'unicode'>


s = '中文'
s = s.encode()
print(s) # 中文
print(type(s)) # <type 'str'>
```

另外`unicode`类型也有`encode`和`decode`类型，总结起来如下：

| 类型 | 方法 | 返回值类型 | 执行过程 |
| -- | -- | -- | -- |
| str | decode() | unicode | |
| str | encode() | str | a.encode(encoding)相当于<br/>a.decode(系统默认编码).encode(encoding) |
| unicode | encode() | str | |
| unicode | decode() | unicode | a.decode(encoding)相当于<br/>a.encode(系统默认编码).decode(encoding) |

既然有这样的执行过程那么，可以在不使用设置默认的编码的方式，解决`str`类型`decode`的问题和`unicode`类型的`encode`的问题,这里有个str的例子：

```
#-*- coding: utf-8 -*-

s = '中文'
u = s.decode('utf-8') # 照常运行
print(u)
print(type(u))


s = '中文'
s = s.encode() # 报错，因为先使用ascii编码进行decode
print(s)
print(type(s))
```


稍微说一下Python3, Python3默认的文件编码方式就是utf-8 因此不需指定文件编码也可以使用中文字面量<br/>
另外，Python3消除了str和unicode有歧义的部分，使用bytes和str替换了对应Python2中的str和unicode。而Python3的str类型只有encode方法(转为bytes),而不再有decode方法,bytes类型只有decode方法(转为str)，而不再有encode方法，这些对开发者来说都是非常友好的。