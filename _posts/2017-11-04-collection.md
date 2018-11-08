---
layout: cnblog_post
title:  "collection"
date:   2017-11-04 06:34:39
categories: Python
---


| -- | -- | -- | -- |
| ABC | Inherits from | Abstract Methods | Mixin Methods |
| -- | -- | -- | -- |
| Sized | | `__len__` | |
| Iterable | | `__iter__` | |
| Container | | `__contains__` | |
| Collection | Sized, Iterable, Container | `__contains__, __iter__, __len__	` | | 
| Mapping | Collection | `__getitem__, __iter__, __len__` | `__contains__, keys, items, values, get, __eq__, and __ne__ ` |
| MutableMapping | Mapping | `__getitem__, __setitem__, __delitem__, __iter__, __len__ `| |
| ChainMap、UserDict | MutableMapping | | |
| Counter, OrderedDict, defaultdict | dict | | |


```
issubclass(dict, MutableMapping)  # True

# from collections import ChainMap, Counter, OrderedDict, defaultdict, UserDict
issubclass(ChainMap, Mapping) # True
issubclass(Counter, Mapping) # True
issubclass(OrderedDict, Mapping) # True
issubclass(defaultdict, Mapping) # True
issubclass(UserDict, Mapping) # True
```

```
def my_print(**kwargs):
    print(kwargs)
    print(type(kwargs))

c = Counter(['a', 'b', 'c', 'a'])
my_print(**c)

o = OrderedDict({'key2':'a', 'key1': 'b'})
my_print(**o)

s = 'mississippi'
d = defaultdict(int)
for k in s:
    d[k] += 1
# sorted(d.items()) # [('i', 4), ('m', 1), ('p', 2), ('s', 4)]
my_print(**d)



```
