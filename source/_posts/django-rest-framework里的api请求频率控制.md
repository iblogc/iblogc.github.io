title: Django Rest framework里的API请求频率控制
comments: true
date: 2016-12-17 14:48:19
categories:
tags: [Django, restful, api， version]
sticky:
---
<br />
<!--more-->

## 更新记录
2016-08-25 初稿

`Django Rest framework`有自带的频率控制配置
## 全局设置
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        # 开启匿名用户接口请求频率限制
        'rest_framework.throttling.AnonRateThrottle',
        # 开启授权用户接口请求频率限制
        'rest_framework.throttling.UserRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        # 频率限制有second, minute, hour, day
        # 匿名用户请求频率
        'anon': '100/day',
        # 授权用户请求频率
        'user': '1000/day'
    }
}
```

## 类视图单独配置
```python
from rest_framework.response import Response
from rest_framework.throttling import UserRateThrottle
from rest_framework.views import APIView
 
class ExampleView(APIView):
    throttle_classes = (UserRateThrottle,)
 
    def get(self, request, format=None):
        content = {
            'status': 'request was permitted'
        }
        return Response(content)
```

## 方法视图配置
```python
@api_view(['GET'])
@throttle_classes([UserRateThrottle])
def example_view(request, format=None):
    content = {
        'status': 'request was permitted'
    }
    return Response(content)
```

## 自定义
方法一：
```python
class BurstRateThrottle(UserRateThrottle):
    scope = 'burst'
 
class SustainedRateThrottle(UserRateThrottle):
    scope = 'sustained'
...and the following settings.
```

`settings.py`
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'example.throttles.BurstRateThrottle',
        'example.throttles.SustainedRateThrottle'
    ),
    'DEFAULT_THROTTLE_RATES': {
        'burst': '60/min',
        'sustained': '1000/day'
    }
}
```
然后在视图里设置`throttle_classes`即可。

方法二：
`settings.py`
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.ScopedRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'contacts': '1000/day',
        'uploads': '20/day'
    }
}
```
然后在类视图中设置`throttle_scope `
```python
class ContactListView(APIView):
    throttle_scope = 'contacts'
    ...
 
class ContactDetailView(APIView):
    throttle_scope = 'contacts'
    ...
 
class UploadView(APIView):
    throttle_scope = 'uploads'
    ...
```


**1. 匿名用户频率如果设置大于授权用户频率，则以授权用户频率为准。**
**2. 频率限制是针对单个接口的频率，而不是所有接口的频率。**

