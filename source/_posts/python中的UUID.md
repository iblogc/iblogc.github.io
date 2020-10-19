---
title: python中的UUID
date: 2015-06-02 23:32:59
categories: [程序猿]
tags: [Python, UUID]
---
## 简介（维基百科）
> 通用唯一识别码（英语：Universally Unique Identifier，简称UUID）是一种软件建构的标准，亦为开放软件基金会组织在分散式计算环境领域的一部份。  
UUID的目的，是让分散式系统中的所有元素，都能有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。如此一来，每个人都可以建立不与其它人冲突的UUID。在这样的情况下，就不需考虑资料库建立时的名称重复问题。目前最广泛应用的UUID，是微软公司的全局唯一标识符（GUID），而其他重要的应用，则有Linux ext2/ext3档案系统、LUKS加密分区、GNOME、KDE、Mac OS X等等。另外我们也可以在e2fsprogs套件中的UUID函式库找到实现。[[3]][1]

<!--more-->

## 定义 :
UUID是由一组32位数的16进位数字所构成，是故UUID理论上的总数为1632=2128，约等于3.4 x 1038。也就是说若每纳秒产生1兆个UUID，要花100亿年才会将所有UUID用完，，它保证对在同一时空中的所有机器都是唯一的（重复机率请参考[随机UUID的重复机率](http://zh.wikipedia.org/zh-hans/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81)）。

## 算法
1. uuid1()——基于时间戳
由MAC地址、当前时间戳、随机数生成。可以保证全球范围内的唯一性，
但MAC的使用同时带来安全性问题，局域网中可以使用IP来代替MAC。
2. uuid2()——基于分布式计算环境DCE（Python中没有这个函数）
算法与uuid1相同，不同的是把时间戳的前4位置换为POSIX的UID。
实际中很少用到该方法。
3. uuid3()——基于名字的MD5散列值
通过计算名字和命名空间的MD5散列值得到，保证了同一命名空间中不同名字的唯一性，
和不同命名空间的唯一性，但同一命名空间的同一名字生成相同的uuid。
4. uuid4()——基于随机数
由伪随机数得到，有一定的重复概率，该概率可以计算出来。
5. uuid5()——基于名字的SHA-1散列值
算法与uuid3相同，不同的是使用 Secure Hash Algorithm 1 算法

## 在`python`中在生成UUID
`import uuid`后即可使用
示例代码
```python
import uuid
uuid.uuid1()
uuid.uuid3(namespace, name)
uuid.uuid4()
uuid.uuid5(namespace, name)
```

## 参考
> http://zh.wikipedia.org/zh-hans/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81
> https://docs.python.org/2/library/uuid.html

[1]: http://zh.wikipedia.org/zh-hans/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81 "通用唯一识别码"