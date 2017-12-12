---
permalink: /misc/django-haystack+whoosh+jiaba
layout: cnblog_post
title:  'django使用haystack、whoosh、jieba'
date:   2017-12-06 06:34:39
categories: misc
---

### django 项目配置haystack和whoosh

1.安装haystack和whoosh

```sh
pip install django-haystack # 与haystack冲突， 先卸载 haystack
pip install whoosh
```

2.settings.py文件中注册应用haystack并做如下配置:

```python
INSTALLED_APPS 添加
'haystack',

HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',
        'PATH': os.path.join(BASE_DIR, 'whoosh_index'),
    }
}

HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
HAYSTACK_SEARCH_RESULTS_PER_PAGE=1
```

3.索引文件生成

在app目录下面创建search_indexes.py文件，创建索引类

```python
from haystack import indexes
from app.goods.models import GoodsSKU

class GoodsSKUIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True)

    def get_model(self):
        return GoodsSKU

    def index_queryset(self, using=None):
        return self.get_model().objects.all()

```

在templates下面新建目录search/indexes/{{ app.name }} 在此目录下面新建一个文件goodssku_text.txt(lower(model_name)_text.txt)，在该文件中指定索引搜索的属性：

```txt
{{ '{{' }} object.name }}
{{ '{{' }} object.desc }}
{{ '{{' }} object.goods.detail }}
```

4.使用命令生成索引文件

```sh
python manage.py rebuild_index
```


### 全文检索的使用

1.配置路由

```python
^search, include('haystack.urls')
```

2.请求参数传递使用q作为filed指定查询的内容

```
example.com/search?q=haha&page=1
```

3.模板处理

在templates/search目录下创建search.html<br>
在模板中可以使用这几个对象：<br>
query：搜索关键字<br>
page：当前页的page对象 –>遍历page对象，获取到的是SearchResult类的实例对象，对象的属性object才是模型类的对象。<br>
paginator：分页paginator对象


### 改变分词方式

1.安装jiaba

```
pip install jieba
```

2.

```
cd ....../site-packages/haystack/backends/
#新建ChineseAnalyzer.py文件
vim ChineseAnalyzer.py
```

```python
import jieba
from whoosh.analysis import Tokenizer, Token

class ChineseTokenizer(Tokenizer):
    def __call__(self, value, positions=False, chars=False,
                 keeporiginal=False, removestops=True,
                 start_pos=0, start_char=0, mode='', **kwargs):
        t = Token(positions, chars, removestops=removestops, mode=mode, **kwargs)
        seglist = jieba.cut(value, cut_all=True)
        for w in seglist:
            t.original = t.text = w
            t.boost = 1.0
            if positions:
                t.pos = start_pos + value.find(w)
            if chars:
                t.startchar = start_char + value.find(w)
                t.endchar = start_char + value.find(w) + len(w)
            yield t

def ChineseAnalyzer():
    return ChineseTokenizer()
```

3.创建中文引擎类

```sh
# 复制whoosh_backend.py
cp whoosh_backend.py whoosh_cn_backend.py
vim whoosh_cn_backend.py
```

from .ChineseAnalyzer import ChineseAnalyzer<br>
查找<br>
analyzer=StemmingAnalyzer()<br>
改为<br>
analyzer=ChineseAnalyzer()<br>

4.settings 修改引擎

```python
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.whoosh_cn_backend.WhooshEngine',
        'PATH': os.path.join(BASE_DIR, 'whoosh_index'),
    }
}
```

5.重建索引
python manage.py rebuild_index






