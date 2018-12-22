---
layout: cnblog_post
title:  "dateformat"
permalink: '/misc/dateformat'
date:   2018-12-22 07:34:39
categories: misc
---

>UTC : Coordinated Universal  Time 协调的世界时间，是一种计时方式
>GMT : Greenwich Mean Time 格林尼治平时，零时区时间，实际等同于UTC
>一般情况下，GMT 和 UTC 可以互换，但是实际上，GMT 是一个时区，而 UTC 是一个时间标准。

ISO-8601是时间格式协议：包含以下内容

```
2011-08-12T20:17:46.384Z 

# Z means "zero hour offset" also known as "Zulu time" (UTC)
Z 代表偏移0小时，也就是Zulu时间，即UTC时间
```

以下三个格式表示的时间等价:

```
2011-08-12T20:17:46.384Z
2011-08-12T20:17:46.384
2011-08-12T20:17:46.384+00:00
# 更精确的时间
2011-08-12T20:17:46.384000Z
2011-08-12T20:17:46.384000 
2011-08-12T20:17:46.384000+00:00
```

北京时间使用东八区时间:

```
2018-07-04T19:09:17+08:00
# 等同于以下时间
2018-07-04T11:09:17+00:00
```

注意：代表时区的`+`一旦指定了，决不能再添加`Z`，即使是零时区也不行，下面全是错误的时间格式：

```
2018-07-04T19:09:17+08:00Z # 错误
2018-07-04T19:09:17+00:00Z # 错误
```
也就是说`Z`本身就代表了`+00:00`，不可重复添加，当既没有`Z`，也没有`+xx:xx`的时候就代表零时区时间。

###### 一些Python代码

```
from datetime import timezone, datetime, timedelta

n = datetime.now(timezone.utc)
print(n) # 2018-12-22 09:04:47.902814+00:00
cst = timezone(offset=timedelta(hours=8), name='Beijing')
print(n.astimezone(cst)) # 2018-12-22 17:04:47.902814+08:00
n = datetime.now(cst) # 2018-12-22 17:04:47.902882+08:00
print(n)
print(datetime.now()) # 2018-12-22 17:04:47.902882
print(datetime.now().isoformat()) # 2018-12-22T17:04:47.902882
```

now方法实现：

```
def now(cls, tz=None):
    "Construct a datetime from time.time() and optional time zone info."
    t = _time.time()
    return cls.fromtimestamp(t, tz)
```

Linux 系统  `date`命令输出 `Sat Dec 22 17:10:51 CST 2018`<br/>
这里的CST是 `China Standard Time`, 具体查看参考 [List of time zone abbreviations](https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations)<br>

Django 依赖于第三方库`pytz` 依据settings中的`TIME_ZONE`的值设置时间的offset，

```
# Django
# /xxx/lib/python3.6/site-packages/django/utils/timezone.py
@functools.lru_cache()
def get_default_timezone():
    """
    Return the default time zone as a tzinfo instance.

    This is the time zone defined by settings.TIME_ZONE.
    """
    return pytz.timezone(settings.TIME_ZONE)
```

pytz存储了城市与offset的映射表，很方便地将城市名转为timezone


###### 参考
What is this date format? 2011-08-12T20:17:46.384Z [https://stackoverflow.com/questions/8405087/what-is-this-date-format-2011-08-12t201746-384z#answer-8405125](https://stackoverflow.com/questions/8405087/what-is-this-date-format-2011-08-12t201746-384z#answer-8405125)
ISO_8601 on wiki [https://en.wikipedia.org/wiki/ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)
iso-8601-date-and-time-format [https://www.iso.org/iso-8601-date-and-time-format.html](https://www.iso.org/iso-8601-date-and-time-format.html)
UTC 和ISO 8601时间格式的一些疑问 [https://segmentfault.com/q/1010000004333145](https://segmentfault.com/q/1010000004333145)
python-dateutil [https://github.com/paxan/python-dateutil](https://github.com/paxan/python-dateutil)
Date_format_by_country [https://en.wikipedia.org/wiki/Date_format_by_country](https://en.wikipedia.org/wiki/Date_format_by_country)
List of time zone abbreviations [https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations](https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations)