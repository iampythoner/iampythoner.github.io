---
layout: cnblog_post
title:  "rest_framework_source"
permalink: '/misc/rest_framework_source'
date:   2018-11-20 07:34:39
categories: misc
---


是否partial更新在与是否在kwargs中传入， wsgi 怎么把partial传给as_view() 返回的view的？？？


HTTP POST => create 方法

```
创建过程的serializer的执行过程 就看create （CreateModelMixin）
①get_serializer == > serializer_class(data=request.data) # request.data loads之后的字典, 将这个值赋值给了initial_data
②is_valid(raise_exception=True) ==> assert hasattr(self, 'initial_data')
# 在 这个not hasattr(self, '_validated_data')前提下，设置 _validated_data:
self._validated_data = self.run_validation(self.initial_data)
self._errors = exc.detail
if self._errors and raise_exception:
    raise ValidationError(self.errors)
return not bool(self._errors)
③self.perform_create(serializer)
④headers = self.get_success_headers(serializer.data)
⑤return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

.data属性

if hasattr(self, 'initial_data') and not hasattr(self, '_validated_data'):
    抛异常，可见在使用data之前必须要调用is_valid,
    而对于传入的如果是instance则不会进入这个if分支

# data属性 惰性加载的实际值
if not hasattr(self, '_data'):
    if self.instance is not None and not getattr(self, '_errors', None):
        self._data = self.to_representation(self.instance)
    elif hasattr(self, '_validated_data') and not getattr(self, '_errors', None):
        self._data = self.to_representation(self.validated_data)
    else:
        self._data = self.get_initial()
```


二、update的实现（UpdateModelMixin）:

```
def update(self, request, *args, **kwargs):
    partial = kwargs.pop('partial', False)
    instance = self.get_object()
    serializer = self.get_serializer(instance, data=request.data, partial=partial)
    serializer.is_valid(raise_exception=True)
    self.perform_update(serializer)

    if getattr(instance, '_prefetched_objects_cache', None):
        # If 'prefetch_related' has been applied to a queryset, we need to
        # forcibly invalidate the prefetch cache on the instance.
        instance._prefetched_objects_cache = {}

    return Response(serializer.data)
```


无论是create、还是update,如果想要自定义validate 就要对is_valid 实现过程中调用的过程函数重写，<br/>
在这个重写的过程中，要考虑到is_valid 是否先验证了这个值的required 而直接报错，这个好像无法动态调整，<br/>
只能先将这个参数设置为非必须的，然后再通过validator 依据其他参数动态验证。<br/>
