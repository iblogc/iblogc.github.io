title: Django内置的flatpages应用
date: 2015-09-08 21:17:55
updated: 2015-09-08 21:17:55
categories: [程序猿]
tags: [Python, Django]
excerpt: true
---
不知道大家平时写网站时类似「关于页面」，「用户协议」这类页面是如何处理的。这里列出了这类页面的两个特点：
1. 页面数据简单（不会有很多动态数据展示）;
2. 有更新但频率不高;

如果要为这类页面展示建立多个数据表，显然是很浪费的行为，但如果直接写成静态页面文件，更新又比较很麻烦，这时候就可以使用`flatpages `来解决这类问题了。
<!--more-->

`django.contrib.flatpages`是`Django `的内置app，用于添加更新的一些简单的页面，具体设置，请继续查看以下步骤。

## 安装
首先确保`INSTALLED_APPS`中已经存在`django.contrib.sites`，因为`django.contrib.flatpages`依赖于此包。
`settings.py`
```python
INSTALLED_APPS = (
    # ...
    'django.contrib.sites',
    'django.contrib.flatpages',
)
# ...
# 如果没有设置`SITE_ID`值，则需要设置，这里直接设置为1
SITE_ID = 1
```
执行`python manage.py migrate`建表

## 配置
路由配置可先以有多种形式
`urls.py`
第一种（需放在最后，推荐）
```
from django.contrib.flatpages import views
urlpatterns += [
    url(r'^(?P<url>.*/)$', views.flatpage),
]
```
第二种（每个页面都需要写一个url，推荐）
```
from django.contrib.flatpages import views
urlpatterns = [
    url(r'^about-us/$', views.flatpage, {'url': '/about-us/'}, name='about'),
    url(r'^license/$', views.flatpage, {'url': '/license/'}, name='license'),
]
```
或者，如果你不想配置路由，还有一种更简单的方法，直接在`settings.py`的里添加中间件
```
MIDDLEWARE_CLASSES = (
    # ...
    'django.contrib.flatpages.middleware.FlatpageFallbackMiddleware',
)
```
*为确保配置生效，保险的方法是把`django.contrib.flatpages.middleware.FlatpageFallbackMiddleware`放在最后一行*

## 管理`flatpages`

默认的你可以登录超级管理员后台（如果开启），找到`Flat pages`，进去点击添加，可以看到可配置的选项有，`URL` `Title` `Content` `Site` `Enable comments` `Registration required` `Template name`

### 数据项说明
- `URL`:  页面所处的 URL，不包括域名，但是包含前导斜杠 (例如 /about/contact/ )
- `Title`: 页面的标题，框架不对它作任何特殊处理。由你通过模板来显示它
- `Content`: 页面的内容 (即 HTML 页面)，框架不会对它作任何特别处理。由你负责使用模板来显示
- `Site`: 页面放置的站点，该项设置集成了 Django 多站点框架
- `Enable comments`: 是否允许该简单页面使用评论，框架不对此做任何特别处理。你可在模板中检查该值并根据需要显示评论窗体
- `Registration required`: 是否注册用户才能查看此简单页面，该设置项集成了 Djangos 验证/用户框架，该框架于第十二章详述。
- `Template name`: 用来解析该简单页面的模板名称，这是一个可选项，如果未指定模板或该模板不存在，系统会退而使用默认模板 `flatpages/default.html`（我在`Django1.8.4`里死活没找到，只好自己写好一个扔进去）

当添加相应的数据后，剩下工作就交给`flatpages`吧，如果你是使用中间件形式的，则`flatpages `会在配置完所有`urls.py`后，没有找到配置到对应的`URL`，才会到`flatpages `中查找，如果还是找不到，则会引发`Http404`异常，即`FlatpageFallbackMiddleware `只在`404`时会被激活，而不会在`500`或其它错误响应时被激活。

如果你需要自己定制，则可以针对`django/contrib/flatpages/models.py`自己写增删改方法就可以。
`models.py`
```python
class FlatPage(models.Model):
    url = models.CharField(_('URL'), max_length=100, db_index=True)
    title = models.CharField(_('title'), max_length=200)
    content = models.TextField(_('content'), blank=True)
    enable_comments = models.BooleanField(_('enable comments'), default=False)
    template_name = models.CharField(_('template name'), max_length=70, blank=True,
        help_text=_(
            "Example: 'flatpages/contact_page.html'. If this isn't provided, "
            "the system will use 'flatpages/default.html'."
        ),
    )
    registration_required = models.BooleanField(_('registration required'),
        help_text=_("If this is checked, only logged-in users will be able to view the page."),
        default=False)
    sites = models.ManyToManyField(Site)
 
    class Meta:
        db_table = 'django_flatpage'
        verbose_name = _('flat page')
        verbose_name_plural = _('flat pages')
        ordering = ('url',)
 
    def __str__(self):
        return "%s -- %s" % (self.url, self.title)
 
    def get_absolute_url(self):
        # Handle script prefix manually because we bypass reverse()
        return iri_to_uri(get_script_prefix().rstrip('/') + self.url)
```

## 模板
默认模板路径为`flatpages/default.html`
```html
<!DOCTYPE html>
<html>
<head>
<title>{{ flatpage.title }}</title>
</head>
<body>
{{ flatpage.content }}
</body>
</html>
```
> 在实际应用中，我们不太可能会使用默认的模板，你可能需要自己写一个漂亮模板，比如有一个头部和底部，头部可能还需要添加`requeset.user`显示用户信息等。

## 高级应用

获取`flatpages`实例列表
```
{% load flatpages %}
{% get_flatpages as flatpages %}
```

获取当前用户能打开的`flatpages`实例列表
```
{% load flatpages %}
{% get_flatpages for request.user as about_pages %}
```

获取链接以`/about/`为开头的`flatpages`实例列表
```
{% load flatpages %}
{% get_flatpages '/about/' as about_pages %}
```

上面两种也可以组合使用
```
{% load flatpages %}
{% get_flatpages '/about/' for someuser as about_pages %}
```

## 生成`sitemaps.xml`
```
from django.conf.urls import url
from django.contrib.flatpages.sitemaps import FlatPageSitemap
from django.contrib.sitemaps.views import sitemap
 
urlpatterns = [
    # ...
 
    # the sitemap
    url(r'^sitemap\.xml$', sitemap,
        {'sitemaps': {'flatpages': FlatPageSitemap}},
        name='django.contrib.sitemaps.views.sitemap'),
]
```

## 容易踩的坑
最好把`settings.py`里的`APPEND_SLASH`设置为`Ture`， 这样不管是`/about-us`还是`/about-us/`都可以访问到。

## 参考资料
- https://docs.djangoproject.com/en/1.8/ref/contrib/flatpages/
- http://djangobook.py3k.cn/2.0/chapter16/
