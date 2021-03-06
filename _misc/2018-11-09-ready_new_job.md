---
layout: cnblog_post
title:  "ready_new_job"
permalink: '/misc/ready_new_job'
date:   2018-11-09 07:34:39
categories: misc
---

new vim

```
w b 移动单词
gg文件首部  G 文件尾部
zz 当前编辑的行放到屏幕中间
zt 放到屏幕顶部
zb 放到屏幕底部
括号跳转 %
翻屏幕 ctrl + f 
ctrl + u
ctrl +b
 
[[ 移动到本文件中上一个class
]] 移动到本文件中下一个class

]M 移动到下一个method
[M 移动到上一个method

easymotion  

插入 a i o大小写

删除
x
d dw dd dt)

s 
S = D + i


修改
修改一个字符
r + 修改后的字符

c
c w 删除后面的一个单词，进入编辑模式

查询
/ 
? 和/方向相反

-------------分屏
:vs 竖分屏
:sp 横分屏
```

## Django

| Release Series	| Release Date | End of mainstream support1	| End of extended support2 |
| ----- | ----- | ----- | ----- |
| 1.9	| December 2015 |	August 2016 |	April 2017 |
| 1.10	| August 2016	| April 2017	| December 2017 |
|1.11 LTS |	April 2017	| December 2017 |	Until at least April 2020 |
| 2.0	| December 2017 |	August 2018	| April 2019 |

The latest version of this table is found on the download page.<br>
参考[https://www.djangoproject.com/weblog/2015/jun/25/roadmap/](https://www.djangoproject.com/weblog/2015/jun/25/roadmap/)

###### 依赖

```
Installing collected packages: pytz, django
Successfully installed django-2.1.3 pytz-2018.7
```


###### django-admin --help

```
Type 'django-admin help <subcommand>' for help on a specific subcommand.

Available subcommands:

[django]
    check
    compilemessages
    createcachetable
    dbshell
    diffsettings
    dumpdata
    flush
    inspectdb
    loaddata
    makemessages
    makemigrations
    migrate
    runserver
    sendtestemail
    shell
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations
    startapp
    startproject
    test
    testserver
```


##### 模板基础

点语法查找顺序：
字典key，属性名，方法名，列表索引

找不到会抛异常，如果变量含有 silent_variable_failure = True属性那么会转为空串


```
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-*(其中×与要安装的软件有关)
```

参考[https://blog.csdn.net/u011092188/article/details/64123561/](https://blog.csdn.net/u011092188/article/details/64123561/)


### ORM 

#### MySQL 驱动

如果使用`django.db.backends.mysql`这个引擎，需要安装mysqlclient 或者 MySQL-python, 另外如果遇到

```
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: dlopen(/Users/mike/.pyenv/versions/3.6.7/envs/django_3_6_7/lib/python3.6/site-packages/_mysql.cpython-36m-darwin.so, 2): Library not loaded: libmysqlclient.18.dylib
  Referenced from: /Users/mike/.pyenv/versions/3.6.7/envs/django_3_6_7/lib/python3.6/site-packages/_mysql.cpython-36m-darwin.so
  Reason: image not found.
```

这个问题，我参考这个解决[https://github.com/PyMySQL/mysqlclient-python/issues/14#issuecomment-436257659](https://github.com/PyMySQL/mysqlclient-python/issues/14#issuecomment-436257659)


如果使用`mysql.connector.django`引擎，安装mysql-connector-python 这个是官方的
参考[https://dev.mysql.com/doc/connector-python/en/connector-python-installation.html#c14809](https://dev.mysql.com/doc/connector-python/en/connector-python-installation.html#c14809)

#### Django model
参考[https://docs.djangoproject.com/en/2.1/intro/tutorial02/](https://docs.djangoproject.com/en/2.1/intro/tutorial02/)

终端中直接操作

```
from django.db import connection
cursor = connection.cursor()
```

###### Field继承关系

```
0: object
1: RegisterLookupMixin, DateTimeCheckMixin, PositiveIntegerRelDbTypeMixin

2: 
Field,
PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)

3:
AutoField, BooleanField, CharField, DateField(DateTimeCheckMixin, Field),
DecimalField, DurationField
FilePathField, FloatField, IntegerField,
IPAddressField, GenericIPAddressField, NullBooleanField,
TextField,
TimeField(DateTimeCheckMixin, Field)
BinaryField
UUIDField

4: 
AutoField子类：BigAutoField
CharField子类：CommaSeparatedIntegerField, EmailField, SlugField, URLField
DateField子类：DateTimeField
IntegerField子类：BigIntegerField、SmallIntegerField
```


```
提示：Django根据属性的类型确定以下信息：
    当前选择的数据库支持字段的类型
    渲染管理表单时使用的默认html控件
    在管理站点最低限度的验证
    使用时需要引入from django.db import models包
AutoField：自动增长的IntegerField，通常不用指定
    不指定时Django会自动创建属性名为id的自动增长属性
BooleanField：布尔字段，值为True或False
NullBooleanField：支持Null、True、False三种值
CharField(max_length=字符长度)：字符串
    参数max_length表示最大字符个数
TextField：大文本字段，一般超过4000个字符时使用
IntegerField：整数
DecimalField(max_digits=None, decimal_places=None)：可以指定精度的十进制浮点数
    参数max_digits表示总位数
    参数decimal_places表示小数位数
FloatField：浮点数
DateField[auto_now=False, auto_now_add=False])：日期
    参数auto_now表示每次保存对象时，自动设置该字段为当前时间，用于"最后一次修改"的时间戳，它总是使用当前日期，默认为false
    参数auto_now_add表示当对象第一次被创建时自动设置当前时间，用于创建的时间戳，它总是使用当前日期，默认为false
    参数auto_now_add和auto_now是相互排斥的，组合将会发生错误
TimeField：时间，参数同DateField
DateTimeField：日期时间，参数同DateField
FileField：上传文件字段
ImageField：继承于FileField，对上传的内容进行校验，确保是有效的图片
```

##### Filed 默认参数

参考[https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/](https://docs.djangoproject.com/zh-hans/2.1/ref/models/fields/)

Django生成CREATE TABLE语句自动为每个字段显式加上NOT NULL，可以通过添加null=True来 指定一个字段允许为NULL, 也就是说null关键字参数默认值是False.

| params | means |
| ----- | ----- | 
| null | null：If True, Django will store empty values as NULL in the database. Default is False. 如果为True，空值将会被存储为NULL，默认为False。|
| blank | blank： If True, the field is allowed to be blank. Default is False. 如果为True，字段允许为空，默认不允许。|

null True 和 blank True的区别[https://stackoverflow.com/questions/8609192/differentiate-null-true-blank-true-in-django](https://stackoverflow.com/questions/8609192/differentiate-null-true-blank-true-in-django)

```
null：如果为True，表示允许为空，默认值是False
blank：如果为True，则该字段允许为空白，默认值是False
    对比：null是数据库范畴的概念，blank是表单验证范畴的
db_column：字段的名称，如果未指定，则使用属性的名称
db_index：若值为True, 则在表中会为此字段创建索引，默认值是False
default：默认值
primary_key：若为True，则该字段会成为模型的主键字段，默认值是False，一般作为AutoField的选项使用
unique：如果为True, 这个字段在表中必须有唯一值，默认值是False
注意：Django会自动为表创建主键字段
    如果使用选项设置某属性为主键字段后，Django不会再创建自动增长的主键字段
    默认创建的主键字段为id，可以使用pk代替，pk全拼为primary key
```

###### 关系

```
关系型数据库的关系包括三种类型：
    ForeignKey：一对多，将字段定义在多的一端中
    ManyToManyField：多对多，将字段定义在任意一端中
    OneToOneField：一对一，将字段定义在任意一端中
可以维护递归的关联关系，使用self指定
```


###### 元选项

```
# 书籍信息模型
class BookInfo(models.Model):
    name = models.CharField(max_length=20) #图书名称

    class Meta: #元信息类
        db_table = 'bookinfo' #自定义表的名字
```

##### 迁移

`python manage.py makemigrations book`

```
Migrations for 'book':
  book/migrations/0001_initial.py
    - Create model Author
    - Create model Book
    - Create model Publisher
    - Add field publisher to book
```

`python manage.py migrate `

create those model tables in your database在数据库中创建这些模型表(执行迁移)

```
Operations to perform:
  Apply all migrations: admin, auth, book, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying book.0001_initial... OK
  Applying sessions.0001_initial... OK
```


`python manage.py sqlmigrate books 0001`

```
BEGIN;
--
-- Create model Author
--
CREATE TABLE `book_author` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `first_name` varchar(30) NOT NULL, `last_name` varchar(40) NOT NULL, `email` varchar(254) NOT NULL);
--
-- Create model Book
--
CREATE TABLE `book_book` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `title` varchar(100) NOT NULL, `publication_date` date NOT NULL);
CREATE TABLE `book_book_authors` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `book_id` integer NOT NULL, `author_id` integer NOT NULL);
--
-- Create model Publisher
--
CREATE TABLE `book_publisher` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `name` varchar(30) NOT NULL, `address` varchar(50) NOT NULL, `city` varchar(60) NOT NULL, `state_province` varchar(30) NOT NULL, `country` varchar(50) NOT NULL, `website` varchar(200) NOT NULL);
--
-- Add field publisher to book
--
ALTER TABLE `book_book` ADD COLUMN `publisher_id` integer NOT NULL;
ALTER TABLE `book_book_authors` ADD CONSTRAINT `book_book_authors_book_id_2a4a45bb_fk_book_book_id` FOREIGN KEY (`book_id`) REFERENCES `book_book` (`id`);
ALTER TABLE `book_book_authors` ADD CONSTRAINT `book_book_authors_author_id_dc6a47c1_fk_book_author_id` FOREIGN KEY (`author_id`) REFERENCES `book_author` (`id`);
ALTER TABLE `book_book_authors` ADD CONSTRAINT `book_book_authors_book_id_author_id_652071c2_uniq` UNIQUE (`book_id`, `author_id`);
CREATE INDEX `book_book_publisher_id_7f77c06a` ON `book_book` (`publisher_id`);
ALTER TABLE `book_book` ADD CONSTRAINT `book_book_publisher_id_7f77c06a_fk_book_publisher_id` FOREIGN KEY (`publisher_id`) REFERENCES `book_publisher` (`id`);
COMMIT;
```

###### 其他数据迁移相关指令

`python manage.py sqlmigrate book 0001`
打印指定迁移名的SQL语句。


`python manage.py inspectdb`
内省数据库中的表，并打印对应的Django模型

`python manage.py diffsettings` 对比默认setting和现在的setting，如果默认中没出现的，则使用#

`python manage.py dbshell` 为指定的数据库运行命令行客户端，如果没有指定则使用默认的数据库。

`python manage.py sqlsequencereset` Prints the SQL statements for resetting sequences for the given app name(s).
打印为指定app重置序列的SQL语句


------------------------------

#### C create
`Publisher.objects.create(kwargs)` 相当于下面

```
p = Publisher(kwargs)
p.save()
return p
```

#### R read
`Publisher.objects.all()` # 返回的不是list类型，而是`QuerySet（django.db.models.query.QuerySet）`类型，
此处是QuerySet<Publisher>

###### 题外：reload module

```
from book.models import Publisher
publisher_list = Publisher.objects.all()
publisher_list

# reload
import importlib
from book import models
importlib.reload(models)
from book.models import Publisher
```

##### 常用的查询方法：

返回list 的过滤器：

```
all()：返回所有的数据
filter()：返回满足条件的数据
exclude()：返回满足条件之外的数据，相当于sql语句中where部分的not关键字
order_by()：返回排序后的数据
```

返回单个数据或值的过滤器：

```
get()：返回单个满足条件的对象
    如果未找到会引发"模型类.DoesNotExist"异常
    如果多条被返回，会引发"模型类.MultipleObjectsReturned"异常
count()：返回当前查询的总条数
aggregate()：聚合
exists()：判断查询集中是否有数据，如果有则返回True，没有则返回False
```

#### 指定查询条件， where

`Publisher.objects.filter(name='Apress')`返回的是`QuerySet<Publisher>`
相当于

```
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE name = 'Apress';
```

###### 多个查询条件

`Publisher.objects.filter(country="U.S.A.", state_province="CA")`相当于

```
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE country = 'U.S.A.' AND state_province = 'CA';
```

###### 使用特殊的关键字参数，进行模糊查询like

`Publisher.objects.filter(name__contains="press")`相当于

```
SELECT id, name, address, city, state_province, country, website
FROM books_publisher
WHERE name LIKE '%press%';
```

###### 其他关键字参数

参考[https://docs.djangoproject.com/en/1.7/ref/models/querysets/#methods-that-return-new-querysets](https://docs.djangoproject.com/en/1.7/ref/models/querysets/#methods-that-return-new-querysets)

```
__exact
__iexact
__icontains(大小写无关的LIKE)
__startswith、istartswith
__endswith、iendswith
__in
__range(SQLBETWEEN)
__gt
__gte
__lt
__lte
__date
__year
__month
__day
__day__gte
__week
__week_day
__quarter
__time
__hour
__minute
__second
__isnull
__regex
__iregex
```

##### 高级查询 F和Q

###### F

`from django.db.models import F`

1.查询阅读量大于评论量的书籍

```
books = Book.objects.filter(read_count__gt=F('comment_count'))
```

相当于SQL:

```
SELECT `id`, `title`, `publisher_id`, `publication_date`, `comment_count`, `read_count` FROM `book` WHERE `read_count` > (`comment_count`)
```

2.查询阅读量大于2倍评论量的书籍 : F对象支持运算

```
books = Book.objects.filter(read_count__gt=F('comment_count')*2)
```

相当于SQL:

```
SELECT `id`, `title`, `publisher_id`, `publication_date`, `comment_count`, `read_count` FROM `book` WHERE `read_count` > ((`comment_count`) *2)
```

`F常用于不同字段之间的比较`


###### Q

`Q`是属性包装器，便于对查询条件包装

`Q(模型属性1__条件运算符=值) | Q(模型属性2__条件运算符=值)`

1.查询阅读量大于20，或编号小于3的图书

```
Book.objects.filter(Q(read_count__gt=20) & Q(id__lt=3))
```

2.查询编号不等于3的书籍

```
Book.objects.filter(~Q(id__exact=3))
```

##### 查询单个对象，get方法

如果查询结果是多个对象（MultipleObjectsReturned），或者没有查找到（DoesNotExist），都会抛出异常

##### 排序，order_by方法

`Publisher.objects.order_by("address")`

多个字段排序:`Publisher.objects.order_by("state_province", "address")`

逆序，字段名前+`-`:
`Publisher.objects.order_by("-name")`

可以在model定义的时候指定默认的排序方式:

```
class Publisher(models.Model):
  # ...
  class Meta:
    ordering = ['name']
```


##### filter + order_by

`Publisher.objects.filter(country="U.S.A.").order_by("-name")`

##### limit 

```
Publisher.objects.order_by('name')[0] # limit 1
Publisher.objects.order_by('name')[0:2] # offset 0 limit 2
# 不支持负索引
```

##### 聚合

```
from django.db.models import Sum
Book.objects.aggregate(Sum('read_count'))
```

#### U update

##### 更新单个对象

```
p = Publisher.objects.get(name='Apress')
p.name = 'Apress Publishing'
p.save()
```

##### 更新多个对象

```
Publisher.objects.filter(id=52).update(name='Apress Publishing')
Publisher.objects.all().update(country='USA')
# 对应单句update SQL语句，不会引起竞态条件
```

#### D delete

```
# 单个
publisher_obj.delete()
# 多个
Publisher.objects.filter(country='USA').delete()
Publisher.objects.all().delete()
# 为了预防误删除掉某一个表内的所有数据，Django要求在删除表内所有数据时显示 使用all()
```


### ORM高级

#### 一对多

```
class Publisher(models.Model):
    # ....

class Book(models.Model):
    # ....
    publisher = models.ForeignKey(Publisher)
```

查找指定书的出版社：

```
b = Book.objects.get(id=50)
b.publisher
```

查找指定出版社的所有图书：

```
p = Publisher.objects.get(name='Apress Publishing')
p.book_set.all()
```

`p.boo_set`是`django.db.models.fields.related_descriptors.create_reverse_many_to_one_manager.<locals>.RelatedManager`类型

如果想在这个查找结果中继续寻找指定的书籍：

```
p.book_set.filter(name__icontains='django')
```


#### 多对多

```
class Author(models.Model):
    # ....

class Book(models.Model):
    # ....
    authors = models.ManyToManyField(Author)
```

查找指定书对应的作者:

```
b = Book.objects.get(id=50)
b.authors.all()

b.authors.filter(first_name='Adrian')
```

查找指定作者对应的书：

```
a = Author.objects.get(first_name='Adrian', last_name='Holovaty')
a.book_set.all()
```

总之是没在Model中定义字段，而是因为其他表的关联关系建立联系的字段，在反向查询的时候需要添加`modelname_set`

#### 关联查询

##### 一般 inner join
查询出版社包含'press'的书籍

```
Book.objects.filter(publisher__name__contains='press')
```

相当于SQL:

```
SELECT `book``id`, `book``title`, `book``publisher_id`, `book``publication_date`, `book``comment_count`, `book``read_count` FROM `book_book` INNER JOIN `book_publisher` ON (`book``publisher_id` = `publisher``id`) WHERE `publisher``name` LIKE BINARY '%press%'
```

查询书名中包含Python的出版社：

```
Publisher.objects.filter(book__title__contains='Python')
```

相当于SQL:

```
SELECT `publisher``id`, `publisher``name`, `publisher``address`, `publisher``city`, `publisher``state_province`, `publisher``country`, `publisher``website`, `publisher``code` FROM `book_publisher` INNER JOIN `book_book` ON (`publisher``id` = `book``publisher_id`)
```

##### 自关联

省市区模型

```
class AreaInfo(models.Model):
    name = models.CharField(max_length=30)
    parent = models.ForeignKey('self',null=True,blank=True)

    class Meta:
        db_table = 'areainfo'
```

```
city = AreaInfo.objects.get(name='广州市')
print(city.name) # 广州市
print(city.parent.name) # 广东省
print(city.areainfo_set.all()) # QuerySet<AreaInfo>
```

##### 模型修改、数据库修改（迁移）

```
修改py文件
创建迁移：python manage.py makemigrations
执行迁移：python manage.py migrate
```

##### Manager

`Book.objects`类型是`django.db.models.manager.Manager`,主要属性有:

```
 'aggregate', 'all', 'annotate', 'auto_created', 'bulk_create', 'check', 'complex_filter', 'contribute_to_class', 'count', 'create', 'creation_counter', 'dates', 'datetimes', 'db', 'db_manager', 'deconstruct', 'defer', 'difference', 'distinct', 'earliest', 'exclude', 'exists', 'extra', 'filter', 'first', 'from_queryset', 'get', 'get_or_create', 'get_queryset', 'in_bulk', 'intersection', 'iterator', 'last', 'latest', 'model', 'name', 'none', 'only', 'order_by', 'prefetch_related', 'raw', 'reverse', 'select_for_update', 'select_related', 'union', 'update', 'update_or_create', 'use_in_migrations', 'using', 'values', 'values_list'
```

可以通过Model的Manager类来管理、简化、HOOK日常的查询工作：

```
class BookManager(models.Manager):
    def title_count(self, keyword):
        return self.filter(title__icontains=keyword).count()
    
class Book(models.Model):
    # ...
    objects = BookManager()
    # other_custom_objects = CustomBookManager()
```

`Book.objects.title_count('python')`

`obejcts.all()` 实际上调用了`get_query_set`，可以重写`get_query_set`来控制all()返回的结果:

```
class CustomBookManager(models.Manager):
def get_query_set(self):
    return super(CustomBookManager, self).get_query_set().filter(author='Roald Dahl')
```

### 表单处理

#### request  django.core.handlers.wsgi.WSGIRequest

```
def hello(request):
    print(type(request))  # django.core.handlers.wsgi.WSGIRequest
    print(dir(request)) 
    return HttpResponse('hello world')
```

dir(request)

```
['COOKIES', 'FILES', 'GET', 'META', 'POST',
'__class__', '__delattr__', '__dict__', '__dir__', '__doc__',
'__eq__', '__format__', '__ge__', '__getattribute__', '__gt__',
'__hash__', '__init__', '__init_subclass__', '__iter__', '__le__',
'__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__',
'__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__',
'__weakref__', '_encoding', '_get_post', '_get_raw_host', '_get_scheme',
'_initialize_handlers', '_load_post_and_files', '_mark_post_parse_error',
'_messages', '_post_parse_error', '_read_started', '_set_post', '_stream',
'_upload_handlers', 'body', 'build_absolute_uri', 'close', 'content_params',
'content_type', 'csrf_processing_done', 'encoding', 'environ', 'get_full_path',
'get_host', 'get_port', 'get_raw_uri', 'get_signed_cookie', 'is_ajax', 'is_secure',
'method', 'parse_file_upload', 'path', 'path_info', 'read', 'readline', 'readlines',
'resolver_match', 'scheme', 'session', 'upload_handlers', 'user', 'xreadlines']
```

##### 属性

| 属性名 | 作用 | 类型 | 例子 |
| ----- | ----- | ----- | ----- |
| COOKIES | request中的cookie | dict | {'csrftoken': 'YCkPQvFchCxD7kJftHkgl9kv2Dg9WvfaDvuFDAmVkw1yNTntb5H3WNEr8ROHMiYH', 'UM_distinctid': '166f370b08de7d-'} |
| FILES | multipart表单中的file | django.utils.datastructures.MultiValueDict | <MultiValueDict: {'t_file': [<InMemoryUploadedFile: xxxx.pdf (application/pdf)>]}>  |
| GET | querystring中的参数 | django.http.request.QueryDict |  {'a': ['b']}> |
| META | environ | dict | {'PATH': ..., 'VIM_PATH': ..., 'QUERY_STRING':,'REMOTE_ADDR':,'HTTP_ACCEPT':..} 注意：header从这里获取，request.META.get("header key") 大写，另外可以从这里获取QUERY_STRING， request.META.get('QUERY_STRING', '')|
| POST | BODY中除了文件的其他所有参数，无论enctype | django.http.request.QueryDict |  {'csrfmiddlewaretoken': ['xjRojfcaWKBCicGn4CDWyN4hGRP4ee73cc1e6kTTZE5xYLkBM00J9rodM5nC41QA'], 'username': ['Mike']}<br/> 注意：如果使用x-www-form-urlencoded传文件，那么body中只是含有一个文件名的参数，这样使用request.POST也能拿到这个参数，但是使用multipart是是拿不到的，这个文件连同文件名都直接交给了request.FILES,|
| body | HTTP BODY | bytes | 但是当使用multipart/form-data时，无论文件是否为空都会报错：You cannot access body after reading from request's data stream|
| content_params |  | dict |  |
| content_type |  | str | application/x-www-form-urlencoded |
| csrf_processing_done |  | bool | True |
| encoding |  |  |  |
| environ |  | dict | 同META |
| method |  | str | POST |
| path |  | str | / |
| path_info |  | str | / |
| resolver_match |  | django.urls.resolvers.ResolverMatch | ResolverMatch(func=book.views.hello, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[]) |
| scheme |  | str | http |
| session |  | django.contrib.sessions.backends.db.SessionStore |  |
| upload_handlers |  | list | [<django.core.files.uploadhandler.MemoryFileUploadHandler object at 0x105646b38>, <django.core.files.uploadhandler.TemporaryFileUploadHandler object at 0x105646b70>] |
| user |  | django.utils.functional.SimpleLazyObject | AnonymousUser |

两种情况：使用multipart/form-data POST只能拿到文件之外的参数
使用x-www-form-urlencoded 可以拿到所有的参数，
只看BODY，不看input类型

##### method list

| 方法名 | 作用 | 例子 |
| ----- | ----- | ----- |
| build_absolute_uri |  | http://127.0.0.1:8000/hello?a=10 |
| close |  | None |
| get_full_path |  | /hello?a=10 |
| get_host |  | 127.0.0.1:8000 |
| get_port |  | 8000 |
| get_raw_uri |  | http://127.0.0.1:8000/hello?a=10 |
| get_signed_cookie |  |
| is_ajax | 请求是否是ajax | False |
| is_secure | 是否是https | False |
| parse_file_upload |  |
| read |  |
| readline |  |
| readlines |  |
| xreadlines |  |


##### 获取header

request.META 中的key

| HTTP header | 获取方式 |
| ----- | ----- |
| Content-Length | CONTENT_LENGTH |
| Content-Type | CONTENT_TYPE |
| Host | HTTP_HOST |
| Connection | HTTP_CONNECTION |
| Cache-Control | HTTP_CACHE_CONTROL |
| Upgrade-Insecure-Requests | HTTP_UPGRADE_INSECURE_REQUESTS |
| User-Agent | HTTP_USER_AGENT |
| Accept | HTTP_ACCEPT |
| Accept-Encoding | HTTP_ACCEPT_ENCODING |
| Accept-Language | HTTP_ACCEPT_LANGUAGE |
| Cookie | HTTP_COOKIE |

一个完整的 META字典打印:

```
{
    'PATH': '/Users/mike/.pyenv/versions/3.7.1/envs/django/bin:/Users/mike/.rvm/bin:/Users/mike/.pyenv/plugins/pyenv-virtualenv/shims:/Users/mike/.pyenv/shims:/Users/mike/.pyenv/bin:/usr/local/Cellar/emacs/25.3/bin:/usr/local/redis/bin:/usr/local/mongodb/bin:/usr/local/mysql/bin:/usr/local/mysql/support-files:/Applications/CMake.app/Contents/bin:/Users/mike/Documents/go/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:.:/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/Users/mike/.pyenv/versions/3.7.1/bin:/Users/mike/.pyenv/libexec:/Users/mike/.pyenv/plugins/python-build/bin:/Users/mike/.pyenv/plugins/pyenv-virtualenv/bin:/Users/mike/.rvm/gems/ruby-2.4.1/bin:/Users/mike/.rvm/gems/ruby-2.4.1@global/bin:/Users/mike/.rvm/rubies/ruby-2.4.1/bin:/Users/mike/.rvm/bin:/Users/mike/.pyenv/plugins/pyenv-virtualenv/shims:/Users/mike/.pyenv/shims:/Users/mike/.pyenv/bin:/usr/local/Cellar/emacs/25.3/bin:/usr/local/redis/bin:/usr/local/mongodb/bin:/usr/local/mysql/bin:/usr/local/mysql/support-files:/Applications/CMake.app/Contents/bin:/Users/mike/Documents/go/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:.:/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home/bin:/Library/Frameworks/Python.framework/Versions/2.7/bin:/Library/Frameworks/Python.framework/Versions/3.6/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/X11/bin:/usr/local/go/bin:/Library/Frameworks/Mono.framework/Versions/Current/Commands',
    'SHELL': '/bin/zsh', 'ZDOTDIR': '', 'SECURITYSESSIONID': '186a5', 'TERM': 'xterm-256color', 'USER': 'mike',
    'COMMAND_MODE': 'unix2003', 'TMPDIR': '/var/folders/yx/wl9qvynx639_vy6y6blgq2280000gn/T/',
    'SSH_AUTH_SOCK': '/private/tmp/com.apple.launchd.GaKP6eiYQM/Listeners',
    'DISPLAY': '/private/tmp/com.apple.launchd.oMPppJo1qE/org.macosforge.xquartz:0', 'XPC_FLAGS': '0x0',
    '__CF_USER_TEXT_ENCODING': '0x1F5:0x19:0x34',
    'Apple_PubSub_Socket_Render': '/private/tmp/com.apple.launchd.WUHTOX8z5h/Render', 'LOGNAME': 'mike',
    'LC_CTYPE': 'zh_CN.UTF-8', 'XPC_SERVICE_NAME': '0', 'HOME': '/Users/mike', 'SHLVL': '1',
    'PWD': '/Users/mike/Documents/xx_pro/first_django', 'OLDPWD': '/Users/mike/Documents/xx_pro/first_django',
    'ZSH': '/Users/mike/.oh-my-zsh', 'PAGER': 'less', 'LESS': '-R', 'LSCOLORS': 'Gxfxcxdxbxegedabagacad',
    'ANDROID_HOME': '/Users/mike/Library/Android/sdk',
    'JAVA_HOME': '/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home',
    'CLASS_PATH': '/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home/lib',
    'PYTHONPATH': '/Library/Frameworks/Python.framework/Versions/3.6/bin', 'GOROOT': '/usr/local/go',
    'GOPATH': '/Users/mike/Documents/go', 'GOBIN': '/Users/mike/Documents/go/bin',
    'VIM_PATH': '/usr/local/Cellar/vim/latest', 'CMAKE_PATH': '/Applications/CMake.app/Contents/bin',
    'WORKON_HOME': '/Users/mike/.virtualenvs',
    'VIRTUALENVWRAPPER_PYTHON': '/Library/Frameworks/Python.framework/Versions/3.6/bin/python3',
    'VIRTUALENVWRAPPER_PROJECT_FILENAME': '.project', 'VIRTUALENVWRAPPER_WORKON_CD': '1',
    'VIRTUALENVWRAPPER_SCRIPT': '/Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh',
    'VIRTUALENVWRAPPER_HOOK_DIR': '/Users/mike/.virtualenvs',
    'HOMEBREW_GITHUB_API_TOKEN': 'fa2f132e6d3842aabbc8cc2dab390147685f2048', 'MYSQL_HOME': '/usr/local/mysql',
    'MYSQL_SERVER': '/usr/local/mysql/support-files', 'MONGODB': '/usr/local/mongodb',
    'REDIS_HOME': '/usr/local/redis', 'EMACS_HOME': '/usr/local/Cellar/emacs/25.3',
    'PYENV_ROOT': '/Users/mike/.pyenv', 'PYENV_SHELL': 'zsh', 'PYENV_VIRTUALENV_INIT': '1', 'LOCAL_MODE': '1',
    'rvm_prefix': '/Users/mike', 'rvm_path': '/Users/mike/.rvm', 'rvm_bin_path': '/Users/mike/.rvm/bin',
    '_system_type': 'Darwin', '_system_name': 'OSX', '_system_version': '10.12', '_system_arch': 'x86_64',
    'rvm_version': '1.29.4 (latest)', 'SSL_CERT_FILE': '/Users/mike/cacert.pem',
    'VIRTUAL_ENV': '/Users/mike/.pyenv/versions/3.7.1/envs/django',
    '_': '/Users/mike/.pyenv/versions/3.7.1/envs/django/bin/python',
    'DJANGO_SETTINGS_MODULE': 'first_django.settings', 'TZ': 'Asia/Shanghai', 'RUN_MAIN': 'true',
    'SERVER_NAME': '1.0.0.127.in-addr.arpa', 'GATEWAY_INTERFACE': 'CGI/1.1', 'SERVER_PORT': '8000',
    'REMOTE_HOST': '', 'CONTENT_LENGTH': '', 'SCRIPT_NAME': '', 'SERVER_PROTOCOL': 'HTTP/1.1',
    'SERVER_SOFTWARE': 'WSGIServer/0.2', 'REQUEST_METHOD': 'GET', 'PATH_INFO': '/book_hello', 'QUERY_STRING': '',
    'REMOTE_ADDR': '127.0.0.1',
    'CONTENT_TYPE': 'text/plain',
    'HTTP_HOST': '127.0.0.1:8000',
    'HTTP_CONNECTION': 'keep-alive',
    'HTTP_CACHE_CONTROL': 'max-age=0',
    'HTTP_UPGRADE_INSECURE_REQUESTS': '1',
    'HTTP_USER_AGENT': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36',
    'HTTP_ACCEPT': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8',
    'HTTP_ACCEPT_ENCODING': 'gzip, deflate, br',
    'HTTP_ACCEPT_LANGUAGE': 'zh-CN,zh;q=0.9,en;q=0.8',
    'HTTP_COOKIE': 'UM_distinctid=166f370b08de7d-046d841bcbb768-1e3f6654-13c680-166f370b08e3ac; username=Mike:1gLisi:FPZjN2ItB8mPmwYjXXC6Suqz4SY; csrftoken=kAX31PkCYzOwXRiwGbIgXHevERhP06wPpD7ltZ9CFSd9kqlTciaTkpCnvMLTjaI1; CNZZDATA1264388688=1009762917-1541682540-http%253A%252F%252F127.0.0.1%253A4000%252F%7C1541916879',
    'wsgi.input': '< _io.BufferedReader name = 5 >',
    'wsgi.errors': '''< _io.TextIOWrapper name = '<stderr>' mode = 'w' encoding = 'UTF-8' >''',
    'wsgi.version': (1, 0), 'wsgi.run_once': False, 'wsgi.url_scheme': 'http', 'wsgi.multithread': True,
    'wsgi.multiprocess': False, 'wsgi.file_wrapper': '''<class 'wsgiref.util.FleWrapper'>''',
    'CSRF_COOKIE': 'kAX31PkCYzOwXRiwGbIgXHevERhP06wPpD7ltZ9CFSd9kqlTciaTkpCnvMLTjaI1'
}
```



### URL 配置


django.urls.resolvers

```
@lru_cache.lru_cache(maxsize=None)
def get_resolver(urlconf=None):
    if urlconf is None:
        from django.conf import settings
        urlconf = settings.ROOT_URLCONF
    return RegexURLResolver(r'^/', urlconf)
```

RegexURLResolver

```
@cached_property
def urlconf_module(self):
    if isinstance(self.urlconf_name, six.string_types):
        return import_module(self.urlconf_name)
    else:
        return self.urlconf_name

@cached_property
def url_patterns(self):
    # urlconf_module might be a valid set of patterns, so we default to it
    patterns = getattr(self.urlconf_module, "urlpatterns", self.urlconf_module)
    try:
        iter(patterns)
    except TypeError:
        msg = (
            "The included URLconf '{name}' does not appear to have any "
            "patterns in it. If you see valid patterns in the file then "
            "the issue is probably caused by a circular import."
        )
        raise ImproperlyConfigured(msg.format(name=self.urlconf_name))
    return patterns
```

##### urls 基本配置，基于函数的视图

```
# views.py
def hello(request):
    return HttpResponse("hello world")

# urls.py # settings.ROOT_URLCONF
urlpatterns = [
    url(r'^hello$', views.hello),
]
```

##### 分模块(app), Including another URLconf

```
# urls.py # settings.ROOT_URLCONF
from django.conf.urls import url, include
urlpatterns = [
    url(r'^order/', include('order.urls', namespace='order'))
]

# urls.py # order.urls
from django.conf.urls import url
import order.views

urlpatterns = [
    url('^detail', order.views.detail, name='detail')
]

# order.views.py
from django.http import HttpResponse
def detail(request):
    return HttpResponse('Detail')
```

##### 基于类的视图， Class-based views

```
# urls.py
urlpatterns = [
    url(r'^$', Home.as_view(), name='home')
]

# views.py
from django.views import View
from django.http import HttpResponse
class Home(View):
    def get(self, request):
        return HttpResponse('Homepage')
```

##### 路径参数 （带参路径）

正则中的每个分组对应着参数

```
urlpatterns = [
    url(r'^getdata/(\d{1,2})/(\d{1,2})$', get_data),
]

def get_data(request, month, day):
    return HttpResponse('{}/{}'.format(month, day))
```

命名参数：

```
url(r'^getdata/(?P<day>\d{1,2})/(?P<month>\d{1,2})$', get_data),
# 这样第一个是day，第二个才是month
```

##### 反向解析

```
from django.shortcuts import redirect
from django.core.urlresolvers import reverse

def hello(request):
    print(reverse('order:detail')) # /order/detail
    return redirect(reverse('order:detail'))
```


log_required 装饰器:

```
def requires_login(view):
    def new_view(request, *args, **kwargs):
        if not request.user.is_authenticated():
            return HttpResponseRedirect('/accounts/login/')
        return view(request, *args, **kwargs)
    return new_view
```

### 常用设置

```
# False的时候不带/的path  不不不会转为带/
# 默认是True, 将不带/的path 301重定向到带/的
APPEND_SLASH = False


# log SQL到控制台
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level': 'DEBUG',
        },
    }
}

# -------------------------------------- # Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases
DATABASES = {
    'default': {
        'ENGINE': 'mysql.connector.django',
        # 'ENGINE': 'django.db.backends.mysql',
        'NAME': 'my_pro',
        'HOST': '192.168.199.130',
        'PORT': '3306',
        'USER': 'mike',
        'PASSWORD': 'Mike123456-',
    }
}


# use db. Default.
# SESSION_ENGINE = 'django.contrib.sessions.backends.db'
# use cache
# SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
# mix
# SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'
```


### Cookie 和 Session

#### cookie 常用操作

```
# 设置
response.set_cookie('mark','hello')
# 读取
c = request.COOKIES.get('mark')
# 判断
if request.COOKIES.has_key('mark'):
    # do something..
# 删除
response.delete_cookie('mark')
```


#### session

session 存储过程：


`request.session[key] = v`

```
两个操作:
①向db中存储INSERT INTO `django_session` (`session_key`, `session_data`, `expire_date`) VALUES ('j8d4s9xli4b2ddy1u9i98yvac40udjyv', 'YWQzNjk3NDRjMDA5Y2IzN2Y0Y2RjYjI1NjQ3YjkyYzc1YjI1Y2YzYTp7ImEiOjEwfQ==', '2018-11-25 12:16:16.933391'); args=('j8d4s9xli4b2ddy1u9i98yvac40udjyv', 'YWQzNjk3NDRjMDA5Y2IzN2Y0Y2RjYjI1NjQ3YjkyYzc1YjI1Y2YzYTp7ImEiOjEwfQ==', b'2018-11-25 12:16:16.933391')
②响应有header： Set-Cookie: sessionid=j8d4s9xli4b2ddy1u9i98yvac40udjyv
```

浏览器请求，带上cookie

`request.session.get(key)`

```
查询
SELECT `django_session`.`session_key`, `django_session`.`session_data`, `django_session`.`expire_date` FROM `django_session` WHERE (`django_session`.`session_key` = 'j8d4s9xli4b2ddy1u9i98yvac40udjyv' AND `django_session`.`expire_date` > '2018-11-11 12:17:55.657736'); args=('j8d4s9xli4b2ddy1u9i98yvac40udjyv', b'2018-11-11 12:17:55.657736')
```

如果再次设置session： `request.session['a'] = 10`

```
①更新
UPDATE `django_session` SET `session_data` = 'YWQzNjk3NDRjMDA5Y2IzN2Y0Y2RjYjI1NjQ3YjkyYzc1YjI1Y2YzYTp7ImEiOjEwfQ==', `expire_date` = '2018-11-25 12:17:55.706915' WHERE `django_session`.`session_key` = 'j8d4s9xli4b2ddy1u9i98yvac40udjyv'; args=('YWQzNjk3NDRjMDA5Y2IzN2Y0Y2RjYjI1NjQ3YjkyYzc1YjI1Y2YzYTp7ImEiOjEwfQ==', b'2018-11-25 12:17:55.706915', 'j8d4s9xli4b2ddy1u9i98yvac40udjyv')

②设置新的过期时间
Set-Cookie: sessionid=j8d4s9xli4b2ddy1u9i98yvac40udjyv; expires=Sun, 25-Nov-2018 12:17:55 GMT; HttpOnly; Max-Age=1209600; Path=/
```


之前的总结

```
def cookie(request):
    response = HttpResponse('hehe')
    response.set_cookie('uname', 'Mike')
    return response

def session(request):
    # from django.contrib.sessions.backends.base import SessionBase
    # from django.contrib.sessions.backends.cached_db import SessionStore
    # SessionStore 继承自 SessionBase


    # 这里的session是SessionStore类型
    # print(request.session)

    # print(request.session.get_expiry_age())
    # if request.session:
    #     # 查询是否过期
    #     print(request.session.get_expiry_age()) # 只是获取了当时设置的过期秒数 这个值不会变化
    #     print(request.session.get_expiry_date())
    #     print(timezone.now())
    #     if request.session.get_expiry_date() >= timezone.now():
    #         return HttpResponse('session 过期 重新登录')
    #     else:
    #         return HttpResponse('自动登录')
    # else:
    #     return HttpResponse('请登录')

    #######注意
    ####### request.session的值是本次会话，是SessionStore实例，确切地说它是一个方便操纵session仓库的工具
    # 它并不会主动根据当前的请求的sessionid取到的之前存储的会话
    # 因此使用它的get_expiry_age()或者get_expiry_date()方法获取过期时间，得到的值永远是两周之后。
    # 如果本次请求传递了sessionid，对它进行的操作都是对那个sessionid对应的Session进行的操作，
    # 如果本次请求没有传递sessionid，对它的操作就是对新创建的session进行的操作，并且在返回的时候存储session同时返回给客户端sessionid

    sessionid = request.COOKIES.get('sessionid')
    print(request.session.exists(sessionid))
    if not sessionid:
        return HttpResponse('需要登录')

    session = Session.objects.get(pk=sessionid)
    print(sessionid)
    print(session.expire_date)
    print(timezone.now())
    if not session or session.expire_date <= timezone.now():
        return HttpResponse('session 过期 重新登录')
    else:
        return HttpResponse('自动登录')

def login_success(request):
    # 直接设置值，如果sessionid不存在会创建Session
    # request.session['is_login'] = True

    # 调用create()方法也会创建Session
    # request.session.create()

    # 调用save()也会创建Session
    # request.session.save()

    # 使用set_expiry方法也会创建Session
    request.session.set_expiry(60 * 60 * 2) # 2小时之后过期
    return HttpResponse('登录成功 设置session')

def logout(request):
    # Deletes the current session data from the session and deletes the session cookie.
    # This is used if you want to ensure that the previous session data can’t be accessed again
    # from the user’s browser (for example, the django.contrib.auth.logout() function calls it).
    request.session.flush()

    # It also has these methods:
    # 只清除数据，但是这个session仍然保留
    # request.session.clear()
    return HttpResponse('退出登录，删除session')
```


```
# 设置Session
request.session['键'] = 值

# 读取Session
request.session.get('键',默认值)

# 删除指定键的值，在存储中只删除某个键对应的值
del request.session['键']

# 清除所有键的值，在存储中删除所有键对应的值
request.session.clear()

# 清除Session数据，在存储中删除所有键及对应的值，形成空表
request.session.flush()

# 设置会话的超时时间
request.session.set_expiry(value)

如果没有指定过期时间则两个星期后过期
如果value是一个整数，会话将在value秒没有活动后过期
如果value为0，那么用户会话的Cookie将在用户的浏览器关闭时过期
如果value为None，那么会话永不过期
```



























