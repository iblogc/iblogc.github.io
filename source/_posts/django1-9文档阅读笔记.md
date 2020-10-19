---
title: Django 1.9文档阅读笔记
comments: true
date: 2016-04-04 15:27:20
categories: [程序猿]
tags: [Django, 文档, 笔记]
---
<br/>
<!--more-->


<!--more-->

## 更新记录
2016-04-04 初稿
2016-06-30 更新内容


## [模型](http://python.usyiyi.cn/django/topics/db/models.html)
[Model _meta API](http://python.usyiyi.cn/django_182/ref/models/meta.html#model-meta-field-api)

限制普通字段的选择范围
`choices `，value-text，
显示`get_foo_display()`

模型继承
1. 抽象基类
  - 默认继承元类
2. 多表继承
  - 一般情况不继承元类
3. 代理模型

多重继承主要用于`mix-in`类

多表继承时使用`parent_link=True`显示指定OneToOne字段


## [模型字段参考](http://python.usyiyi.cn/django/ref/models/fields.html#lazy-relationships)

与尚未定义的模型关联使用模型名字（字符串）而非本身（类）

关联自己使用`self`

`related_name` `relate_query_name`

外键关联到特定字段
`to_field `

限制外键的选择范围（可以是一个字典、一个Q 对象或者一个返回字典或Q对象的可调用对象）
`limit_choices_to `

外键关联对象删除行为
`on_delete`

1.8以后保存模型时，未保存的外键对象将被忽略，除非设置`allow_unsaved_instance_assignment=True`

关联自身的多对多关系默认对称，取消对称设置`symmetrical=False`

`ImageField`中的`height_field`和`width_field`是用来存储存入图片的高度和宽度值的

##[执行查询](http://python.usyiyi.cn/django/topics/db/queries.html#spanning-multi-valued-relationships)

[可自定义查询（高级查找）](http://python.usyiyi.cn/django/howto/custom-lookups.html)
`exclude`多条件查询时是用or关系而不是and关系

`F()`
用于模型内部字段间的比较支持加法、减法、乘法、除法、取模以及幂计算等算术操作
支持.bitand() 和.bitor()位操作，`update()`也可以使用`F()`但有限制（在update 中你不可以使用F() 对象引入join —— 你只可以引用正在更新的模型的字段）

查询集缓存
当只对查询集的部分进行求值时会检查缓存， 但是如果这个部分不在缓存中，那么接下来查询返回的记录都将不会被缓存。
```python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```
```python
>>> queryset = Entry.objects.all()
>>> print queryset[5] # Queries the database
>>> print queryset[5] # Queries the database again
```
```
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print queryset[5] # Uses cache
>>> print queryset[5] # Uses cache
```

`Q()`
可使用Q对象进行复杂查询

判断两相模型实例是否相同，直接使用`==`比较即可

默认批量删除对象时不会调用实例的`delete`方法

拷贝实例，把`pk`设置为`None`再`save`即可（如果是继承的，则`pk`和`id`都需要设置为`None`）

`update()`方法也不会调用模型的`save()`方法，不会发出`pre_save`和`post_save`信号，字段的`auto_now`也不会起作用

一对多关联对象访问会缓存
```python
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # Hits the database to retrieve the associated Blog.
>>> print(e.blog)  # Doesn't hit the database; uses cached version.
```

自定义反向管理器1.7+
```python
from django.db import models
 
class Entry(models.Model):
    #...
    objects = models.Manager()  # Default Manager
    entries = EntryManager()    # Custom Manager
 
b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
```

## [查询集API 参考](http://python.usyiyi.cn/django/ref/models/querysets.html)
## [聚合](http://python.usyiyi.cn/django/topics/db/aggregation.html)
一次创建多条数据（只有一条sql）
`bulk_create`

根据提供的一组`pk`查询出所有对应的对象
`in_bulk`

在查作者列表时要查每个作者有几篇博文
```python
>>> from django.db.models import Count
>>> authors = Author.object.all().annotate(Count('blog'))
# authors[0]作者的博文数
>>> authors[0].blog__count
3
# 或
>>> authors = Author.object.all().annotate(number_of_blog=Count('blog'))
>>> authors[0].number_of_blog
3
```
算出所有作者的年龄总合（不需要其它数据）
```python
>>> ageAuthor.objects.all().aggregate(Sum('age'))
{'age__sum': 26}
```
~~`annotate`和~~`aggregate`都可写入多个注解表达式
`annotate`和`aggregate`可聚合关联对象

对注解进行过滤
```python
# 查询出作者数大于1的书本
# 只有一条sql
Book.objects.annotate(num_authors=Count('authors')).filter(num_authors__gt=1)
```
但顺序不一样，结果也不同，如：
```python
Publisher.objects.annotate(num_books=Count('book')).filter(book__rating__gt=3.0)
Publisher.objects.filter(book__rating__gt=3.0).annotate(num_books=Count('book')) 
```
对注解项进行排序
```python
Book.objects.annotate(num_authors=Count('authors')).order_by('num_authors')
```
`values()`使用注解时要小心，如果`values()`在注解之前，会对结果进行分组，注解会作用在分组上而不是整个查询集上
> 与默认排序交换或order_by()¶

> 在查询集中的order_by() 部分(或是在模型中默认定义的排序项) 会在选择输出数据时被用到，即使这些字段没有在 values() 调用中被指定。这些额外的字段可以将相似的数据行分在一起，也可以让相同的数据行相分离。在做计数时，  就会表现地格外明显：

> 通过例子中的方法，假设有一个这样的模型：

```python
from django.db import models 
class Item(models.Model):
    name = models.CharField(max_length=10)
    data = models.IntegerField() 
    class Meta:
        ordering = ["name"]
```

> 关键的部分就是在模型默认排序项中设置的name字段。如果你想知道每个非重复的data值出现的次数，可以这样写：

```python
# Warning: not quite correct!
Item.objects.values("data").annotate(Count("id"))
```

> ...这部分代码想通过使用它们公共的 data 值来分组 Item对象，然后在每个分组中得到  id 值的总数。但是上面那样做是行不通的。这是因为默认排序项中的 name也是一个分组项，所以这个查询会根据非重复的 (data, name) 进行分组，而这并不是你本来想要的结果。所以，你应该这样改写：

```python
Item.objects.values("data").annotate(Count("id")).order_by()
```

> ...这样就清空了查询中的所有排序项。 你也可以在其中使用 data ，这样并不会有副作用，这是因为查询分组中只有这么一个角色了。

> 这个行为与查询集文档中提到的 distinct() 一样，而且生成规则也一样：一般情况下，你不想在结果中由额外的字段扮演这个角色，那就清空排序项，或是至少保证它仅能访问 values()中的字段。


## 静态文件 
http://python.usyiyi.cn/django/intro/tutorial06.html
http://python.usyiyi.cn/django/ref/templates/builtins.html 

```html
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static user_stylesheet %}" />
<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />
<link rel="stylesheet" type="text/css" href="{% get_static_prefix %}pools/style.css" />
{% get_static_prefix as STATIC_PREFIX %}
<link rel="stylesheet" type="text/css" href="{{ STATIC_PREFIX }}pools/style.css" />
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}"></img>
```

还有`get_media_prefix`


## [模型实例参考](http://python.usyiyi.cn/django/ref/models/instances.html#django.db.models.Model)

从数据库中重新加载值
`Model.refresh_from_db(using=None, fields=None, **kwargs)`

返回模型中当前所有延迟字段的属性名称
` Model.get_deferred_fields()`

验证对象
> 字段的基本验证会最先跑，但不管前面运行是否通过，对于每个字段，如果Field.clean() 方法抛出 ValidationError，那么将不会调用该字段对应的clean_<fieldname>()方法。 但是，剩余的字段的验证方法仍然会执行。
> 先跑`form`里验证，再跑`modle`验证
> 先跑验证器，再跑`clean`
> 先跑单个字段验证，再跑整体验证
> `Model.clean_field()`会覆盖`Model`里所有字段的验证器，但不会对`Form`里的验证器产生影响

验证模型的字段`Model.clean_fields(exclude=None)`
验证模型的完整性`Model.clean()`
验证模型的唯一性`Model.validate_unique(exclude=None)`
调用`full_clean()`时，上面三个方法都会执行（执行顺序即上面的书写顺序），`ModelForm`的`is_valid()`也会执行上所有验证
`Model.full_clean(exclude=None, validate_unique=True)`
 `save()`时，`full_clean()`不会被调用，如果想验证数据，可手动调用

`Model.clean()`时，引发特定字段的异常
使用一个字典实例化`ValidationError`即可或使用`add_error(field, msg)`方法

在数据库字段值的基础上进行简单的算法操作，应该尽量使用`F()`表达式，避免问题竞态条件

> 指定要保存的字段

> 如果传递给save() 的update_fields 关键字参数一个字段名称列表，那么将只有该列表中的字段会被更新。如果你想更新对象的一个或几个字段，这可能是你想要的。不让模型的所有字段都更新将会带来一些轻微的性能提升。例如：

```python
product.name = 'Name changed again'
product.save(update_fields=['name'])
```

> `update_fields` 参数可以是任何包含字符串的可迭代对象。空的`update_fields`可迭代对象将会忽略保存。如果为`None`值，将执行所有字段上的更新。

> 指定`update_fields`将强制使用更新操作。

> 当保存通过延迟模型加载（`only()` 或`defer()`）进行访问的模型时，只有从数据库中加载的字段才会得到更新。这种情况下，有个自动的`update_fields`。如果你赋值或者改变延迟字段的值，该字段将会添加到更新的字段中。

new in 1.9
使用`Model.delete()`删除多表继承的子表数据时，使用``keep_parents=True`可以保留上级数据，默认为`False`
返回值为删除数据的条数

`DateField`和`DateTimeField`字段如果`null=False`则支持下面两个方法
```python 
Model.get_next_by_FOO(**kwargs)¶ 
Model.get_previous_by_FOO(**kwargs)
```
## [管理器](http://python.usyiyi.cn/django/topics/db/managers.html)

django遇到的第一个管理器为默认管理器

如果需要访问关联对象调用关联对象的默认管理器，需要在管理器中加`use_for_related_fields=True`，不然会调用朴素管理器
```python
# -*- coding: utf-8 -*-
 
from django.db import models
 
 
class DefaultManager(models.Manager):
    def get_queryset(self):
        queryset = super(DefaultManager, self).get_quertset().filter(is_delete=False)
        return queryset
 
 
class Author(models.Model):
    name = models.CharField(max_length=100)
    is_delete = models.BooleanField()
    objects = DefaultManager()
 
 
class Post(models.Model):
    author = models.ForeignKey(Author)
    title = models.CharField(max_length=100)
    content = models.TextField()
    is_delete = models.BooleanField()
    objects = DefaultManager()
 
 
author = Author.objects.get(pk=1)
post = Post.objects.get(pk=2)
 
# 调用DefaultManager管理器
author.post_set.all()
# 调用朴素管理器，如果要调用DefaultManager管理器，需要设置DefaultManager管理器的类变量use_for_related_fields=True
post.author
```
*注：朴素管理器里找不到的方法会在默认管理器里查找*

[从Manager中调用自定义的QuerySet](http://python.usyiyi.cn/django/topics/db/managers.html#calling-custom-queryset-methods-from-the-manager)

## [进行原始的SQL查询](http://python.usyiyi.cn/django/topics/db/sql.html)

```python
Manager.raw(raw_query, params=None, translations=None)
```

> django.db.connection对象提供了常规数据库连接的方式。为了使用数据库连接，先要调用connection.cursor()方法来获取一个游标对象之后，调用cursor.execute(sql, [params])来执行sql语句，调用cursor.fetchone()或者cursor.fetchall()来返回结果行。

## [数据库事务](http://python.usyiyi.cn/django/topics/db/transactions.html)

详细笔记见django1.8事务.md

将每个HTTP请求封装在一个数据库事务中
settings中设置`ATOMIC_REQUESTS=True`

单独给一个方法加上数据库事务控制使用`atomic`
```python
from django.db import transaction
 
@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()
```
或
```python
from django.db import transaction
 
def viewfunc(request):
    # This code executes in autocommit mode (Django's default).
    do_stuff()
 
    with transaction.atomic():
        # This code executes inside a transaction.
        do_more_stuff()
```
避免在 atomic里捕获异常!

## 查询表达式
使用数据库的方法
```python
from django.db.models import Func, F 
queryset.annotate(field_lower=Func(F('field'), function='LOWER'))
```
或
```python
class Lower(Func):
    function = 'LOWER' 
queryset.annotate(field_lower=Lower(F('field')))
```

## [条件表达式](http://python.usyiyi.cn/django/ref/models/conditional-expressions.html)
高级用法查看在线版
`When`
`Case`

## [数据库函数](http://python.usyiyi.cn/django/ref/models/database-functions.html)
`Coalesce` 接收一个含有至少两个字段名称或表达式的列表，返回第一个非空的值（空字符串不认为是一个空值）

## [将遗留数据库整合到Django](http://python.usyiyi.cn/django/howto/legacy-databases.html)

根据遗留数据库生成models
```python
python manage.py inspectdb > models.py
```

## [为模型提供初始数据](http://python.usyiyi.cn/django/howto/initial-data.html)

使用fixtures
```
[
  {
    "model": "myapp.person",
    "pk": 1,
    "fields": {
      "first_name": "John",
      "last_name": "Lennon"
    }
  },
  {
    "model": "myapp.person",
    "pk": 2,
    "fields": {
      "first_name": "Paul",
      "last_name": "McCartney"
    }
  }
]
```
导入数据命令
```
python manage.py loaddata <fixturename>
```

## 数据库访问优化

添加索引，比任何查询语法优化都来的重要
理解查询集
QuerySets是延迟的。
什么时候它们被计算出来。
数据在内存中如何存储。

使用`cached_property`装饰器，只要是同一个实例，一个方法就只会执行一次
使用`with`模版标签
使用`iterator`迭代器

在数据库中而不是python中做数据库工作
使用过滤器和反射过滤器对数据进行过滤
使用`F()`表达式
使用注解和聚合
使用原始SQL

用唯一的或被索引的列来检索独立对象

在不同位置多次访问数据库，每次获取一个数据集，不如在一次查询中获取它们。比如循环的时候。

使用`select_related()`和`prefetch_related()`

不检索你不需要的信息
使用`QuerySet.values()`和`QuerySet.values_list()`

使用`QuerySet.defer()`和`QuerySet.only()`

计算数量不要使用`len(queryset)`而是使用`QuerySet.count()`

判断是否存在结果使用`QuerySet.exists()`而不是用`if queryset`

但不要过度使用`count()`和`exists()`，如果你本来就需要里面的数据，那就不要使用

使用`QuerySet.update()`和`QuerySet.delete()`批量操作数据

直接使用外键的值
```python
entry.blog_id
# 而不是
entry.blog.id
```

如果你并在意结果集的顺序，不要进行排序，移除`Meta.ordering`

创建对象时尽可能使用`bulk_create()`来减少sql查询数量
这也适用于`ManyToManyFields`的情况，一起`add`而不是一个一个`add`
```python
my_band.members.add(me, my_friend) 
#更优于 
my_band.members.add(me)
my_band.members.add(my_friend)
```

## [URL调度器](http://python.usyiyi.cn/django/topics/http/urls.html)

url捕获的参数永远是字符串

在根url上获取的参数不影响参数传递
```python
# In settings/urls/main.py
from django.conf.urls import include, url
 
urlpatterns = [
    url(r'^(?P<username>\w+)/blog/', include('foo.urls.blog')),
]
 
# In foo/urls/blog.py
from django.conf.urls import url
from . import views
 
urlpatterns = [
    url(r'^$', views.blog.index),
    url(r'^archive/$', views.blog.archive),
]
```
在上面的例子中，捕获的"username"变量将被如期传递给include()指向的URLconf。

可嵌套
```python
from django.conf.urls import url
 
urlpatterns = [
    url(r'blog/(page-(\d+)/)?$', blog_articles),                  # bad
    url(r'comments/(?:page-(?P<page_number>\d+)/)?$', comments),  # good
]
```

传递额外的参数
```python
from django.conf.urls import url
from . import views
 
urlpatterns = [
    url(r'^blog/(?P<year>[0-9]{4})/$', views.year_archive, {'foo': 'bar'}),
]
```
当url捕获的参数和字典中传递的参数同名时，将忽略url捕获的参数而使用字典里的参数值

传递额外的参数给`include()`
```python
# main.py
from django.conf.urls import include, url
 
urlpatterns = [
    url(r'^blog/', include('inner'), {'blogid': 3}),
]
 
# inner.py
from django.conf.urls import url
from mysite import views
 
urlpatterns = [
    url(r'^archive/$', views.archive),
    url(r'^about/$', views.about),
]
```
效果等同
```python
# main.py
from django.conf.urls import include, url
from mysite import views
 
urlpatterns = [
    url(r'^blog/', include('inner')),
]
 
# inner.py
from django.conf.urls import url
 
urlpatterns = [
    url(r'^archive/$', views.archive, {'blogid': 3}),
    url(r'^about/$', views.about, {'blogid': 3}),
]
```

[反查带命名空间的URL](http://python.usyiyi.cn/django/topics/http/urls.html#reversing-namespaced-urls)

## [编写视图](http://python.usyiyi.cn/django/topics/http/views.html)

`HttpResponse`子类，状态码
- `HttpResponseRedirect` 临时重定向，302
- `HttpResponsePermanentRedirect` 永久重定向，301
- `HttpResponseNotModified` 没有任何修改，304
- `HttpResponseBadRequest` 语义有误码，当前请求不被服务器理解，400
- `HttpResponseNotFound` 页面没找到，404
- `HttpResponseForbidden` 服务器理解请求，但拒绝执行，403
- `HttpResponseNotAllowed` 请求中指定的请求方式不能用于请求相应资源，405
- `HttpResponseGone` 请求的资源在服务器上已经不可用，而且没有已知的转发地址，410
- `HttpResponseServerError` 服务器遇到了一个意外的错误，导致无法完成对请求的处理，500
- `HttpResponse(status=201)` 自定义返回状态码

重写错误视图（在url中）
```
handler404 = 'mysite.views.my_custom_page_not_found_view'
handler500 = 'mysite.views.my_custom_error_view' 
handler403 = 'mysite.views.my_custom_permission_denied_view'
handler400 = 'mysite.views.my_custom_bad_request_view' 
```

## [Django 的快捷函数](http://python.usyiyi.cn/django/topics/http/shortcuts.html)

`template_name`可传一个模版序列，django将使用存在的第一个模版

`redirect(to, [permanent=False, ]*args, **kwargs)[source]`
> 为传递进来的参数返回HttpResponseRedirect 给正确的URL 。 
> 参数可以是：
>  
>     一个模型：将调用模型的get_absolute_url() 函数
>     一个视图，可以带有参数：将使用urlresolvers.reverse 来反向解析名称
>     一个绝对的或相对的URL，将原样作为重定向的位置。
>  
> 默认返回一个临时的重定向；传递permanent=True 可以返回一个永久的重定向。

`get_object_or_404(klass, *args, **kwargs)`可以传Model也可以传QuerySet实例和关联的管理器
`get_list_or_404(klass, *args, **kwargs)`可以传Model也可以传QuerySet实例和关联的管理器

## [视图装饰器](http://python.usyiyi.cn/django/topics/http/decorators.html)
[按需内容处理](http://python.usyiyi.cn/django/topics/conditional-view-processing.html )
`django.views.decorators.http`包里的装饰器可以基于请求的方法来限制对视图的访问。若条件不满足会返回 django.http.HttpResponseNotAllowed。
`require_http_methods(request_method_list)`限制视图只能服务于规定的http方法（需要大写）
`require_GET()`
`require_POST()`
`require_safe()`只允许视图接受GET和HEAD方法的装饰器。

```python
@condition(etag_func=None, last_modified_func=None)
```
```python
@last_modified(last_modified_func)
```
根据最后修改时间来决定是否运行视图，可减少流量
```python
@etag(etag_func)
```
`etag`（版本？）和`last_modified`不能同时使用

`GZip`对内容进行压缩，节省流量，但增加处理时间

`vary_on_cookie`
`vary_on_headers`
基于特定的请求头部来控制缓存

`never_cache`

## [Request 对象和Response 对象](http://python.usyiyi.cn/django/ref/request-response.html)

`HttpRequest`对象(除非特殊说明，所有属性都是只读，`session`属性是个例外)
`HttpRequest.scheme` 请求方案（通常为http或https）
`HttpRequest.body` 字节字符串，表示原始http请求正文
`HttpRequest.path` 字符串，表示请求的页面的完整路径，不包含域名
`HttpRequest.path_info`    在某些Web 服务器配置下，主机名后的URL 部分被分成脚本前缀部分和路径信息部分。path_info 属性将始终包含路径信息部分，不论使用的Web 服务器是什么。使用它代替path 可以让代码在测试和开发环境中更容易地切换。 
    例如，如果应用的WSGIScriptAlias 设置为"/minfo"，那么当path 是"/minfo/music/bands/the_beatles/" 时path_info 将是"/music/bands/the_beatles/"。
`HttpRequest.method` 请求使用的http方法，大写
`HttpRequest.encoding` 表示提交的数据的编码方式，可写
`HttpRequest.GET`
`HttpRequest.POST`
`HttpRequest.REQUEST`不建议使用，使用`GET`和`POST`代替
`HttpRequest.COOKIES` 字典，键和值都是字符串
`HttpRequest.FILES` 类似字典的对象，包含所有的上传文件，<form>带有`enctype="multipart/form-data"`才会有数据
`HttpRequest.META` 标准的python字典，包含所有http请求头部
`HttpRequest.user`
`HttpRequest.session` 类似字典的对象
`HttpRequest.urlconf` 如果其它地方设置了，则用来取代`ROOT_URLCONF `
`HttpRequest.resolver_match` 会在url解析之后设置，一个`ResolverMatch`实例，表示解析之后的url
`HttpRequest.get_host()` 获取原始主机地址
`HttpRequest.get_port()` 获取请求端端口号
`HttpRequest.get_full_path()` 返回完整的path，包括查询字符串
`HttpRequest.build_absolute_uri(location)` 返回绝对url
`HttpRequest.get_signed_cookie(key, default=RAISE_ERROR, salt='', max_age=None)` 返回签名过的Cookie对应的值
`HttpRequest.is_secure()` 如果请求是通过https发起的，则返回True
`HttpRequest.is_ajax()` 如果请求是通过XMLHttpRequest发起的，则返回True
```python 
HttpRequest.read(size=None)
HttpRequest.readline()
HttpRequest.readlines()
HttpRequest.xreadlines()
HttpRequest.__iter__()
```
这几个方法实现类文件的接口用于读取HttpRequest· 实例

`QueryDict`对象
request.POST 和request.GET 的QueryDict 在一个正常的请求/响应循环中是不可变的。若要获得可变的版本，需要使用.copy()。

## [TemplateResponse 和SimpleTemplateResponse](http://python.usyiyi.cn/django/ref/template-response.html)
`SimpleTemplateResponse`
`TemplateResponse`
TemplateResponse 对象和普通的django.http.HttpResponse 一样可以用于任何地方。它可以用来作为render() 和render_to_response() 的另外一种选择。

例如，下面这个简单的视图使用一个简单模板和包含查询集的上下文返回一个TemplateResponse：
```python
from django.template.response import TemplateResponse
 
def blog_index(request):
    return TemplateResponse(request, 'entry_list.html', {'entries': Entry.objects.all()})
```

## [文件上传](http://python.usyiyi.cn/django/topics/http/file-uploads.html)
```python
def handle_uploaded_file(f):
    with open('some/file/name.txt', 'wb+') as destination:
        for chunk in f.chunks():
            destination.write(chunk)
```
遍历UploadedFile.chunks()，而不是使用read()，能确保大文件并不会占用系统过多的内存。

上传处理器
```
("django.core.files.uploadhandler.MemoryFileUploadHandler",
"django.core.files.uploadhandler.TemporaryFileUploadHandler",)
```
MemoryFileUploadHandler 和TemporaryFileUploadHandler一起提供了Django的默认文件上传行为，将小文件读取到内存中，大文件放置在磁盘中。

你可以编写自定义的处理器，来定制Django如何处理文件。例如，你可以使用自定义处理器来限制用户级别的配额，在运行中压缩数据，渲染进度条，甚至是向另一个储存位置直接发送数据，而不把它存到本地。关于如何自定义或者完全替换处理器的行为，详见编写自定义的上传处理器。

更改上传处理器的行为
``` 
DEFAULT_FILE_STORAGE
FILE_CHARSET
FILE_UPLOAD_HANDLERS
FILE_UPLOAD_MAX_MEMORY_SIZE
FILE_UPLOAD_PERMISSIONS
FILE_UPLOAD_TEMP_DIR
MEDIA_ROOT
MEDIA_URL
```

在运行中更改上传处理器
```pyhton
request.upload_handlers.insert(0, ProgressBarUploadHandler())
```

>  注意

>  你只可以在访问request.POST或者request.FILES之前修改上传处理器-- 在上传处理工作执行之后再修改上传处理就毫无意义了。如果你在读取request.FILES之后尝试修改request.upload_handlers，Django会抛出异常。

>  所以，你应该在你的视图中尽早修改上传处理器。

>  CsrfViewMiddleware 也会访问request.POST，它是默认开启的。意思是你需要在你的视图中使用csrf_exempt()，来允许你修改上传处理器。接下来在真正处理请求的函数中，需要使用csrf_protect()。注意这意味着处理器可能会在CSRF验证完成之前开始接收上传文件。例如：

> ```from django.views.decorators.csrf import csrf_exempt, csrf_protect 
@csrf_exempt
def upload_file_view(request):
    request.upload_handlers.insert(0, ProgressBarUploadHandler())
    return _upload_file_view(request) 
@csrf_protect
def _upload_file_view(request):
    ... # Process request
    ```
 ```

## [File对象](http://python.usyiyi.cn/django/ref/files/file.html)
`File`类
`ContentFile`类
`ImageFile`类 比`File`多了`width`和`height`属性
附加到对象的文件有额外的方法
 ```
File.save(name, content[, save=True])
```
提供文件名和内容保存一个新文件，不会替换已存在文件，但会创建一个新文件，并且更新对象来指向它。
测试出来直接`car.save()`也不会覆盖已存在文件，如果有重写会在原有名字后面加字符串
如果save为True，模型的save()方法会在文件保存之后调用。这就是说，下面两行：
​```python
>>> car.photo.save('myphoto.jpg', content, save=False)
>>> car.save()
```
等价于：
```python
>>> car.photo.save('myphoto.jpg', content, save=True)
```


从模型实例中移除文件，并且删除内部文件
```
 File.delete([save=True])
```
在页面展示中，`ImageFile`自带的清除勾选框勾选后只是清除了数据库中这具字段的值，并不会删除文件系统里对应的文件，而`File.delete()`会删除文件系统里的文件

## [文件储存API](http://python.usyiyi.cn/django/ref/files/storage.html)

`DefaultStorage`
`FileSystemStorage`
`Storage`

## [管理文件](http://python.usyiyi.cn/django/topics/files.html)

```python
from django.db import models
 
class Car(models.Model):
    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    photo = models.ImageField(upload_to='cars')
```
`photo`有以下方法
`photo.path`相对路径
`photo.url`绝对路径

*实际测试有出入*
```python
# 官方示例
>>> car.photo.path
'/media/cars/chevy.jpg'
>>> car.photo.url
'http://media.example.com/cars/chevy.jpg'

# 实际测试结果
>>> car.photo.path
'E:\workspace\parking\parking\upload\20151230171832_0.jpg'
>>> car.photo.url
'/upload/20151230171832_0.jpg'
```

更改一个文件的存储位置
```python
>>> import os
>>> from django.conf import settings
>>> initial_path = car.photo.path
>>> car.photo.name = 'cars/chevy_ii.jpg'
>>> new_path = settings.MEDIA_ROOT + car.photo.name
>>> # Move the file on the filesystem
>>> os.rename(initial_path, new_path)
>>> car.save()
>>> car.photo.path
'/media/cars/chevy_ii.jpg'
>>> car.photo.path == new_path
True
```

## [编写自定义存储系统](http://python.usyiyi.cn/django/howto/custom-file-storage.html)

1. 必须是`django.core.files.storage.Storage`的子类
2. Django必须能够不带任何参数来实例化
3. 必须实现 _open() 和 _save()方法，以及任何适合于你的储存类的其它方法
4. 你的储存类必须是 可以析构的，所以它在迁移中的一个字段上使用的时候可以被序列化。只要你的字段拥有自己可以序列化的参数，你就可以为它使用django.utils.deconstruct.deconstructible类装饰器（这也是Django用在FileSystemStorage上的东西）

## [基于类的视图](http://python.usyiyi.cn/django/topics/class-based-views/index.html)

## [基于类的内建通用视图](http://python.usyiyi.cn/django/topics/class-based-views/generic-display.html)

`ListView`类视图中，默认的对象列表名除了`object_list`，还有一个`<model_name>_list`

## [使用基于类的视图处理表单](http://python.usyiyi.cn/django/topics/class-based-views/generic-editing.html)

如果对应模型存在`get_absolute_url`方法的前提下`CreateView`和`UpdateView`类视图的`success_url`默认使用`get_absolute_url`

如何定义`form_class`，即使`form_class`是`ModelForm`也还是需要指定模型

如果没有定义`form_class`，则必须定义`fields`，`fields`和`form_class`不能同时存在

如果模型某个字段存的是模板路径，并且想通过此字段来动态的控制表单页的模板，可通过`template_name_field`来指定此字段。

## [Mixin](http://python.usyiyi.cn/django/topics/class-based-views/mixins.html)

## [基于类的视图的Mixin](http://python.usyiyi.cn/django/ref/class-based-views/mixins.html)
`ContextMixin`所有基于类的通用视图的这个模板Context 都包含一个view 变量指向视图实例。
> Use alters_data where appropriate
注意，将视图实例包含在模板Context 中可能将有潜在危险的方法暴露给模板作者。为了避免在模板中被调用类似这样的方法，可以在这些方法上设置alters_data=True。更多信息，参见渲染模板Context 的文档。
很显然，调用某些变量会带来副作用，允许模板系统访问它们将是愚蠢的还会带来安全漏洞。
    每个Django 模型对象的delete() 方法就是一个很好的例子。模板系统不应该允许下面的行为：
    I will now delete this valuable data. {{ data.delete }}
    设置可调用变量的alters_data 属性可以避免这点。如果变量设置alters_data=True ，模板系统将不会调用它，而会无条件使用string_if_invalid 替换这个变量。Django 模型对象自动生成的delete() 和save() 方法自动 设置alters_data=True。 例如：
```python
def sensitive_function(self):
        self.database_record.delete()
    sensitive_function.alters_data = True
```
> 有时候，处于某些原因你可能想关闭这个功能，并告诉模板系统无论什么情况下都不要调用变量。设置可调用对象的do_not_call_in_templates 属性的值为True 可以实现这点。模板系统的行为将类似这个变量是不可调用的（例如，你可以访问可调用对象的属性）。
`query_pk_and_slug`如果为`True`,`get_object()`将使用两者一起来查找。可以防止只使用`pk`时，如果`pk`连续，直接被攻击者都遍历`pk`获取整个列表


## [内建基于类的视图的API](http://python.usyiyi.cn/django/ref/class-based-views/index.html)


```python
urlpatterns = [
    url(r'^view/$', MyView.as_view(size=42)),
]
```
> 视图参数的线程安全性
传递给视图的参数在视图的每个实例之间共享。这表示不应该使用列表、字典或其它可变对象作为视图的参数。如果你真这么做而且对共享的对象做过修改，某个用户的行为可能对后面访问同一个视图的用户产生影响。

## [基于类的通用视图 —— 索引](http://python.usyiyi.cn/django/ref/class-based-views/flattened-index.html)

## [使用Django输出CSV](http://python.usyiyi.cn/django/howto/outputting-csv.html)

```python
import csv
from django.http import HttpResponse
 
def some_view(request):
    # Create the HttpResponse object with the appropriate CSV header.
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'
 
    writer = csv.writer(response)
    writer.writerow(['First row', 'Foo', 'Bar', 'Baz'])
    writer.writerow(['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"])
 
    return response
```

## [使用Django输出PDF](http://python.usyiyi.cn/django/howto/outputting-pdf.html)

```python
from reportlab.pdfgen import canvas
from django.http import HttpResponse
 
def some_view(request):
    # Create the HttpResponse object with the appropriate PDF headers.
    response = HttpResponse(content_type='application/pdf')
    response['Content-Disposition'] = 'attachment; filename="somefilename.pdf"'
 
    # Create the PDF object, using the response object as its "file."
    p = canvas.Canvas(response)
 
    # Draw things on the PDF. Here's where the PDF generation happens.
    # See the ReportLab documentation for the full list of functionality.
    p.drawString(100, 100, "Hello world.")
 
    # Close the PDF object cleanly, and we're done.
    p.showPage()
    p.save()
    return response
```

## [中间件](http://python.usyiyi.cn/django/topics/http/middleware.html)

中间件的顺序很重要
接受请求时，自上向下调用中间件
返回响应时，自下向上调用中间件
`process_request(request)`
在django决定执行哪个视图之前（也就是解析url之前）被调用
返回`None`继续处理请求
返回`HttpResponse`不再去调用其它的request、view 或exception 中间件，或对应的视图，直接调用响应阶段的中间件，并返回结果

`process_view(request, view_func, view_args, view_kwargs)`
*注：`view_args`和`view_kwargs`都不包含`request`*
在django调用视图之前被调用
返回`None`继续处理请求
返回`HttpResponse`不再去调用其它的view 或exception 中间件，或对应的视图，直接调用响应阶段的中间件，并返回结果
> 注意
在中间件内部，从process_request 或process_view 中访问request.POST 或request.REQUEST 将阻碍该中间件之后的所有视图无法修改请求的上传处理程序，一般情况下要避免这样使用。
类CsrfViewMiddleware可以被认为是个例外，因为它提供csrf_exempt() 和csrf_protect()两个装饰器，允许视图显式控制在哪个点需要开启CSRF验证。

`process_template_response(request, response)`
在视图刚好执行完毕之后被调用
必须返回一个实现了`render`方法的响应对象

`process_response(request, response)`
在所有响应返回浏览器之前被调用
必须返回`HttpResponse`或者`StreamingHttpResponse`对象
***[处理流式响应](http://python.usyiyi.cn/django/topics/http/middleware.html#dealing-with-streaming-responses)***

`process_exception(request, exception)`
在视图抛出异常时被调用
返回`None`
返回`HttpResponse` `process_template_response`和响应中间件会被调用
**在处理响应期间，中间件的执行顺序是倒序执行的，这包括process_exception，如果一个中间件的`process_exception`返回了一个响应，那么这个中间件上面的中间件中的`process_exception`都不会被调用**

`__init__()`
大多数中间件类都不需要初始化方法
django初始化中间件无需任何参数，所以不能定义一个有参数的`__init__方法
`__init__`不会每次请求都执行，只在Web服务器响应第一个请求时执行
标记中间件不被使用
`__init__`抛出`django.core.exceptions.MiddlewareNotUsed`异常，django会从中间件处理过程中移动这部分中间件，并且当DEBUG为True的时候在django.request记录器中记录调试信息。

- 中间件类不能是任何类的子类
- 中间件可以放在python路径中的任务位置
正常
```
A.init
B.init
C.init
D.init
A.process_request
B.process_request
C.process_request
D.process_request
A.process_view
B.process_view
C.process_view
D.process_view
 
D.process_template_response
C.process_template_response
B.process_template_response
A.process_template_response
D.process_responst
C.process_responst
B.process_responst
A.process_responst
```
视图异常
```
A.init
B.init
C.init
D.init
A.process_request
B.process_request
C.process_request
D.process_request
A.process_view
B.process_view
C.process_view
D.process_view

D.process_responst
C.process_responst
B.process_responst
A.process_responst
```

## [django中可用的中间件](http://python.usyiyi.cn/django/ref/middleware.html#middleware-ordering )

### `class CommonMiddleware`
`DISALLOWED_USER_AGENTS`禁用匹配的`user-agents`访问网站
`APPEND_SLASH`如果url结尾没有斜杠结尾，并且没有找到匹配的url，django会在结尾加上斜杠再匹配一次
`PREPEND_WWW`如果url会重定向到www到头的网址
`USE_ETAGS`设置来处理ETag。如果设置USE_ETAGS为True，Django会通过MD5-hashing处理页面的内容来为每一个页面请求计算Etag，并且如果合适的话，它将会发送携带Not Modified的响应。
### `class BrokenLinkEmailsMiddleware`
向`MANAGERS` 发送死链提醒邮件

### `class GZipMiddleware`
为支持`GZip`压缩的浏览器压缩内容
建议放在中间件配置列表的第一个
可通过`gzip_page()`装饰器使用独立的`GZip`压缩

### `class ConditionalGetMiddleware`

### `class LocaleMiddeware`
基于请求中的数据开启语言选择，它可以为每个用户进行定制。

### `class MessageMiddleware`
开启基于`Cookie`和会话的消息支持

### `class SecurityMiddleware`

[中间件的排序](http://python.usyiyi.cn/django/ref/middleware.html#middleware-ordering )

## [模版](http://python.usyiyi.cn/django/topics/templates.html)

`DjangoTemplates`引擎`OPTIONS`配置项中接受以下参数
`string_if_invalid`当模版变量无效时，使用此值代替
可使用
comment 
和
endcomment
进行多行注释

## [Django模版语言](http://python.usyiyi.cn/django/ref/templates/language.html )

当模版系统遇到`.`时，按下面顺序查询
从技术上来说，当模版系统遇到点(".")，它将以这样的顺序查询：
- 字典查询（Dictionary lookup）
- 属性或方法查询（Attribute or method lookup）
- 数字索引查询（Numeric index lookup）

模版变量最终解释成字面量，而不是变量值

load
可接受多个库名称
load humanize i18n
load
不支持继承

## [内置标签与过滤器](http://python.usyiyi.cn/django/ref/templates/builtins.html)

### 标签
`filter`对一段内容进行过滤，使用`|`对多个过滤器进行连接，且过滤器可以有参数
*比如一段纯文本不能使用之前说的过滤器写法，则可以使用`filter`*
`firstof`输出第一个不为`False`的参数
```
{% firstof var1 var2|safe var3 "<strong>fallback value</strong>"|safe %}
```
`ifchanged`检查循环中的一个值从最近一次重复其是否改变，支持`else

`with`可往`include`的模版里传上下文件变量
```
{% include "name_snippet.html" with person="Jane" greeting="Hello" %}
```
```
{% include "name_snippet.html" with greeting="Hi" only %}
```
```
{% lorem %}
```
设计人员工具，好像是生成随机单词和段落
```django
{% lorem %}
{% lorem 3 p  %}
{% lorem 10 w random %}
```

## [人性化](http://python.usyiyi.cn/django/ref/contrib/humanize.html)
`apnumber`转换整数或整数的字符串形式为英文描述
1 会变成one
`intcomma`转换成第三位带一个逗号
4500 会变成 4,500
`intword`将大的整数转换为友好的文字表示
1000000 会变成 1.0 million
`naturalday`对于当天或者一天之内的日期， 返回“今天”，“明天”或者“昨天”，视情况而定。否则，使用传进来的格式字符串给日期格式化
`naturaltime`对于日期时间的值，返回一个字符串来表示多少秒、分钟或者小时之前
例如（其中“现在”是2007年2月17日16时30分0秒）：
17 Feb 2007 16:30:00 会变成 now
17 Feb 2007 16:29:31 会变成 29 seconds ago
`ordinal`将一个整数或是整数的字符串，转换为它的序数词
1 会变成 1st
2 会变成  2nd
3 会变成  3rd

## [Django 模板语言：面向Python程序员](http://python.usyiyi.cn/django/ref/templates/api.html)

`string_if_invalid`建议只在调试时设置，调试完成后就关闭，开发时最好不要使用，不然可能会遇到渲染问题

每个上下文都包含`True` `False` `None`

### [使用`Context`对象]
*[这里比较难理解](http://python.usyiyi.cn/django/ref/templates/api.html#playing-with-context-objects)*
```python
Context.get(key, otherwise=None)
Context.pop()
Context.push()
Context.update(other_dict)
```

> 上下文处理器应用的时机
> 上下文处理器应用在上下文数据的顶端。也就是说，上下文处理器可能覆盖你提供给Context 或RequestContext 的变量，所以要注意避免与上下文处理器提供的变量名重复。
> 如果想要上下文数据的优先级高于上下文处理器，使用下面的模式：
> ```python
> from django.template import RequestContext
> request_context = RequestContext(request)
> request_context.push({"my_name": "Adrian"})
> ```
```
Django 通过这种方式允许上下文数据在render() 和 TemplateResponse 等API 中覆盖上下文处理器。
你还可以赋予`RequestContext `一个额外的处理器列表，使用第三个可选的位置参数processors。在下面的示例中，RequestContext 实例获得一个ip_address 变量
​```python
def some_view(request):
    # ...
    c = RequestContext(request, {
        'foo': 'bar',
    }, ['ip_address':'127.0.0.1'])
    return HttpResponse(t.render(c))
```
上面例子中`ip_address`也会加入到上下文中

### 内建的模板上下文处理器
下面是内奸的上下文处理器所添加的内容
`django.contrib.auth.context_processors.auth`
- `user`
- `perms`

`django.template.context_processors.debug`
- debug
- sql_queryes
一个{'sql': ..., 'time': ...} 字典的列表，表示请求期间到目前为止发生的每个SQL 查询及花费的时间。这个列表按查询的顺序排序，并直到访问时才生成。

`django.template.context_processors.i18n`
- `MEDIA_URL`

`django.template.context_processors.static`
- `STATIC_URL`

`django.template.context_processors.csrf`
- `csrf_token`

`django.template.context_processors.request`
- `request`

`django.contrib.messages.context_processors.messages`
- `messages`
- `DEFAULT_MESSAGE_LEVELS`


## [自定义模板标签和过滤器](http://python.usyiyi.cn/django/howto/custom-template-tags.html)

### 自定义过滤器
```python
from django import template
register = template.Library()
@register.filter(name='cut')
 
register.filter('cut', cut)
register.filter('lower', lower)
# or
def cut(value, arg):
    return value.replace(arg, '')
 
@register.filter
def lower(value):
    return value.lower()
```
可使用`SafeData`来验证是否是安全数据
```python
if isinstance(value, SafeData):
    # Do something with the "safe" string.
    ...
```
或使用`is_safe`来控制只接收的安全的数据
```python
@register.filter(is_safe=True)
def myfilter(value):
    return value
```

### 自定义标签
```python
import datetime
from django import template
 
register = template.Library()
 
@register.simple_tag
def current_time(format_string):
    return datetime.datetime.now().strftime(format_string)
```

```
{% show_results poll %}
```

写一个标签，实现下面的效果
```html
<ul>
  <li>First choice</li>
  <li>Second choice</li>
  <li>Third choice</li>
</ul>
```
例子1开始
```python
@register.inclusion_tag('results.html')
def show_results(poll):
    choices = poll.choice_set.all()
    return {'choices': choices}
```

`results.html`
```html
<ul>
{% for choice in choices %}
    <li> {{ choice }} </li>
{% endfor %}
</ul>
```
例子1结束

可使用`takes_context=True`直接访问上下文件中的数据
```python
@register.inclusion_tag('link.html', takes_context=True)
def jump_link(context):
    # 因为takes_context=True所以这里的context就是上下文，可以从里面拿想要的数据，如果有多个参数，方法里的第一个参数名必须是context
    return {
        'link': context['home_link'],
        'title': context['home_title'],
    }
```
`link.html`
```html
<a href="{{ link }}">{{ title }}</a>.
```
页面直接写
```html
{% jump_link %}
```

位置参数和关键字参数和`python`语法一样
```python
@register.inclusion_tag('my_template.html')
def my_tag(a, b, *args, **kwargs):
    warning = kwargs['warning']
    profile = kwargs['profile']
    ...
    return ...
```
```django
{% my_tag 123 "abcd" book.title warning=message|lower profile=user.profile %}
```
还有一个`register.assignment_tag`与`register.simple_tag`功能一样，不知道有什么特殊作用

## [使用表单](http://python.usyiyi.cn/django/topics/forms/index.html)

一些表单输入自带有html5的验证，要禁用这些验证可以设置`form`标签的`novalidate`属性

`is_bound`可以判断一个表单是否具有绑定数据
```python
# 未绑定表单
f = ContactForm()
data = {'subject': 'hello',
        'message': 'Hi there',
        'sender': 'foo@example.com',
        'cc_myself': True}
# 已绑定的表单
f = ContactForm(data)
```

当表单通过`is_valid()`方法验证后，可以直接在`form.cleaned_data`中拿值，并且是已经转换好的`python`格式的数据，但仍然可以从`request.POST`直接访问到未验证的数据。

表单排列
`{{ form.as_table }}`
`{{ form.as_p }}`
`{{ form.as_ul }}`

表单属性
`{{ form.name }}`字段html标签
`{{ form.name.label_tag }}`字段的`lable`html标签
`{{ form.name.id_for_label }}`字段`lable`标签上的`for`值，也是字段标签上的`id`

`{{ form.hidden_fields }}`隐藏字段列表
`{{ form.visible_fields }}`显示的字段列表


错误信息
`{{ form.non_field_errors }}`不是特定字段的错误
`{{ form.errors }}`全部错误，一个字典
`{{ form.name.errors }}`字段错误

可从`form`从遍历出`field`
`{{ field }}`有以下属性
`{{ field.label }}``Model`或是`Form`上的`label`的值
`{{ field.label_tag }}`整个`label`标签，包含冒号
`{{ field.id_for_label }}`字段的id
`{{ field.value }}`字段的值
`{{ field.html_name }}`字段的`name`，考虑表单的前缀
`{{ field.help_text }}`字段的帮助文档
`{{ field.errors }}`字段的错误
`{{ field.is_hidden }}`判断字段是否隐藏
`{{ field.field }}`表单类中`Field`的实例，可以使用它来访问`Field`属性，如
```python
name.field.max_length
```

## [表单 API](http://python.usyiyi.cn/django/ref/forms/api.html)
```python
# 未绑定表单
f = ContactForm()
data = {'subject': 'hello',
        'message': 'Hi there',
        'sender': 'foo@example.com',
        'cc_myself': True}
# 已绑定的表单
f = ContactForm(data)
```
表单实例一但创建，数据不可更改

### `Form.clean()`
### `Form.is_valid()`

### `Form.errors`
> `Form.errors`
访问errors 属性可以获得错误信息的一个字典：
```python
>>> f.errors
{'sender': ['Enter a valid email address.'], 'subject': ['This field is required.']}
```
在这个字典中，键为字段的名称，值为表示错误信息的Unicode 字符串组成的列表。错误信息保存在列表中是因为字段可能有多个错误信息。
你可以在调用is_valid() 之前访问errors。表单的数据将在第一次调用is_valid() 或者访问errors 时验证。
验证只会调用一次，无论你访问errors 或者调用is_valid() 多少次。这意味着，如果验证过程有副作用，这些副作用将只触发一次。

### `Form.errors.as_data()`
> 返回一个字典，它映射字段到原始的ValidationError 实例

### `Form.errors.as_json(escape_html=False)`
> 返回JSON 序列化后的错误。

### `Form.add_error(field, error)`
> 这个方法允许在Form.clean() 方法内部或从表单的外部一起给字段添加错误信息
> Form.add_error() 会自动删除cleaned_data 中的相关字段

### `Form.has_error(field, code=None)`
> 这个方法返回一个布尔值，指示一个字段是否具有指定错误code 的错误。当code 为None 时，如果字段有任何错误它都将返回True。
> 若要检查非字段错误，使用NON_FIELD_ERRORS 作为field 参数。

### `Form.non_field_errors()`
> 这个方法返回Form.errors 中不是与特定字段相关联的错误。它包含在Form.clean() 中引发的ValidationError 和使用Form.add_error(None, "...") 添加的错误。

未绑定表单的行为
验证没有绑定数据的表单是没有意义的，下面的例子展示了这种情况：

```python
>>> f = ContactForm()
>>> f.is_valid()
False
>>> f.errors
{}
```

### `Form.initial`
```python
>>> f = ContactForm(initial={'subject': 'Hi there!'})
```
这些值只显示在没有绑定的表单中，即使没有提供特定值它们也***不会作为后备的值***。
优先级高于`Form`中的`initial`
```python
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='class')
...     url = forms.URLField()
...     comment = forms.CharField()
>>> f = CommentForm(initial={'name': 'instance'}, auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="instance" /></td></tr>
<tr><th>Url:</th><td><input type="url" name="url" /></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>
```

### `Form.has_changed()`
**也有`Field.has_changed()`方法**
检查表单数据是否从初始数据发生改变
当提交表单时，我们可以重新构建表单并提供初始值，这样可以实现比较：
```python
>>> f = ContactForm(request.POST, initial=data)
>>> f.has_changed()
```
如果request.POST 中的数据与initial 中的不同，has_changed() 将为True，否则为False。 计算的结果是通过调用表单每个字段的Field.has_changed() 得到的。

`Form.fields`
从表单中访问字段
是一个`OrderedDict`
可你可以修改表单实例的字段来改变字段在表单中的表示：
```python
>>> f.as_table().split('\n')[0]
'<tr><th>Name:</th><td><input name="name" type="text" value="instance" /></td></tr>'
>>> f.fields['name'].label = "Username"
>>> f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="instance" /></td></tr>'
```
注意不要改变base_fields 属性，因为一旦修改将影响同一个Python 进程中接下来所有的ContactForm 实例：
```python
>>> f.base_fields['name'].label = "Username"
>>> another_f = CommentForm(auto_id=False)
>>> another_f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="class" /></td></tr>'
```
> cleaned_data 始终只 包含表单中定义的字段，即使你在构建表单 时传递了额外的数据。
cleaned_data 始终只 包含表单中定义的字段，即使你在构建表单 时传递了额外的数据。
当表单合法时，cleaned_data 将包含所有字段的键和值，即使传递的数据不包含某些可选字段的值。

### `Form.cleaned_data`

### `Form.as_p`
`Form.as_ul`
`Form.as_table`

### `Form.error_css_class` `Form.required_css_class`

在`Form`类下可以用上面两个属性定义错误样式和必填样式，没有默认值，`required_css_class`也会回在`label`标签上

## `Form.auto_id`
控制表单上的`label`和表单元素的id，值为`True`，`False`或字符串，支持`%s`占位符，表示当前字段名
> 如果auto_id 设置为任何其它的真值 —— 例如不包含%s 的字符串 —— 那么其行为将类似auto_id 等于True。
默认情况下，auto_id 设置为'id_%s'。

### `Form.label_suffix`
默认为英文的`:`

### `BoundField`
```python
form = ContactForm()
for boundfield in form:
    print(boundfield)
# 或
from['name']
```
`BoundField.errors`
`BoundField.label_tag(contents=None, attrs=None, label_suffix=None)`
`BoundField.css_classes()`
`BoundField.value()`
提供初始值，会被绑定值覆盖
`BoundField.id_for_label`

### `Form.is_multipart()`
可判断表单是否需要`multipart`
```django
{% if form.is_multipart %}
    <form enctype="multipart/form-data" method="post" action="/foo/">
{% else %}
    <form method="post" action="/foo/">
{% endif %}
{{ form }}
</form>
```

子类化表单时可通过设置`None`来删除从父类中继承过来的字段
```python
>>> from django import forms
 
>>> class ParentForm(forms.Form):
...     name = forms.CharField()
...     age = forms.IntegerField()
 
>>> class ChildForm(ParentForm):
...     name = None
 
>>> ChildForm().fields.keys()
... ['age']
```

### `Form.prefix`
如果在页面中需要放多个相同的表单，可以设置表单的前缀
```python
>>> father = PersonForm()
>>> print(father.as_ul())
<li><label for="id_first_name">First name:</label> <input type="text" name="first_name" id="id_first_name" /></li>
<li><label for="id_last_name">Last name:</label> <input type="text" name="last_name" id="id_last_name" /></li>
>>> mother = PersonForm(prefix="mother")
>>> print(mother.as_ul())
<li><label for="id_mother-first_name">First name:</label> <input type="text" name="mother-first_name" id="id_mother-first_name" /></li>
<li><label for="id_mother-last_name">Last name:</label> <input type="text" name="mother-last_name" id="id_mother-last_name" /></li>
```

## [表单字段](http://python.usyiyi.cn/django/ref/forms/fields.html)

### `Field.has_change()`
检查字段的值是否从初始值发生改变

### 内建字段
#### `BooleanField`
Widget：`CheckboxInput`
错误信息的键：`required`

#### `CharField`
Widget：`TextInput`
错误信息的键：`required``max_length``min_length`
接收两个可选参数
`max_length``min_length`

#### `ChoiceField`
Widtget：`Select`
错误信息的键：`required``invalid_choice`
`invalid_choice`错误消息可能包含`%(value)s`，它将被选择的选项替换掉。
接收一个额外的必选参数
`choices`
是一个二元组组成的可迭代对象

#### `TypeChoiceField`
Widget：`Select`
错误信息的键：`required``invalid_choice`
接收额外的参数
`choices`
是一个二元组组成的可迭代对象
`coerce`
    接收一个参数并返回强制转换后的值的一个函数。例如内建的int、float、bool 和其它类型。默认为id 函数。注意强制转换在输入验证结束后发生，所以它可能强制转换不在 choices 中的值
`empty_value`
    用于表示“空”的值。默认为空字符串；None 是另外一个常见的选项。注意这个值不会被coerce 参数中指定的函数强制转换，所以请根据情况进行选择

#### `DateField`
Widget：`DateInput`
错误信息的键：`required``invalid`
接收一个可选参数
`input_formats`
一个格式的列表，用于转换一个字符串为`datateim.date`对象
默认为
```
['%Y-%m-%d',      # '2006-10-25'
'%m/%d/%Y',       # '10/25/2006'
'%m/%d/%y']       # '10/25/06'
```
另外，如果你在设置中指定USE_L10N=False，以下的格式也将包含在默认的输入格式中：
```
['%b %d %Y',      # 'Oct 25 2006'
'%b %d, %Y',      # 'Oct 25, 2006'
'%d %b %Y',       # '25 Oct 2006'
'%d %b, %Y',      # '25 Oct, 2006'
'%B %d %Y',       # 'October 25 2006'
'%B %d, %Y',      # 'October 25, 2006'
'%d %B %Y',       # '25 October 2006'
'%d %B, %Y']      # '25 October, 2006'
```

#### `DateTimeField`
Widget：`DateTimeInput`
错误信息的键：`required``invalid`
接收一个可选参数
`input_formats`
一个格式的列表，用于转换一个字符串为`datetime.datetime`对象
默认为
```
['%Y-%m-%d %H:%M:%S',    # '2006-10-25 14:30:59'
'%Y-%m-%d %H:%M',        # '2006-10-25 14:30'
'%Y-%m-%d',              # '2006-10-25'
'%m/%d/%Y %H:%M:%S',     # '10/25/2006 14:30:59'
'%m/%d/%Y %H:%M',        # '10/25/2006 14:30'
'%m/%d/%Y',              # '10/25/2006'
'%m/%d/%y %H:%M:%S',     # '10/25/06 14:30:59'
'%m/%d/%y %H:%M',        # '10/25/06 14:30'
'%m/%d/%y']              # '10/25/06'
```

#### `DecimalField`
Widget：当`Field.localize`是`False`时为NumberInput，否则为`TextInput`
错误信息的键：`required``invalid``max_value``min_digits``max_decimal_places``max_whole_digits`
`max_value`和`min_value`错误信息可能包含`%(limit_value)s`，它们将被真正的限制值替换。类似地，`max_digits`、`max_decimal_places`和 `max_whole_digits`错误消息可能包含`%(max)s`
接收四个可选参数
`max_value`
`min_value`
`max_digits`最大位数
`decimal_places`最大小数位

#### `DurationField`
Widget：`TextInput`
错误信息的键：`required``invalid`

#### `EmailField`
Widget：`EmailInput`
错误信息的键：`required``invalid`
接收两个可选参数
`max_length``min_length`

#### `FileField`
Widget：`ClearableFileInput`
错误信息的键：`required``invalid``missing``empty``max_length`
接收两个可选参数
`max_length``allow_empty_file`如果提供，这两个参数确保文件名的最大长度，而且即使文件内容为空时验证也会成功
`max_length`错误信息表示文件名的长度。在错误信息中，`%(max)d`将替换为文件的最大长度，%`(length)d` 将替换为当前文件名的长度

#### `FilePathField`
Widget：`Select`
错误信息的键：`required``invalid_choice`
这个字段允许从一个特定的目录选择文件
接收五个参数
`path`
必须
想要列出的目录的绝对路径
`recursive`
可选
布尔值，默认为`False`，是否需要递归这个目录
`match`
可选
正则表达式表示一个模式，只有匹配这个表达式的名称才会允许作为选项
`allow_files`
可选
布尔值，默认为`True`，表示是否应该包含指定位置的文件，它和`allow_folders`必须有一个为`True`
`allow_folders`
可选
布尔值，默认为`True`，表示是否应该包含指定位置的目录，和`allow_files`必须有一个为`True`

#### `FloatField`
Widget：当`Field.localize`是False 时为`NumberInput`，否则为`TextInput`
错误信息的键：`required``invalid``max_value``min_value`
接收两个可选参数
`max_value``min_value`

#### `ImageField`
Widget：`ClearableFileInput`
错误信息的键：`required``invalid``missing``empty``invalid_image`

#### `IntegerField`
Widget：当`Field.localize`是`False`时为`NumberInput`，否则为`TextInput`
错误信息的键：`required``invalid``max_value``min_value`
接收两个可选参数
`max_value``min_value`

#### `IPAddressField`
1.7弃用

#### `GenericIPAddressField`
Widget：`TextInput`
错误信息的键：`required``invalid`
接收两个可选参数
`protocol``unpack_ipv4`

#### `MultipleChoiceField`
Widget：`SelectMultiple`
错误信息的键：`required``invalid_choice``invalid_list`

#### `TypedMultipleChoiceField`
#### `NullBooleanField`
#### `RegexField`
#### `SlugField`
#### `TimeField`
#### `URLField`
#### `UUIDField`
输出时需要`.hex`
#### `ComboField`
```python
>>> from django.forms import ComboField
>>> f = ComboField(fields=[CharField(max_length=20), EmailField()])
>>> f.clean('test@example.com')
'test@example.com'
>>> f.clean('longemailaddress@example.com')
Traceback (most recent call last):
...
ValidationError: ['Ensure this value has at most 20 characters (it has 28).']
```
#### `MultiValueField`
#### `SplitDateTimeField`
#### `ModelChoiceField`
```python
# A custom empty label
field1 = forms.ModelChoiceField(queryset=..., empty_label="(Nothing)")
 
# No empty label
field2 = forms.ModelChoiceField(queryset=..., empty_label=None)
```

#### `ModelMultipleChoiceField`

## [Widgets](http://python.usyiyi.cn/django/ref/forms/widgets.html)

处理文本输入的Widget
- `TextInput`
- `NumberInput`
- `EmailInput`
- `URLInput`
- `PasswordInput`
- `HiddenInput`
- `DateInput`
- `DateTimeInput`
- `TimeInput`
- `Textarea`

选择和复选框Widget
-  `CheckboxInput`
-  `Select`
-  `NullBooleanSelect`
-  `SelectMultiple`
-  `RadioSelect`
-  `CheckboxSelectMultiple`

文件上传`Widget`
- `FileInput`
- `ClearableFileInput`

复合Widget
- `MultipleHiddenInput`
- `SplitDateTimeWidget`
- `SplitHiddenDateTimeWidget`
- `SelectDateWidget`


## [从模型创建表单](http://python.usyiyi.cn/django/topics/forms/modelforms.html)
下面两种方法效果相同
```python
author = Author(title='Mr')
form = PartialAuthorForm(request.POST, instance=author)
form.save()
# or
form = PartialAuthorForm(request.POST)
author = form.save(commit=False)
author.title = 'Mr'
author.save()
```

显式定义的字段不会从对于的模型中获取属性，例如 max_length 或required。 如果你希望保持模型中指定的行为，你必须设置在声明表单字段时显式设置相关的参数。

例如，如果Article 模型像下面这样：
```python
class Article(models.Model):
    headline = models.CharField(max_length=200, null=True, blank=True,
                                help_text="Use puns liberally")
    content = models.TextField()
```
而你想为headline 做一些自定义的验证，在保持blank 和help_text 值的同时，你必须定义这样定义ArticleForm：

```python
class ArticleForm(ModelForm):
    headline = MyFormField(max_length=200, required=False,
                           help_text="Use puns liberally")
 
    class Meta:
        model = Article
        fields = ['headline', 'content']
```

创建简单的表单或表单集可以使用`modelform_factory()``modelformset_factory()`方法来新建。

启用字段的本地化功能¶

默认情况下，ModelForm 中的字段不会本地化它们的数据。你可以使用Meta 类的localized_fields 属性来启用字段的本地化功能。
```python
>>> from django.forms import ModelForm
>>> from myapp.models import Author
>>> class AuthorForm(ModelForm):
...     class Meta:
...         model = Author
...         localized_fields = ('birth_date',)
```
如果localized_fields 设置为`'__all__' `这个特殊的值，所有的字段都将本地化。

提供的初始值会覆盖从实例取得的值
```python
>>> article = Article.objects.get(pk=1)
>>> article.headline
'My headline'
>>> form = ArticleForm(initial={'headline': 'Initial headline'}, instance=article)
>>> form['headline'].value()
'Initial headline'
```

如果不需要很多自定义，可以直接使用工厂方法来生成表单类
```python
>>> from django.forms.models import modelform_factory
>>> from myapp.models import Book
>>> BookForm = modelform_factory(Book, fields=("author", "title"))
>
```
```python
>>> from django.forms import Textarea
>>> Form = modelform_factory(Book, form=BookForm,
...                          widgets={"title": Textarea()})
```
```python
>>> Form = modelform_factory(Author, form=AuthorForm, localized_fields=("birth_date",))
```
表单集
```python
>>> from django.forms.models import modelformset_factory
>>> from myapp.models import Author
>>> AuthorFormSet = modelformset_factory(Author, fields=('name', 'title'))
```
使用`model`生成的`formset `默认带一个包含全部对象的`queryset`
`formset``save()`之后，会有新的属性
```python
models.BaseModelFormSet.changed_objects
models.BaseModelFormSet.deleted_objects
models.BaseModelFormSet.new_objects
```
`max_num`为最大的表单数，如果初始`queryset`长度比`max_num`，则按照`queryset`来，`extra`是可以额外添加的空表单的个数，但`extra`和`queryset`长度相加如果大于`max_num`，则`extra`和实例设置可能表现不一样，如`queryset`长度为2，`max_num`为4，`extra`不管是2还是5，最终表现出来都是2。
```python
AuthorFormSet = modelformset_factory(Author, fields=('name',), max_num=4, extra=2)
```
`max_num`默认只影响显示，不影响验证，如果需要影响验证添加`validate_max=True`即可

## [表单素材 ( Media 类)](http://python.usyiyi.cn/django/topics/forms/media.html)

**`Form`和`Widget`都可以定义素材**

```python
from django import forms
 
class CalendarWidget(forms.TextInput):
    class Media:
        css = {
            'all': ('pretty.css',)
        }
        js = ('animations.js', 'actions.js')
```
使用`CalendarWidget`会自动引入下列资源
```html
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
```
`Widget`会默认继承父类的素材，如果不想继承在`Media`里使用`extend`禁止。

动态定义
```python
class CalendarWidget(forms.TextInput):
    def _media(self):
        return forms.Media(css={'all': ('pretty.css',)},
                           js=('animations.js', 'actions.js'))
    media = property(_media)
```
两个`Media`实例可以相加
```python
>>> from django import forms
>>> class CalendarWidget(forms.TextInput):
...     class Media:
...         css = {
...             'all': ('pretty.css',)
...         }
...         js = ('animations.js', 'actions.js')
 
>>> class OtherWidget(forms.TextInput):
...     class Media:
...         js = ('whizbang.js',)
 
>>> w1 = CalendarWidget()
>>> w2 = OtherWidget()
>>> print(w1.media + w2.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>
```

表单`Media`
```python
>>> class ContactForm(forms.Form):
...     date = DateField(widget=CalendarWidget)
...     name = CharField(max_length=40, widget=OtherWidget)
...
...     class Media:
...         css = {
...             'all': ('layout.css',)
...         }
 
>>> f = ContactForm()
>>> f.media
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<link href="http://static.example.com/layout.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script
```

## [表单集](http://python.usyiyi.cn/django/topics/forms/formsets.html)

表单集控制
`max_num`
`min_num`
`validate_max`
`validate_min`
`can_order`
`can_delete`

其中`can_order``can_delete`默认以以下形式展现



如果是使用`Model`生成的表单集，如果`delete`后，在调用`formset.save()`会自动删除相应的数据，但如果调用了`formset.save(commit=False)`，则需要手动删除（1.6或更早版还是会自动删除）
```python
>>> instances = formset.save(commit=False)
>>> for obj in formset.deleted_objects:
...     obj.delete()
```
如果要兼容1.6或更早版，可以这么写
```python
>>> try:
>>>     # For Django 1.7+
>>>     for obj in formset.deleted_objects:
>>>         obj.delete()
>>> except AssertionError:
>>>     # Django 1.6 and earlier already deletes the objects, trying to
>>>     # delete them a second time raises an AssertionError.
>>>     pass
```

在表单集里添加额外的字段
```python
>>> from django.forms.formsets import BaseFormSet
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> class BaseArticleFormSet(BaseFormSet):
...     def add_fields(self, form, index):
...         super(BaseArticleFormSet, self).add_fields(form, index)
...         form.fields["my_field"] = forms.CharField()
 
>>> ArticleFormSet = formset_factory(ArticleForm, formset=BaseArticleFormSet)
>>> formset = ArticleFormSet()
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" id="id_form-0-pub_date" /></td></tr>
<tr><th><label for="id_form-0-my_field">My field:</label></th><td><input type="text" name="form-0-my_field" id="id_form-0-my_field" /></td></tr>
```

表单集对应的模版写法
```django
<form method="post" action="">
    <table>
        {{ formset }}
    </table>
</form>
```
也可以手动渲染，不能缺少`{{ formset.management_form }}`
```django
<form method="post" action="">
    {{ formset.management_form }}
    <table>
        {% for form in formset
        {{ form }}
        {% endfor
    </table>
</form>
```
如果是手动渲染，`can_order``can_delete`需要手动添加
```django
<form method="post" action="">
    {{ formset.management_form }}
    {% for form in formset
        <ul>
            <li>{{ form.title }}</li>
            <li>{{ form.pub_date }}</li>
            {% if formset.can_delete
                <li>{{ form.DELETE }}</li>
            {% endif
        </ul>
    {% endfor
</form>
```

在一个视图中使用多个`FormSet`需要回前缀`prefix`
```python
from django.forms.formsets import formset_factory
from django.shortcuts import render_to_response
from myapp.forms import ArticleForm, BookForm
 
def manage_articles(request):
    ArticleFormSet = formset_factory(ArticleForm)
    BookFormSet = formset_factory(BookForm)
    if request.method == 'POST':
        article_formset = ArticleFormSet(request.POST, request.FILES, prefix='articles')
        book_formset = BookFormSet(request.POST, request.FILES, prefix='books')
        if article_formset.is_valid() and book_formset.is_valid():
            # do something with the cleaned_data on the formsets.
            pass
    else:
        article_formset = ArticleFormSet(prefix='articles')
        book_formset = BookFormSet(prefix='books')
    return render_to_response('manage_articles.html', {
        'article_formset': article_formset,
        'book_formset': book_formset,
    })
```

## [表单验证和字段验证](http://python.usyiyi.cn/django/ref/forms/validation.html)
格式标准
抛出单个错误
```python
raise ValidationError(
    _('Invalid value: %(value)s'),
    code='invalid',
    params={'value': '42'},
)
```
招聘多个错误
```python
# Good
raise ValidationError([
    ValidationError(_('Error 1'), code='error1'),
    ValidationError(_('Error 2'), code='error2'),
])
 
# Bad
raise ValidationError([
    _('Error 1'),
    _('Error 2'),
])
```
创建一个新的表单字段添加默认验证
```python
from django import forms
from django.core.validators import validate_email
 
class MultiEmailField(forms.Field):
    def to_python(self, value):
        "Normalize data to a list of strings."
 
        # Return an empty list if no input was given.
        if not value:
            return []
        return value.split(',')
 
    def validate(self, value):
        "Check if value consists only of valid emails."
 
        # Use the parent's handling of required fields, etc.
        super(MultiEmailField, self).validate(value)
 
        for email in value:
            validate_email(email)
```

## [Django 的设置](http://python.usyiyi.cn/django/topics/settings.html)
django-admin 工具

当使用django-admin 时， 你可以设置只设置环境变量一次，或者每次运行该工具时显式传递设置模块。

例如（Unix Bash shell）：
```
export DJANGO_SETTINGS_MODULE=mysite.settings
django-admin runserver
```
例如（Windows shell）：
```
set DJANGO_SETTINGS_MODULE=mysite.settings
django-admin runserver
```
使用--settings 命令行参数可以手工指定设置：
```
django-admin runserver --settings=mysite.settings
```

使用下面的命令可以查询设置与默认设置的不同
```python
python manage.py diffsettings
```
在django app中使用设置应使用以下导入方式
```python
from django.conf import settings
```
注意，django.conf.settings 不是一个模块 —— 它是一个对象。所以不可以导入每个单独的设置：
```python
from django.conf.settings import DEBUG  # This won't work.
```
不要在应用运行时改变设置

## [完整列表设置(Settings)](http://python.usyiyi.cn/django/ref/settings.html)

`CSRF_COOKIE_SECURE=True`只通过`HTTPS`传递`cookie`

`DATABASES['CONN_MAX_AGE']`数据库连接的戚时间，默认为0（历史遗留行为），设置为`None`表示无限的持久连接
`DECIMAL_SEPARATOR`类型数据的分隔符默认为点`.`
`DISALLOWED_USER_AGENTS`编写正则表达式元组禁用代码访问，需要启用`CommonMiddleware`中间件
`INTERNAL_IPS`设置公司内容的ip，在些ip列表中的ip可以访问admindoc下的书签

## [应用](http://python.usyiyi.cn/django_182/ref/applications.html)

```python
# rock_n_roll/apps.py
 
from django.apps import AppConfig
 
class RockNRollConfig(AppConfig):
    name = 'rock_n_roll'
    verbose_name = "Rock ’n’ roll"
```

```python
# rock_n_roll/__init__.py
 
default_app_config = 'rock_n_roll.apps.RockNRollConfig'
```

`AppConfig`可配置的属性
- `AppConfig.name`
- `AppConfig.label`
- `AppConfig.verbose_name`
- `Appconfig.path`

`AppConfig`只读属性
- `AppConfig.module`
- `Appconfig.models_module`

`AppConfig`方法
- `AppConfig.get_models()`
- `AppConfig..get_model(model_name)`
- `AppConfigevaluate.ready()`

```python
>>> from django.apps import apps
>>> apps.get_app_config('admin').verbose_name
'Admin'
```

`APP`
- `apps.ready`
- `apps.get_app_configs()`
- `apps.get_app_config(app_label)`
- `apps.is_installed(app_name)`
- `apps.get_model(app_label, model_name)`


## [Django异常](python.usyiyi.cn/django_182/ref/exceptions.html)

### 核心异常
*`django.core.exceptions`*

- `ObjectDoesNotExist`
对象不存在
`DoesNotExist`的基类
对ObjectDoesNotExist的try/except会为所有模型捕获到所有DoesNotExist 异常
```python
from django.core.exceptions import ObjectDoesNotExist
try:
    e = Entry.objects.get(id=3)
    b = Blog.objects.get(id=1)
except ObjectDoesNotExist:
    print("Either the entry or blog doesn't exist.")
```

- `FieldDoesNotExist`
当被请求的字段在模型或模型的父类中不存在时，`FieldDoesNotExist`异常由模型的 `_meta.get_field()`方法抛出

- `MultipleObjectsReturned`
查询时，预期只有一个对象，但是返回了多个对象会抛出此异常

- `SuspiciousOperation`
当用户进行的操作在安全方面可疑的时候，抛出此异常，例如，篡改`cookie`
子类
  - `DisallowedHost`
  - `DisallowedModelAdminLookup`
  - `DisallowedModelAdminToField`
  - `DisallowedRedirect`
  - `InvalidSessionKey`
  - `SuspiciousFileOperation`
  - `SuspiciousMultipartForm`
  - `SuspiciousSession`

- `PermissionDenied`
当用户不被允许来执行请求的操作时产生

- `ViewDoesNotExist`
当请求的视图不存在时抛出此异常

- `MiddlewareNotUsed`
当中间件没有在服务器配置中出现时，抛出此异常

- `ImproperlyConfigured`
django配置不当时抛出此异常，如`settings.py`中的值不正确或者不可解析

- `FieldError`
当模型上的字段出现问题时，抛出此异常，由以下原因造成：
  - 模型中的字段与抽象基类中的字段重名
  - 排序造成了一个死循环
  - 关键词不能由过滤器参数解析
  - 字段不能由查询参数中的关键词决定
  - 连接（join）不能在指定对象上使用
  - 字段名称不可用
  - 查询包含了无效的`order_by`参数

  - `ValidationError`
  当表单或模型字段验证失败时抛出此异常

- `NON_FIELD_ERRORS`
在表单或者模型中不属于特定字段的`ValidationError`被归类为`NON_FIELD_ERRORS`

### URL解析器异常

- `Resolver404`
`django.http.Http404`的子类
当向`resolve`传递的路径不能匹配到对应视图时抛出此异常

- `NoReverseMatch`
当你的URLconf中的一个匹配的URL不能基于提供的参数识别时，抛出此异常

### 数据库异常
数据库异常由django.db导入
- `Error`
- `InterfaceError`
- `DatabaseError`
- `DataError`
- `OperationalError`
- `IntegrityError`
- `InternalError`
- `ProgrammingError`
- `NotSupportedError`

### HTTP异常
HTTP异常由django.http导入

- `UnreadablePostError`
用户取消上传时抛出此异常

### 事务异常
事务异常定义由`django.db.transaction`导入

### 测试框架异常
由DJango django.test 包提供的异常

- `RedirectCycleError`
当测试客户端检测到重定向的循环或者过长的链时抛出此异常

### `Python`异常
Django在适当的时候也会抛出Python的内建异常

## [django-admin and manage.py](http://python.usyiyi.cn/django_182/ref/django-admin.html)

- `dumpdata`
该命令将所有与被命名应用相关联的数据库中的数据输出到标准输出。
如果在dumpdate命令后面未指定Django应用名，则Django项目中安装的所有应用的数据都将被dump到fixture中
`dumpdata --output data.json`

- `flus`
清空数据库，重新装载初始数据
- `--noinput`
- `--database`
- `--no-initial-data`


- `inspectdb`
根据数据库结构生成model
```python
python manage.py inspectdb > models.py
```

- `loaddata`
导入fixture数据

- `runserver`
启动本地上一个轻量级的Web服务器，默认多线程
`--noreload`禁用自动重新载入
`--nothreading`禁用多线程

```python
runserver 0.0.0.0:80
```

## [ 添加自定义的命令](http://python.usyiyi.cn/django_182/howto/custom-management-commands.html)

向应用下添加management/commands目录，Django会为此目录下的所有没有带下划线开头的python模块都注册一个`manage.py`命令。
在Python 2上，请确保management和management/commands两个目录都包含`__init__.py` 文件。

```python
from django.core.management.base import BaseCommand, CommandError
from polls.models import Poll
 
class Command(BaseCommand):
    help = 'Closes the specified poll for voting'
 
    def add_arguments(self, parser):
        # 命令行接收一个或多个poll_id值
        #
        parser.add_argument('poll_id', nargs='+', type=int)
 
    def handle(self, *args, **options):
        for poll_id in options['poll_id']:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                raise CommandError('Poll "%s" does not exist' % poll_id)
 
            poll.opened = False
            poll.save()
 
            self.stdout.write('Successfully closed poll "%s"' % poll_id)
```
![django-commands](/media/django-commands.png)


