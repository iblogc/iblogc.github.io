---
title: Django REST framework单元测试「Unit Testing」
comments: true
tags: [restful api, 接口, 单元测试, 测试, Django]
categories: [程序猿]
date: 2017-09-05 23:23:41
---
<br />
<!--more-->
## `settings`
`Django`运行单元测试时，会以`settings`里的数据库配置里的`NAME`新建一个以`test_`开关的临时数据库，并在测试结束后删除，默认的测试数据库会以当前的`migrations`文件来创建数据表并进行迁移，但如果`migrations`文件很多，每次运行时间将很久，所以可以跳过迁移，直接以当前`Model`结果来创建表以提升测试效率，如果想进一步加快测试时创建数据库的速度，可以使用`SQLite`数据库引擎，当使用`SQLite`数据库引擎时，测试将默认使用内存数据库。
```python
TESTING = len(sys.argv) > 1 and sys.argv[1] == 'test'
if TESTING:
    # 当使用SQLite数据库引擎时，测试将默认使用内存数据库
    DATABASES['default'] = {
        'ENGINE': 'django.db.backends.sqlite3',
    }
    # 单元测试时, 跳过migrate, 极 的提升测试运 效率
    # 具体可以查看
    # https://simpleisbetterthancomplex.com/tips/2016/08/19/django-tip-12-disabl ing-migrations-to-speed-up-unit-tests.html
    # https://stackoverflow.com/questions/36487961/django-unit-testing-taking-a- very-long-time-to-create-test-database

    class DisableMigrations(object):
        def __contains__(self, item):
            return True
    
        def __getitem__(self, item):
            return "notmigrations"
    
    
    MIGRATION_MODULES = DisableMigrations()
```

## 示例代码
```python
# -*- coding: utf-8 -*-
from __future__ import absolute_import
from __future__ import unicode_literals

from rest_framework import status
from rest_framework.test import APITestCase

from apps.account.models import User
from apps.account.tests.test_utils import TestCaseUtils

__author__ = 'jeff'


class UserAPITests(APITestCase, TestCaseUtils):
    # 初始数据加载，可使用manage.py dumpdata [app_label app_label app_label.Model]生成
    # xml/yaml/json格式的数据
    # 一般放在每个应用的fixtures目录下, 只需要填写json文件名即可，django会自动查找
    # 此测试类运行结束后，会自动从数据库里销毁这份数据
    # fixtures = ['user.json']

    def setUp(self):
        # 在类里每个测试方法执行前会运行
        # 在此方法执行前，django会运行以下操作
        # 1. 重置数据库，数据库恢复到执行migrate后的状态
        # 2. 加载fixtures数据
        # 所以每个测试方法里对数据库的操作都是独立的，不会相互影响
        kwargs = dict(mobile_phone='15999999999', password='111111')
        self.user = User.app_user_objects.create(**kwargs)

    def tearDown(self):
        # 在类里每个方法结束执行后会运行
        pass

    @classmethod
    def setUpClass(cls):
        # 在类初始化时执行，必须调用super
        super(UserAPITests, cls).setUpClass()
        cls.token = ''

    @classmethod
    def tearDownClass(cls):
        # 在整个测试类运行结束时执行，必须调用super
        super(UserAPITests, cls).tearDownClass()

    def test_app_user_login_success(self):
        """APP用户登录接口成功情况"""
        # path使用硬编码，不要使用reverse反解析url，以便在修改url之后能及时发现接口地址变化，并通知接口使用人员
        path = '/api/api-token-auth/'
        data = {'mobile_phone': '15999999999', 'password': '111111'}
        response = self.client.post(path, data)
        # response.data是字典对象
        # response.content是json字符串对象
        self.assertEquals(response.status_code,
                          status.HTTP_200_OK,
                          '登录接口返回状态码错误: 错误信息: {}'.format(response.content))
        self.assertIn('token', response.data, '登录成功后无token返回')

    def test_app_user_login_with_error_pwd(self):
        path = '/api/api-token-auth/'
        data = {'mobile_phone': '15999999999', 'password': '123456'}
        response = self.client.post(path, data)
        self.assertEquals(response.status_code, status.HTTP_400_BAD_REQUEST)
        self.assertJSONEqual('{"errors":["用户名或密码错误。"]}', response.content)

    def test_get_app_user_profile_success(self):
        """成功获取app用户个人信息接口"""
        path = '/api/account/user/profile/'
        headers = self.get_headers(user=self.user)
        response = self.client.get(path, **headers)
        # 校验一些关键数据即可
        # 如果是创建新数据，不仅要校验返回的状态码和数据，
        # 还需要到使用Django ORM去数据库查询数据是否创建成功
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(6, len(response.data))
        self.assertIn('url', response.data)
        self.assertIn('mobile_phone', response.data)
        self.assertIn('avatar', response.data)
        self.assertIn('company_name', response.data)
        self.assertIn('username', response.data)
        self.assertIn('is_inviter', response.data)

    def test_get_app_user_profile_without_token(self):
        """不传token请求获取用户信息接口"""
        path = '/api/account/user/profile/'
        response = self.client.get(path)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

## 断言
```python
# 来自unittest.case.TestCase
assertFalse(expr, msg=None)
assertTrue(expr, msg=None)
assertEqual(first, second, msg=None)
assertNotEqual(first, second, msg=None)
assertAlmostEqual(first, second, places=None, msg=None, delta=None)
assertNotAlmostEqual(first, second, places=None, msg=None, delta=None)
assertSequenceEqual(seq1, seq2, msg=None, seq_type=None)
assertListEqual(list1, list2, msg=None)
assertTupleEqual(tuple1, tuple2, msg=None)
assertSetEqual(set1, set2, msg=None)
assertIn(member, container, msg=None)
assertNotIn(member, container, msg=None)
assertIs(expr1, expr2, msg=None)
assertIsNot(expr1, expr2, msg=None)
assertDictEqual(d1, d2, msg=None)
assertDictContainsSubset(expected, actual, msg=None)
assertItemsEqual(expected_seq, actual_seq, msg=None)
assertMultiLineEqual(first, second, msg=None)
assertLess(a, b, msg=None)
assertLessEqual(a, b, msg=None)
assertGreater(a, b, msg=None)
assertGreaterEqual(a, b, msg=None)
assertIsNone(obj, msg=None)
assertIsInstance(obj, cls, msg=None)
assertNotIsInstance(obj, cls, msg=None)
assertRaisesRegexp(expected_exception, expected_regexp,
                           callable_obj=None, *args, **kwargs)
assertRegexpMatches(text, expected_regexp, msg=None)
assertNotRegexpMatches(text, unexpected_regexp, msg=None)
```


## 测试接口地址
测试接口地址建议使用硬编码，不要使用`reverse`反解析url，原因是接口地址尽量避免改变，如果必须修改，需要以很明显的方式来提醒开发人员以便开发人员通知接口使用人员。

## 测试数据准备
有如下两种方法准备测试数据
1. 简单的数据可以在`setUp()`里来创建；
2. 复杂数据可以使用fixtures来写，并在赋值给测试类的`fixtures`属性；
fixtures数据示例
```json
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

## 测试覆盖率（coverage）
在`Pycharm`里可以通用右键项目，选择`Run 'Test:' with Coverage`来查看测试的覆盖率。也可以通过其它第三方包查看测试覆盖率，具体请自己查询。


