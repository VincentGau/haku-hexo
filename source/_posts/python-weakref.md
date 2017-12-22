---
title: Python 弱引用
date: 2017-05-23 09:35:34+08
tags:
---
Python 的 <code>[weakref](https://docs.python.org/2/library/weakref.html)</code> 模块实现对对象的弱引用（Weak References）。我们知道，Python 的垃圾回收机制基于引用计数器实现，实例创建后将由引用计数器管理，对实例的每一次引用都会使其引用计数器加1，引用的删除使引用计数器减1，如果引用计数器到达0实例将被销毁。但是有时候我们希望在内存需要的时候更早地销毁一个对象，比如发生循环引用时，或者在建立一个对象缓存时；弱引用不能保证对象不被销毁，当一个对象只存在弱引用时，它也会被销毁。

<!-- more -->

弱引用的一个基本用法就是实现缓存和大对象映射，使一个大对象不会因为存在于缓存或者一个键值对中而逃避被销毁。例如，我们有一些大的二进制图片对象，现在打算把他们和名称一一对应，如果使用 Python 的 `dictionary` 将名称映射到图片对象（或者反过来），图片对象会一直存在于内存中，因为它们依然作为字典的值（或者键）存在，弱引用可以解决这个问题，使用基于`weakref`模块实现的 `WeakKeyDictionary` 和 `WeakValueDictionary` ,用弱引用来构造映射关系，我们假设图片对象是 `WeakValueDictionary` 的值，当该对象的引用只是 weak mapping 中的弱引用时，GC会回收该对象，weak mapping 中对应的条目也会被删除；一些内建类型比如 `list`， `dict` 不直接支持弱引用，但可以通过子类增加弱引用支持；对对象的弱引用不会增加其引用计数，见下面代码：

``` python
import sys
import weakref


class A(object):
    pass


a = A()
print sys.getrefcount(a)            # 2

b = a
print sys.getrefcount(a)            # 3

c = weakref.ref(a)
print sys.getrefcount(a)            # 3
print weakref.getweakrefcount(a)    # 1

del b
print sys.getrefcount(a)            # 2

del c
print sys.getrefcount(a)            # 2
print weakref.getweakrefcount(a)    # 0
```

从上面的例子可以看到，对对象的弱引用使用单独的计数器；之所以第一次获取引用计数结果是 `2`而不是 `1`，是因为把 `a` 当参数传入了 `getrefcount()` 方法。

总结，弱引用主要解决垃圾回收的问题，多用在处理对象缓存和循环引用问题上；回顾一下弱引用在 `django` 中使用的场景，`Signal` 类初始化的时候就定义了一个 `receiver` 缓存，在连接`receiver` 和信号的时候，默认对 `receiver` 使用弱引用；`weakref` 模块的方法和属性详见 [weakref](https://docs.python.org/2/library/weakref.html)


相关文档：[https://pymotw.com/2/weakref/index.html](https://pymotw.com/2/weakref/index.html)