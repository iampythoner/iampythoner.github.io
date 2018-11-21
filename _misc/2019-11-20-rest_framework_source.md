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


validate, serializers.py:6
run_validation, serializers.py:437
is_valid, serializers.py:236
create, mixins.py:20
dispatch, views.py:480
view, viewsets.py:103
wrapped_view, csrf.py:54
_get_response, base.py:126
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
__call__, deprecation.py:95
inner, exception.py:35
get_response, base.py:81
__call__, wsgi.py:146
__call__, handlers.py:66
run, handlers.py:137
handle, basehttp.py:154
__init__, socketserver.py:696
finish_request, socketserver.py:361
process_request_thread, socketserver.py:639
run, threading.py:864
_bootstrap_inner, threading.py:916
_bootstrap, threading.py:884


Serializer.run_validation

```
def run_validation(self, data=empty):
    """
    We override the default `run_validation`, because the validation
    performed by validators and the `.validate()` method should
    be coerced into an error dictionary with a 'non_fields_error' key.
    """
    (is_empty_value, data) = self.validate_empty_values(data)
    if is_empty_value:
        return data

    value = self.to_internal_value(data)
    try:
        self.run_validators(value)
        value = self.validate(value)
        assert value is not None, '.validate() should return the validated data'
    except (ValidationError, DjangoValidationError) as exc:
        raise ValidationError(detail=as_serializer_error(exc))

    return value
```

self.to_internal_value(data) # 把dict转为了OrderedDict，并且对内部的Field通过serializer创建的Field进行了一一验证，仍然是空值或者关系的验证
self.run_validators 内部使用了self.validators定义验证器对 OrderedDict进行验证，是关系的验证
self.validate 默认实现是return attrs是基于内容的验证，用户可自定义这个方法


最后这个`return value`赋值给了`self._validated_data`是validated_data属性的实际值

没有传入的属性不会生成，这个属性的添加在save的时候添加：
save实现过程中会调用自身的create方法，BaseSerailzer并没有实现create，而子类ModelSerailzer实现了：

BaseSerializer的save

```
def save(self, **kwargs): # 可在kwargs设置没有在请求中传入的额外参数，通常是一些有业务相关的默认值的属性
    # ....

    validated_data = dict(
        list(self.validated_data.items()) +
        list(kwargs.items())
    )

    if self.instance is not None: # 决定是创建还是更新
        self.instance = self.update(self.instance, validated_data)
        assert self.instance is not None, (
            '`update()` did not return an object instance.'
        )
    else:
        self.instance = self.create(validated_data)
        assert self.instance is not None, (
            '`create()` did not return an object instance.'
        )

    return self.instance
```

ModelSerailzer的create

```
def create(self, validated_data):

    raise_errors_on_nested_writes('create', self, validated_data)

    ModelClass = self.Meta.model

    # Remove many-to-many relationships from validated_data.
    # They are not valid arguments to the default `.create()` method,
    # as they require that the instance has already been saved.
    info = model_meta.get_field_info(ModelClass)
    many_to_many = {}
    for field_name, relation_info in info.relations.items():
        if relation_info.to_many and (field_name in validated_data):
            many_to_many[field_name] = validated_data.pop(field_name)

    try:
        instance = ModelClass.objects.create(**validated_data)
    except TypeError:
        tb = traceback.format_exc()
        msg = (
            'Got a `TypeError` when calling `%s.objects.create()`. '
            'This may be because you have a writable field on the '
            'serializer class that is not a valid argument to '
            '`%s.objects.create()`. You may need to make the field '
            'read-only, or override the %s.create() method to handle '
            'this correctly.\nOriginal exception was:\n %s' %
            (
                ModelClass.__name__,
                ModelClass.__name__,
                self.__class__.__name__,
                tb
            )
        )
        raise TypeError(msg)

    # Save many-to-many relationships after the instance is created.
    if many_to_many:
        for field_name, value in many_to_many.items():
            field = getattr(instance, field_name)
            field.set(value)

    return instance
```
这样model在创建的时候自动添加上了有默认值的一些参数