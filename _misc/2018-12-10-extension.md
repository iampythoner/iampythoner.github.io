---
layout: cnblog_post
title:  "celery"
permalink: '/misc/extension'
date:   2018-12-10 07:34:39
categories: misc
---


```
from rest_framework import serializers

class HeaderFiled(serializers.CharField):
    def __init__(self, primitive_header_name, **kwargs):
        assert primitive_header_name, 'header_name must not empty value'
        self.primitive_header_name = primitive_header_name
        self.header_name = self.convert_to_django_header(primitive_header_name)
        super(HeaderFiled, self).__init__(**kwargs)

    def run_validation(self, primitive_value):
        error_msg = 'required in header, use key: %s' % (self.primitive_header_name)
        if not self.context:
            raise serializers.ValidationError(detail=error_msg)
        request = self.context.get('request')
        if not request or not request.META.get(self.header_name):
            raise serializers.ValidationError(detail=error_msg)
        header_value = request.META.get(self.header_name)

        if header_value == '' or (self.trim_whitespace and header_value.strip() == ''):
            if not self.allow_blank:
                self.fail('blank')
            return ''
        return header_value
```

```
serializer_class = parameter_serializer_generator(
    user_id=HeaderFiled(max_length=255, primitive_header_name='x-user-id'),
)
serializer = serializer_class(data={}, context=self.get_serializer_context())
serializer.is_valid(raise_exception=True)
```


```
class ChoicesEnumMixin(object):
    @classmethod
    def choices(cls):
        return [(i.value, i.name) for i in list(cls)]
```