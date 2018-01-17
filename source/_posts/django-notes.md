---
title: Django 笔记
tags:
  - django
  - python
  - programming
date: 2018-01-12 14:59:48
---

Django是一个用python写的轻量级Web框架，虽说是轻量级，但是它能做的事情并不少，有了Django我们就可以用很少的代码来搭建一个现代化的网站，避免重复造轮子。Django2.0之后已经放弃了对python2 的支持，只支持python3.4 及以上版本。

<!-- more -->

# URL Route
## 路由规则
Django项目的路由规则写在一系列urls.py文件里，包括项目的urls.py和各个app的urls.py。当一个请求到来，在展示最终页面之前需要经过以下步骤：
- 首先确定根URLconf，通常这个值在settings.py 中设置。但是如果传入的HttpRequest对象有urlconf属性，那么它将会作为当前请求的根URLconf，代替settings中的ROOT_URLCONF；
- Django加载该URLconf 模块，找到urlpatterns变量；
- Django**依次**匹配每一个url pattern，在第一个匹配所请求URL的pattern处停下来，因此pattern 的顺序是有影响的；
- 一旦一个URL pattern匹配成功，Django 会调用对应的视图，视图接收以下参数：
    - 一个HTTPRequest 实例;
    - 如果匹配的URL pattern没有返回命名组，正则表达式匹配的内容将会作为positional parameters 提供给视图;
    - 关键字参数由正则表达式匹配的命名组组成，可以被可选参数覆盖；
- 如果没有匹配到任何URL pattern，或者在匹配过程中抛出了异常，Django会调用一个对应的错误处理视图，比如400，500等；

### 传递额外的参数给视图
URLconf提供了一个hook，允许以python dictionary的形式传递额外的参数给视图函数。
```python
urlpatterns = [
    path('blog/<int:year>/', views.year_archive, {'foo': 'bar'}),
]
```
上例中，如果请求`/blog/2018/`，Django会调用 `views.year_archive(request, year=2018, foo='bar')`
## 静态文件
在生产环境我们通常会使用单独的server来提供对静态文件的访问，Django提供了一个static()方法在调试的时候serve静态文件。
```python
urlpatterns = [
    # ... the rest of your URLconf goes here ...
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```


# Model
Django已经包含了ORM，因此只需要关注Model类，而不用关心SQL。前提是要在INSTALLED_APPS 中包含指定的app。
> *Tips：*如果使用PyCharm 的话，通过Ctrl + Alt + R 调出python manage.py 命令界面，可以直接执行migrate，shell 等命令，不用在python 命令行执行.

# Middleware
Django 里的Middleware 可以看成是一个hook，处于request/response 过程之间，是一个轻量级的、底层的插件系统，用于改变Django 的输入输出。

一个middleware factory 是一个callable，它接收一个get_response
 callable作为参数，返回一个middleware；一个middleware 是一个callable，它像view一样，接收一个request， 返回一个response。返回真正的view 之前可能经历数个中间件：在响应阶段，调用视图之前，Django按顺序从上到下应用MIDDLEWARE中定义的中间件。可以把这个结构看成一个洋葱，洋葱的中心是最后展示的view，它被一层层的中间件包裹起来，每一层都调用get_response把request 传递到下一层，直到view，然后response也会按原路返回直到最外层的中间件。 在request 和response 之间想要做的事情，都可以用middleware 来实现。