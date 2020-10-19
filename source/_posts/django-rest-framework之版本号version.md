---
title: Django REST framework之版本号version
comments: true
date: 2016-01-28 23:29:58
categories: [程序猿]
tags: [Django, restful, api, version]
---
<br />
<!--more-->
drf支持以下形式传输版本号
- header
  ```
  GET /bookings/ HTTP/1.1
  Host: example.com
  Accept: application/json; version=1.0
  ```
- URL Path 
  ```
  GET /v1/bookings/ HTTP/1.1
  Host: example.com
  Accept: application/json
  ```
  ```
  urlpatterns = [
      url(
          r'^(?P<version>(v1|v2))/bookings/$',
          bookings_list,
          name='bookings-list'
      ),
  ]
  ```
- Namespace
  ```
  GET /v1/something/ HTTP/1.1
  Host: example.com
  Accept: application/json
  ```
  ```
  urlpatterns = [
      url(r'^v1/bookings/', include('bookings.urls', namespace='v1')),
      url(r'^v2/bookings/', include('bookings.urls', namespace='v2'))
  ]
  ```
- Host Name
  ```
  GET /bookings/ HTTP/1.1
  Host: v1.example.com
  Accept: application/json
  ```
- Query Parameter
  ```
  GET /something/?version=0.1 HTTP/1.1
  Host: example.com
  Accept: application/json
  ```

drf默认是关闭版本控制功能，如需要开启，可在`settings.py`里添加对应的设置
```python
REST_FRAMEWORK = {
    ……
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.AcceptHeaderVersioning',
    # 'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    # 'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning',
    # 'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.HostNameVersioning',
    # 'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning',
    ……
}
```
当然，你也可以为每个视图单独添加，不过不建议这么做
```python
class ProfileList(APIView):
    versioning_class = versioning.QueryParameterVersioning
```
开启版本控制之后，就可以从`request`取得版本号`request.version`（当然你`settings.py`里配置的是什么方式，就用什么方式传版本号，这样就才可以从`request`里获取到版本号）
```python
def get_serializer_class(self):
    if self.request.version == 'v1':
        return AccountSerializerVersion1
    return AccountSerializer
```

启动版本控制后，url逆向解析方法需要传入`request`参数
```python
from rest_framework.reverse import reverse
 
reverse('bookings-list', request=request)
```

如果是使用Namespace时的版本控制，因为配置了`DEFAULT_VERSIONING_CLASS`，所以设置view_name时不需要添加`v1:`前缀，见django rest framework入门笔记.md

最后在设置里添加以下全局设置来控制能访问的版本
```python
'DEFAULT_VERSION': None, #默认版本，request里没有版本信息时，使用的版本，默认为None
'ALLOWED_VERSIONS': [None, 'v1', 'v2'], #允许访问的版本，如果访问的版本不在列表中，则会抛出异常
```
也可以为每个视图单独设置
```python
from rest_framework.versioning import URLPathVersioning
from rest_framework.views import APIView
 
class ExampleVersioning(URLPathVersioning):
    default_version = ...
    allowed_versions = ...
    version_param = ...
 
class ExampleView(APIVIew):
    versioning_class = ExampleVersioning
```
