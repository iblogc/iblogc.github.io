title: Django字段选项related_name和related_query_name
comments: true
date: 2015-10-20 22:22:06
categories: [程序猿]
tags: [Python, Django]
---
`data`
```sqlite3
sqlite> select * from author;
id      name    age
1       jim     12
2       tom     11
sqlite> select * from book;
id      name    author_id
1       learn java      1
2       learn python    1
3       learn c++       2
```
<!--more-->
`models.py`
```python
# -*- coding: UTF-8 -*-
from __future__ import unicode_literals
from django.db import models

# Create your models here.

class Author(models.Model):
    name = models.CharField(verbose_name='姓名', max_length=50)
    age = models.IntegerField(verbose_name='年龄')

class Book(models.Model):
    name = models.CharField(verbose_name='书名', max_length=100)
    author = models.ForeignKey(Author, verbose_name='作者')
```
执行语句
```
>>> Author.objects.filter(book__name='learn java')
[<Author: jim>]
>>> author = Author.objects.get(pk=1)
>>> author.book_set.all()
[<Book: learn java>, <Book: learn python>]
```

假如把类`Book`改成这样
```
class Book(models.Model):
    name = models.CharField(verbose_name='书名', max_length=100)
    author = models.ForeignKey(Author, verbose_name='作者', related_name='bs', related_query_name='b')
```
那么上面查询代码就应该写成这样
```
>>> Author.objects.filter(b__name='learn java')
[<Author: jim>]
>>> author = Author.objects.get(pk=1)
>>> author.bs.all()
[<Book: learn java>, <Book: learn python>]
```
> 如果`book `表里有两个字段都外键关联`author `表，这时`related_name`就非常有用了。




