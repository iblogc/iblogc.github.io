title: Django Rest framework入门笔记及跳坑记录
comments: true
date: 2016-12-17 15:03:10
categories:
tags: [Django, restful, api, 问题]
sticky:
---
<br />
<!--more-->
## 更新记录
2016-01-26 初稿

## 序列化时嵌套显示外键关联字段
- 自动
使用`depth`参数指定外键深度

- 手动指定
使用外键对应`model`的小写为属性，外键对应的`model`序列化程序为值
以下例子在`HospitalPic`序列化结果里嵌套显示`Hospital`
models.py
  ```python
  from django.db import models

  class Hospital(models.Model):
      name = models.CharField()
 
  class HospitalPic(models.Model):
      hospital = models.ForeignKey(Hospital)
  ```
serializers.py
  ```python
  from rest_framework import serializers

  class HospitalSerializer(serializers.HyperlinkedModelSerializer):
      class Meta:
          model = Hospital 
          fields = '__all__'
 
 
  class HospitalPicSerializer(serializers.HyperlinkedModelSerializer):
      hospital = HospitalSerializer()
 
      class Meta:
          model = HospitalPic
          fields = '__all__'
  ```
***反向关系嵌套***
在`Hospital`序列化结果里嵌套显示`HospitalPic`
serializers.py
  ```python
  from rest_framework import serializers
  
  class HospitalPicSerializer(serializers.HyperlinkedModelSerializer): 
      class Meta:
          model = HospitalPic
          fields = '__all__'


  class HospitalSerializer(serializers.HyperlinkedModelSerializer):
      hospitalpic_set = HospitalPicSerializer(many=Ture)
      class Meta:
          model = Hospital
          fields = '__all__'
  ```

## 在序列化对象里添加关联表的字段内容
定义一个`serializer Field`，并添加参数`source`指向外键对对应的字段（`source`值其实是从当前序列化的实例的属性）
```python
my_address= serializers.ReadOnlyField(source='address.full_address')
```

## 在序列化对象里添加自定义内容
```python
from django.contrib.auth.models import User
from django.utils.timezone import now
from rest_framework import serializers
 
class UserSerializer(serializers.ModelSerializer):
    days_since_joined = serializers.SerializerMethodField()
 
    class Meta:
        model = User
 
    def get_days_since_joined(self, obj):
        return (now() - obj.date_joined).days
```

## 使用`ViewSet`，并不有设置`queryset`，而是重写了`get_queryset`时，需要在`router`里增加`base_name`参数（`base_name`为`router`为`ViewSet`注册url时自动添加的name前缀，如果未设置则从`ViewSet`的`queryset`里取，使用`ViewSet`自动生成的url name为<base_name>-list <base_name>-detail 等）
views.py
```python
class ContactViewSet(viewsets.ModelViewSet):
    serializer_class = ContactSerializer
    permission_classes = (permissions.IsAuthenticated,)
 
    def get_queryset(self):
        return self.request.user.contact_set.all()
```

urls.py
```python
router.register(r'contact', ContactViewSet, base_name='contact')
```
未设置`base_name`会报下面错误
```
'base_name' argument not specified, and could not automatically determine the name from the viewset, as it does not have a '.queryset' attribute.
```

## 给api接口的url添加了命名空间`namespace`
urls.py
```python
url(r'^api/', include(router.urls, namespace='api')),
```
需要对`HyperlinkedRelatedField`字段的参数进行修改
serializers.py
```python
class HospitalPicSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = HospitalPic
        fields = '__all__'
        extra_kwargs = {
            'url': {'view_name': 'api:hospitalpic-detail'},
            'hospital': {'view_name': 'api:hospital-detail'}
        }
```

不然会出现以下错误
```python
Could not resolve URL for hyperlinked relationship using view name "user-detail". You may have failed to include the related model in your API, or incorrectly configured the `lookup_field` attribute on this field.
```
不过话说我们全api的url加`namespace`一般是为了版本控制，所以有一种简单的方法,只要在settings.py添加基于`namespace`的版本控制，这样就不需要修改`HyperlinkedRelatedField`字段的`view_name`了
urls.py
```python
url(r'^api/v1/', include(router.urls, namespace='v1')),
url(r'^api/v2/', include(router.urls, namespace='v2')),
```
settings.py
```python
REST_FRAMEWORK = {
    ……
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.NamespaceVersioning',
    ……
}
```

## 要drf的错误提示为中文，需要设置
```python
LANGUAGE_CODE = 'zh-CN'
```
如果设置为
```python
LANGUAGE_CODE = 'zh-Hans'
```
虽然django默认表单错误会输出中文，但drf还是输出英文

## django的`validators`可以直接在drf中使用，不需要做任何修改

## 当字段里的属性`editable=False`时，`ModelSerializer`里该字段会抛弃`model`里显式和隐式（unique）的所有`validators`

## `Serializer`里`write_only`写在`field`里和写在`extra_kwargs`里是有区别的，
```python
class UserRegisterSerializer(serializers.ModelSerializer):
    """用户注册Serializer"""
 
    code = serializers.CharField(min_length=4, max_length=6, label=_('验证码'),
                                 help_text=_('验证码'), write_only=True)
    re_password = serializers.CharField(label=_('重复密码'), help_text=_('重复密码'),
                                        validators=validators.password_validators(),
                                        write_only=True)
 
    class Meta:
        model = User
        fields = ('mobile_phone', 'code', 'password', 're_password')
        extra_kwargs = {'password':
                            {'write_only': True}
                        }
 
    def validate(self, attrs):
        """
        Check that the start is before the stop.
        """
        if attrs['password'] != attrs['re_password']:
            raise serializers.ValidationError(_('密码不一致'))
 
        # 校验验证码
        verify_result = Sms(attrs['mobile_phone']).verify_sms_code(
            attrs.pop('code'))
        if not verify_result:
            error = verify_result.get('error')
            raise ParseError(error)
        return attrs
 
    def create(self, validated_data):
        user = User(
            username=validated_data['mobile_phone'],
            mobile_phone=validated_data['mobile_phone'],
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```
因为`create()`这个方法return了一个`user`实例，`User`里没有的字段`code`和`re_password`需要将`write_only `写在`field`参数里，不然会报以下错误
```
AttributeError: Got AttributeError when attempting to get a value for field `code` on serializer `UserRegisterSerializer`.
The serializer field might be named incorrectly and not match any attribute or key on the `User` instance.
Original exception text was: 'User' object has no attribute 'code'.
```

## 如果使用`django-rest-swagger`报以下错误
```
Can't read from server. It may not have the appropriate access-control-origin settings.
```
注释掉设置里的
```python
    # 'base_path': '127.0.0.1:8000/docs',
```

## `serializer.data`和`serializer.validated_data`
在`serializer`只使用`data`参数实例化的时：
- `serializer.data`是原始数据（字符串），`serializer.validated_data`是进行数据验证并转换成对应数据类型的数据。
- 两者者必须在`serializer`调用`is_valid`方法后才能调用
在`serializer`只使用`instance`参数实例化时：
- 只有`serializer.data`没有`serializer.validated_data`，并且`serializer.data`里的数据也是字符串；
- 没有方法`is_valid`；
- 即`is_valid`和`validated_data`只在有data参数实例化时才可调用；

## 在`serializer`里获取原始请求信息
默认的，上下文信息会被传递到`serializer`里，所以在`serializer`可以直接使用`self.context['request']`来获取请求信息。（在要继承自`viewsets.GenericViewSet`的类里使用的`serializer`才能取到，如果是继承`APIView`的，自己传入即可`serializer = self.serializer_class(data=request.data, context={'request': request})`）

## 自定义`serializer`字段
自定义字段继承`serializers.Field`，`to_representation`方法处理出来的数据用来序列化显示，`to_internal_value`处理接收到的数据，`get_attribute`方法指定这个字段访问的实例属性，`get_value`方法指定
```python
class QiNiuField(serializers.Field):
    def get_attribute(self, instance):
        # （序列化时）从模型实例中取一个值给这个字段处理,也可以使用`source`参数指定
        return instance.key
    
    def get_value(self, dictionary):
        # （反序列化时）从传入数据中提取一个值给这个字段处理
        return super(QiNiuField, self).get_value(dictionary)

    def to_representation(self, value):
        # （序列化时）处理出来的数据用来序列化显示
        return value.url

    def to_internal_value(self, data):
        # （反序列化时）处理接收到的数据
        return data['key']
```

## 嵌套序列化，传参问题
官方文档中有这么一个例子[Dealing with nested objects](http://www.django-rest-framework.org/api-guide/serializers/#dealing-with-nested-objects)
如果是以`Content-Type:application/json`形式传数据格式传数据，直接嵌套传就可以了`{'user': {'email': 'foobar', 'username': 'doe'}, 'content': 'baz'}`，但如果是以,
但是如果以`Content-Type:form-data`或`Content-Type:x-www-form-urlencoded`上传，则上传`user`信息进不是嵌套，而是就`.`连接了，`"user.email":"foobar"`.

