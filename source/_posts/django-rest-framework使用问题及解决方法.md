---
title: Django Rest framework使用问题及解决方法
comments: true
date: 2016-12-17 14:58:04
categories:
tags: [Django, restful, api, 问题]
sticky:
---
<br />
<!--more-->

## 更新记录
2016-01-29 初稿

## 问题1
`ViewSet`没有写`serializer_class`属性，而是重写了`get_serializer_class()`方法，出现
```
Cannot use OrderingFilter on a view which does not have either a 'serializer_class' or 'ordering_fields' attribute.
```
原因：因为启用了`rest_framework.filters.OrderingFilter`而没有设置`ordering_fields`
解决方法：`ViewSet`里加`ordering_fields`属性，可是禁用`rest_framework.filters.OrderingFilter`

## 问题2
`ViewSet`没有写`queryset`属性，而是重写了`get_queryset()`方法，出现
```
'base_name' argument not specified, and could not automatically determine the name from the viewset, as it does not have a '.queryset' attribute.
```
解决方法：需要在`urls.py`里给`ViewSet`注册`Router`时添加`base_name`（`base_name`为`router`为`ViewSet`注册url时自动添加的name前缀，如果未设置则从`ViewSet`的`queryset`里取，使用`ViewSet`自动生成的url name为<base_name>-list <base_name>-detail 等）
urls.py
```
router.register(r'users', UserViewSet, base_name='user')
```

## 问题3
给url设置了`namespace`
urls.py
```python
url(r'^api/', include(router.urls, namespace='api')),
```
访问部分接口出现
```
Could not resolve URL for hyperlinked relationship using view name "user-detail". You may have failed to include the related model in your API, or incorrectly configured the `lookup_field` attribute on this field.
```
解决方法1：给所有的`serializer`里包含的外键字段手动设置`view_name`值（注意，继承`HyperlinkedModelSerializer `，会隐式添加一个`HyperlinkedRelatedField`字段`url`，而所有的外键都会变成`HyperlinkedRelatedField`字段，所以需要对两种类型字段手动设置`view_name`值）
serializers.py
```python
class ContactSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Contact
        fields = '__all__'
        extra_kwargs = {
            'url': {'view_name': 'api:contact-detail'},
            'user':{'view_name':'api:user-detail'}
        }  
```
解决方法2：启动drf基于`NameSpace`的版本控制
settings.py
```python
REST_FRAMEWORK = {
    ……
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning',
    ……
}
```