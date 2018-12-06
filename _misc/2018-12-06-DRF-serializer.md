---
layout: cnblog_post
title:  "DRF-serializer"
permalink: '/misc/DRF-serializer'
date:   2018-12-06 07:34:39
categories: misc
---

初始化之后 直接.data

```
# class BaseSerializer
@property
    def data(self):
        if hasattr(self, 'initial_data') and not hasattr(self, '_validated_data'):
            msg = (
                'When a serializer is passed a `data` keyword argument you '
                'must call `.is_valid()` before attempting to access the '
                'serialized `.data` representation.\n'
                'You should either call `.is_valid()` first, '
                'or access `.initial_data` instead.'
            )
            raise AssertionError(msg)

        if not hasattr(self, '_data'):
            if self.instance is not None and not getattr(self, '_errors', None):
                self._data = self.to_representation(self.instance)
            elif hasattr(self, '_validated_data') and not getattr(self, '_errors', None): # ②②
                self._data = self.to_representation(self.validated_data)
            else:
                self._data = self.get_initial()
        return self._data
```

如果是用data初始化的，应该说只要data在初始化有值，也就是self.initial_data有值，而没有调用is_valid直接异常
如果有data，并且调用过is_valid，那么_validated_data就有值了(或者_errors有值)，那么走②① (或②③),self._data由initial_data的值生成
如果是用instance 初始化的,走 ②②分支，self._data 由instance 的值生成

instance如何通过to_representation函数变成了self._data这个字典
查找instance所有的Field类型的attribute列表, 这个Field属性可以拿到属性名，属性值, 构建一个字典

由instance到字典，这正是一个序列化过程。

另外看一下反序列化过程

```
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
```

这里的 `serializer.validated_data`其实就是包装了_validated_data的属性
`is_valid` 负责干了这几件事:
① self.initial_data 验证并生成 self._validated_data
② 过程①如果发生错误，则self._validated_data = {} ，self._errors = exc.detail，
如果设置了raise_exception为True会直接抛异常400 BAD request

```
# class BaseSerializer
def is_valid(self, raise_exception=False):
    assert hasattr(self, 'initial_data'), (
        'Cannot call `.is_valid()` as no `data=` keyword argument was '
        'passed when instantiating the serializer instance.'
    )

    if not hasattr(self, '_validated_data'):
        try:
            self._validated_data = self.run_validation(self.initial_data)
        except ValidationError as exc:
            self._validated_data = {}
            self._errors = exc.detail
        else:
            self._errors = {}

    if self._errors and raise_exception:
        raise ValidationError(self.errors)

    return not bool(self._errors)
```

serializer.save() 如何转为object ？？

```
def save(self, **kwargs):
    assert hasattr(self, '_errors'), (  # 必须保证调用过is_valid 不然没有errors这个attri
        'You must call `.is_valid()` before calling `.save()`.'
    )

    assert not self.errors, ( # 必须保证调用过is_valid之后errors没有值
        'You cannot call `.save()` on a serializer with invalid data.'
    )

    # Guard against incorrect use of `serializer.save(commit=False)`
    assert 'commit' not in kwargs, ( # 额外参数(补充字段数据)不能含commit
        "'commit' is not a valid keyword argument to the 'save()' method. "
        "If you need to access data before committing to the database then "
        "inspect 'serializer.validated_data' instead. "
        "You can also pass additional keyword arguments to 'save()' if you "
        "need to set extra attributes on the saved model instance. "
        "For example: 'serializer.save(owner=request.user)'.'"
    )

    assert not hasattr(self, '_data'), ( # 不能再save之前就创建了_data这个attr,
    # 言外之意，使用initial_data创建的_data并不合法，创建之后就别想再将这个model写入数据库
        "You cannot call `.save()` after accessing `serializer.data`."
        "If you need to access data before committing to the database then "
        "inspect 'serializer.validated_data' instead. "
    )

    validated_data = dict(### 使用self.validated_data和额外参数创建要存入的model,
    # 因此initial_data中的参数参与了下面的更新或者创建对象的过程,
    # 由此联想默认put方法实现是不是在创建serializer的时候，同时传入了instance和initial_data
        list(self.validated_data.items()) +
        list(kwargs.items())
    )

    if self.instance is not None: # 更新
        self.instance = self.update(self.instance, validated_data)
        assert self.instance is not None, (
            '`update()` did not return an object instance.'
        )
    else: # 创建
        self.instance = self.create(validated_data)
        assert self.instance is not None, (
            '`create()` did not return an object instance.'
        )

    return self.instance
```

注意最终会返回新更新或者创建后的model,这个值存在self.instance中。

由一个dict最终序列化为model.

回过头来看，is_valid验证过程 `self._validated_data = self.run_validation(self.initial_data)`:

```
# Serializer (BaseSerializer的子类)
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

```
# Field类
def validate_empty_values(self, data):
    """
    Validate empty values, and either:

    * Raise `ValidationError`, indicating invalid data.
    * Raise `SkipField`, indicating that the field should be ignored.
    * Return (True, data), indicating an empty value that should be
        returned without any further validation being applied.
    * Return (False, data), indicating a non-empty value, that should
        have validation applied as normal.
    """
    if self.read_only:
        return (True, self.get_default())

    if data is empty:
        if getattr(self.root, 'partial', False):
            raise SkipField()
        if self.required:
            self.fail('required')
        return (True, self.get_default())

    if data is None:
        if not self.allow_null:
            self.fail('null')
        return (True, None)

    return (False, data)
```

```
# Serializer类
def to_internal_value(self, data):
    """
    Dict of native values <- Dict of primitive datatypes.
    """
    if not isinstance(data, Mapping):
        message = self.error_messages['invalid'].format(
            datatype=type(data).__name__
        )
        raise ValidationError({
            api_settings.NON_FIELD_ERRORS_KEY: [message]
        }, code='invalid')

    ret = OrderedDict()
    errors = OrderedDict()
    fields = self._writable_fields

    for field in fields:
        validate_method = getattr(self, 'validate_' + field.field_name, None)
        primitive_value = field.get_value(data)
        try:
            validated_value = field.run_validation(primitive_value)
            if validate_method is not None:
                validated_value = validate_method(validated_value)
        except ValidationError as exc:
            errors[field.field_name] = exc.detail
        except DjangoValidationError as exc:
            errors[field.field_name] = get_error_detail(exc)
        except SkipField:
            pass
        else:
            set_value(ret, field.source_attrs, validated_value)

    if errors:
        raise ValidationError(errors)

    return ret
```

```
# Field类
def run_validators(self, value):
    """
    Test the given value against all the validators on the field,
    and either raise a `ValidationError` or simply return.
    """
    errors = []
    for validator in self.validators:
        if hasattr(validator, 'set_context'):
            validator.set_context(self)

        try:
            validator(value)
        except ValidationError as exc:
            # If the validation error contains a mapping of fields to
            # errors then simply raise it immediately rather than
            # attempting to accumulate a list of errors.
            if isinstance(exc.detail, dict):
                raise
            errors.extend(exc.detail)
        except DjangoValidationError as exc:
            errors.extend(get_error_detail(exc))
    if errors:
        raise ValidationError(errors)
```


