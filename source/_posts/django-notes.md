---
title: Django 笔记
tags:
  - django
  - python
  - programming
date: 2018-01-12 14:59:48
---

Django是一个用python写的轻量级Web框架，虽说是轻量级，但是它能做的事情并不少，有了Django我们就可以用很少的代码来搭建一个现代化的网站，避免重复造轮子。Django2.0之后已经放弃了对python2 的支持，只支持python3.4 及以上版本。

Django框架包含了一个完整Web应用各方面的知识点，下面对Django一些常用的特性做一个梳理，旨在学习Django的架构和源码。

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

> Django2.0 已经不在路由规则中使用符号 `^`、 `$` ，并且将`url`函数改成`path`函数。

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
 
# Authentication
Django 的Authentication 系统处理用户组和账户，用户许可和基于cookie的session，实际上它包含Authentication 和Authorization 两部分，前者负责确认用户的身份，后者决定一个认证用户可以做的事情。

## 自定义User模型
User 对象是认证系统的核心，Django提供了默认的User模型，但是还是推荐使用自定义的user 模型，在将来提供更好的扩展性。
> If you’re starting a new project, it’s highly recommended to set up a custom user model, even if the default User model is sufficient for you. This model behaves identically to the default user model, but you’ll be able to customize it in the future if the need arises.

使用自定义的User模型之前需要三个步骤：  
- 继承AbstractUser 类：
```python
class User(AbstractUser):
    pass
```
- settings.py 中指定 AUTH_USER_MODEL = 'appname.User'
- 在app 的admin.py中注册User模型：
```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```
## Authentication Views
Django 提供了一些默认视图（Authentication Views）处理登录，注销和修改密码操作，可以直接拿来用。如果需要更多个性化的处理，不想使用内置的视图，在项目中新建一个app来处理这些用户操作。

## buit-in forms
如果不想使用内置的视图，但又不想自己写各种表单，authentication 系统提供了一些内置的form。


# Testing
TDD 是敏捷开发的一个核心实践，通过测试来推动开发的进行，不写测试用例节省的时间有可能在将来付出成倍的代价。自动化测试可以发现迭代过程中对已有功能的不可预期的影响，及时解决问题。

Django 的单元测试使用Python的标准库unittest，通过class-based 的方法定义测试用例，执行测试的时候会检查所有以test 开头的文件。

## 测试提速
### 测试并行
如果测试用例相互独立，可以并行执行测试以提升效率。执行test 加上--parallel参数，比如`test --parallel=4`
### 保留测试数据库
保留上一次运行测试创建的册数数据库，节省创建和销毁操作，大大减少运行测试的时间。`test --keepdb`
### 密码哈希
默认的密码哈希算法相当耗时，如果需要在测试用例中大量认证用户，可以自定义hash 算法，在settings.py中设置PASSWORD_HASHERS，指定特定的哈希算法，比如
```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]
```

## 测试工具
### 测试用户登陆状态
调用force_login方法模拟用户已登陆状态，无须先创建用户再模拟登陆：
`self.client.force_login(User.objects.get_or_create(username='testuser')[0])`

### 测试静态文件
调试和发布的时候常常会找不到静态文件，检查静态文件是否被正常serve
```python
from django.contrib.staticfiles import finders

result = finders.find('css/base.css')
```
如果找到静态文件，find() 会返回文件的全路径，否则返回None；

### 日志
Django 使用Python 内置的logging 模块来实现系统日志；一个日志配置包含四个部分：Loggers, Handlers, Filters, Formatters；

## 基本定义
Logger 是日志系统的入口，每一个logger 都是一个named bucket，信息被写入logger 以作后续操作；**logger 有其日志级别**，每一条日志记录也有其级别，只有日志记录的级别高于等于logger 级别的时候，该日志记录才会继续后续的处理，否则会被忽略；当logger 决定一条信息需要被处理的时候，该信息被传递给一个Handler；

Handler 是一个决定logger 中的每一条信息将如何被处理的引擎，它描述一个具体的日志行为，比如把信息输出到屏幕，文件或者socket； **handler 也有级别**，如果日志记录的级别低于handler 的级别，它会被忽略；一个logger 可以有多个handler， 每个handler 可以有不同的日志级别，这样可以根据信息的重要性来提供不同形式的通知，比如可以单独使用一个handler 处理CRITICAL 级别的日志记录，发送告警，同时用另一个handler 处理所有日志记录，写入文件以备后续分析；

Filter 对日志从logger 传递到handler 提供更多的控制，默认情况下，任何符合级别要求的日志记录都会被处理，加上过滤器之后可以给日志处理增加额外的条件；Filter 还可以在日志记录被发出之前对它进行修改，比如写一个过滤器，在符合特定条件的情况下，把日志记录的级别从ERROR 降到WARNING；filter 可以被用在logger 或者handler 上，多个filters 可以通过链式实现过滤操作；一个比较常见的filter 是require_debug_false， 当DEBUG设置为False才处理日志记录；

Formatter 描述具体的文本格式，将日志记录渲染成文本；

处理流程如下：
```python
logger ---------> handlers ---------> formatter ---> files, emails .etc
        filters             filters
```

## Django 的日志组件
Django 提供了一些工具来处理Web 服务环境下的日志，包括内置的一些loggers, filters 和一个AdminEmailHandler；如果logging 配置字典中disable_existing_loggers 值设置成True, 默认配置中的所有logger 会被禁用（禁用不同于移除，这些logger 依然存在，但会丢弃所有给它的内容），为了避免预期之外的情况，最好将其设置成False，如果需要可重定义一些默认的logger；
### AdminEmailHandler
顺带提一句日志配置中的邮件设置，当DEGUG设置为False 的时候，如果在setting.py 中同时设置了ADMINS参数，Django 会在发生500错误的时候发送邮件给ADMINS 中的用户；默认情况下，Django 将从root@localhost 发送邮件，为了保证邮件正确接收，需要设置SERVER_EMAIL 参数为自己的发送邮箱地址；
如果发现邮件发送失败，检查DEBUG是否设置成False，smtp服务器的用户名和密码是否正确，是否设置了SERVER_EMAIL参数；
```python
ADMINS = (
    ('haku', 'g@kohaku.cc'),
)

MANAGERS = ADMINS

# Email
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_USE_TLS = True
EMAIL_HOST = local_settings.EMAIL_HOST
EMAIL_PORT = local_settings.EMAIL_PORT
EMAIL_HOST_USER = local_settings.EMAIL_HOST_USER
EMAIL_HOST_PASSWORD = local_settings.EMAIL_HOST_PASSWORD
DEFAULT_FROM_EMAIL = local_settings.DEFAULT_FROM_EMAIL
SERVER_EMAIL = local_settings.DEFAULT_FROM_EMAIL
```
为了检测配置是否正确，在本地运行Python 内置的SMTP 测试服务器:
`python -m smtpd -n -c DebuggingServer localhost:1025`

然后在settings.py 设置：
```python
EMAIL_HOST='localhost'
EMAIL_PORT=1025
```
触发一个500错误，或者生成一条error及以上级别的日志`logger.error('test')`，在终端上会显示发送的邮件内容；
### 404 Error Reporting
当DEBUG设置成False，并且在MIDDLEWARE设置中包含了django.middleware.common.BrokenLinkEmailsMiddleware，Django就会在抛出404异常并且该请求头**包含referer** 的时候给MANAGERS 列表中的用户发送邮件，这样做既能检测Broken Links，又能避免因为用户手动输入一个无效的地址而报错；


