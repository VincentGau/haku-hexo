---
title: 写更pythonic 的代码
date: 2018-04-16 16:43:13
tags:
- python
- 编码问题
- encode
- decode
---
# Python版本
目前python有两个大版本，Python 2 和Python 3， 这两个版本互不兼容，除了语法上的差异，一些Python 2 的类库没有对应的Python 3 版本，在Python 3中无法使用，Python 3类库也可能不能用在Python 2 中。鉴于python 社区更关注Python 3 的特性和提升，针对Python 2 的更新范围包括bug修复与安全更新，，并且越来越多的开发者企业逐渐放弃对Python 2 的支持，对于新项目Python 3 是一个更长远的选择。不过有虚拟环境的支持，使用不同版本的python也不是问题。
安装python的时候默认的解释器是使用C语言开发的CPython，它编译python代码生成字节码然后执行，同样流行的还有JPython（Java），IronPython（.Net），PyPy（python）这些实现，其中CPython的使用更广泛。

<!-- more -->

# 遵循PEP8 编码规范
Python Enhancement Proposal #8（PEP8）是一套python 的编码规范，包括命名规范，缩进使用等，使用统一的编码规范对自己编写代码和方便别人阅读代码都是很有益的。遵循PEP8 来写出更pythonic 的代码。
## 命名规范
- 函数、变量、属性使用小写字母加下划线格式（eg. my_func）；
- 受保护的实例属性以下划线开头（eg. _protected_attr）；
- 私有属性以双下划线开头（eg. __private_attr）；
- 类和异常使用大写格式（eg. MyClass）；

## 空格使用
经常看到用“游标卡尺”的梗来调侃Python，可见Python对缩进的严格要求，不正确的缩进格式甚至会导致代码无法正确编译，
- 缩进使用空格，不用Tab；
- 每一层语法上的有效缩进是4个空格；
- 每行最多79个字符；
- 长表达式在新一行的延续部分额外使用4个空格；
- 类中的方法用一个空行隔开；   
...

如果使用PyCharm ，通过快捷键Ctrl + Alt + L 迅速格式化代码，默认遵循pep8 规范。

## 更Pythonic的表达式和声明
- 使用内部否定 `if a is not b`，而不用肯定表达式的否定 `if not a is b`；
- 使用 `if not somelist` 来检查是否为空，而不通过检查长度来判定`if len(somelist) == 0`；
- `import` 语句始终放在文件顶部；
- `import` 模块的时候使用绝对导入，如`from foo import bar`，而不是只 `import foo`；如果要做相对导入，也应该使用 `from . import foo`；
- 导入模块的时候按照既定的顺序：先导入标准库模块，然后第三方库，最后导入自己的模块；

更多规范见 [PEP8](https://www.python.org/dev/peps/pep-0008/)

[Pylint](https://www.pylint.org)是一款静态Python 源代码分析工具，检查代码中不合规范的地方，个人认为其作用不止于语法规范，对编码思路也有提升，强烈推荐。

除了PEP8，Google 也有一套关于python的[语言和风格规范](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_language_rules/)。  
建议使用一致的风格。

# Python对象
python中一切值都是对象；对象由内部数据和各种方法组成；python的对象有三个要素：`id`， `type`， `value`，其中id唯一标示一个对象，`type`表示对象类型，value是对象的值； `a is b`，判断的是二者的id是否相同，即二者是否同一对象，==比较二者的值；dir()列出对象所有可用方法；
作为动态语言，Python 对象的特征不由它的类型决定，而是通过它的方法决定，称为*鸭子类型*：
> 如果看起来像，叫起来像，走起来也像鸭子，那么它就是鸭子  


# 数据类型 `tuple(,)` `list[]` `dict{}` `set()`  
`tuple`（元组） 和`list`（列表） 非常类似，都是有序的列表，但是`tuple` 一旦初始化就不能修改，安全性更好；由于系统会为列表分配稍微多一些内存以优化添加新项时的性能， 而元组时不变的，不会占用额外的内存空间；因为小括号既可以表示tuple，也可以表示数学运算中的小括号，所以只有一个元素的`tuple`定义时需要加一个逗号来消除歧义，如`t = (1,)`；内置类型`list(s)`可将任何可迭代类型转换为列表，如果s已经是一个列表，则该函数构造的新列表是s的一个**浅拷贝**；  
`set`（集合）是无序的，不能通过数字进行索引；集合支持一系列标准操作，如求交集（&），并集（|），差集（-），对称差集（^， 在且仅仅在其中一个集合中）；`set`中不能存放可变对象，因为无法判断两个可变对象是否相等；创建非空set需要提供一个list作为输入；  
字典是python中最完善的数据类型；要获得字典的关键字列表，只需要将字典转换为列表：
```
l = list(d)
```
字典在放入一个键值对的时候已经根据key计算出value存放的位置（hash），所以字典的key不能是可变对象（如list），也不能是包含可变对象的对象，必须保证每次计算同一个key的结果相同；

# 弄清 `str` `bytes` `unicode`
在写Python代码的时候，常被编码问题困扰，不胜其烦又无可避免，这一节的目标彻底理解编码。  
- Python 2默认的编码方式是ASCII，而Python 3默认编码格式是Unicode，命名参数`encoding`在Python 3 中默认都是 `utf-8`；
- 在Python 2 中有两种类型来表示字符序列：`str` 和 `unicode`，`str`实例包含二进制数据，`Unicode`实例包含Unicode 字符串；Python 3中有两种类型表示字符序列： `str` 和`bytes`，`str`实例包含Unicode字符串，`bytes`实例包含二进制数据，二者不能通过运算符直接操作；
- 使用`encode`方法将Unicode 字符串转换成二进制数据（编码），反之用`decode`方法（解码）；  

错误的出现有时候是因为我们没有搞清楚对象的类型，比如当我们希望操作二进制数据的时候，实际上参数是UTF-8编码的字符；或者当我们在操作Unicode 字符的时候没有指定编码方式，默认ASCII编码，实际上可能是别的编码方式。为了保证输入的数据类型和我们所期望的一致，我们可以通过一个helper 方法，接收`str` 或者`bytes` 类型，始终返回`str`，或者始终返回`bytes`：
```python
# Python 3
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value # Instance of str
        
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value # Instance of str
```
还有容易产生混淆的地方是，Python 2里， 如果对象只包含ASCII码， 那么`str`和`unicode`看起来没什么区别，他们可以通过 + 运算符进行连接，也能进行比较，也能在格式化字符串`'%s''`中使用Unicode 对象，这意味着当形参需要是`str`时即使传入的参数是`unicode`（或者反之）程序也能执行；但是在Python 3 中，`bytes` 和`str`是完全不同的，即使是空串；

Python 3里涉及file handles（使用`open`打开文件）的默认UTF-8编码，Python 2则默认二进制编码，因此

# 切片
切片是Python的一个重要特性，提供简洁的语法方便获取子集，不仅可用于`list` `str`等内建数据类型，还可扩展至所有实现了`__getitem__` `__setitem__`方法的类。序列类型，包括字符串，列表，元素等，支持解包，索引，切片， `all`， `any`等操作  

切片最基本的使用方式是`somelist[start:end]`：
- 包含start，不包含end（前开后闭 `[)`）；
- 二者都可缺省，可以为负数，表示倒数第几个；
- start 和end 可以超过列表的边界不会报错；
- 切片返回的是一个新的列表对象，对它进行修改不会改变原列表；
```python
l = [1, 2, 3, 4, 5, 6, 7]
l[:]        # [1, 2, 3, 4, 5, 6, 7]
l[:3]       # [1, 2, 3]
l[2:]       # [3, 4, 5, 6, 7]
l[:-1]      # [1, 2, 3, 4, 5, 6]
l[-3,-1]    # [5, 6]
l[:20]      # [1, 2, 3, 4, 5, 6, 7]
l[-20:]     # [1, 2, 3, 4, 5, 6, 7]
```
切片在start end 之外还提供stride 参数，表示步进，每隔stride 取一个元素，`somelist[start: end: stride]`，通过切片可以方便地完成反转列表、取列表中奇数位元素等操作：
```python
l[::-1]     # 翻转列表
l[::2]      # 取下标为偶数的元素
l[1::2]     # 取下标为奇数的元素

```
当stride 为负数的时候表示从最后一个元素开始，并向前移动；如无必要尽量不要使用负数的stride，为了避免理解困难尽量不要同时指定三个参数，可以分成两步实现，一步切片，一步步进。

# 列表推导式
List Comprehensions 是Python最强大的特性之一，它提供了紧凑的语法从一个`list` 生成另一个`list`，比用`map()`方式更清晰，也更灵活，并且支持filter，先来感受一下：
```python
# 求列表中所有元素的平方
a = [1, 2, 3]
s = [i**2 for i in a]                   # s = [1, 4, 9]
等效于：
s = map(lambda x: x**2, a)

# 求列表中所有偶数元素的平方
s2 = [i**2 for i in a if i % 2 == 0]    # s2 = [4]
```
除了list， dict 和set 也支持列表推导式。

# 生成器
通过列表推导式我们处理输入序列的每一个元素得到一个新的列表，对于元素个数不多的序列这样没有问题，一旦元素多了就可能消耗过多内存导致程序崩溃，生成器解决了这个问题，它返回一个迭代器iterator， 而不是整个列表，不对括号内的表达式立即求值，需要的时候才进行运算，迭代地生成结果，像lazy calculate。感受一下，创建generator 的方法很简单，只需要把列表推导式的`[]` 换成`()`：
```python
a = [1, 2, 3]
g = (i**2 for i in a)   # g是一个生成器
print(g.next())         # 不断调用next()计算下一个元素的值，直到结束
print(g.next())         # 不断调用next()计算下一个元素的值，直到结束
print(g.next())         # 不断调用next()计算下一个元素的值，直到结束
print(g.next())         # 不断调用next()计算下一个元素的值，直到结束

# 也可以用for遍历
for i in g: 
    print i
```

另一种定义生成器的方式是使用`yield`关键字，使用`yield`可以让函数生成一个结果序列，而不仅仅是一个值；
```
def countdown(n)
    print 'Counting down'
    while n > 0:
        yield n     #生成一个值（n）
        n -= 1
```
**任何使用`yield`的函数都称为生成器**，生成器是一个函数，它生成一个序列，以便在迭代中使用；调用生成器函数将创建一个对象，该对象通过连续调用`next()`方法生成结果序列；对象每次调用`next()`遇到`yield`返回，再次执行时从上次返回的`yield`处继续执行，遇到return或者最后一条语句结束；生成器传递给`yield`的任何值都会返回给调用者；生成器的微妙之处在与它常常可以与其他可迭代的对象（列表/文件等）混合在一起使用；  
在需要返回一个列表的时候用生成器函数代替，首先代码会更清晰，其次使用生成器不必担心占用过多内存，因为它不必保存所有的输入（比如当输入是流的时候，对于文件也能逐行读取）和输出；

# `enumerate` & `range`
内建方法`range()`遍历一组数字，在Python 2中，`range()`返回列表，`xrange()`迭代器，Python 3中取消了`xrange()`只保留`range()`，返回迭代器。
```python
for i in range(10):
    print i**2
```
有时我们在遍历一个列表的时候还希望知道当前元素的下标，这时可以使用索引迭代`enumerate()`，它和`range()`功能相仿，只是还会额外提供下标：
```python
a = [1, 2, 3]
for i, value in enumerate(a):
    print(f'{i}:{value**2}')
    
0:1
1:4
2:9
```
这里用到了`f前缀`，Python 3.6的新特性，方便格式化字符串；  
可以给`enumerate()`提供第二个参数指定`i`从哪个数字数起，默认是0， 比如：
```python
a = [1, 2, 3]
for i, value in enumerate(a, 1):
    print(f'{i}:{value**2}')
    
1:1
2:4
3:9
```

# zip
`zip()`同时遍历多个可迭代对象，将他们打包成`tuple` 返回，直接看代码：
```python
names = ['Haku', 'Vincent', 'G']
numbers = [1, 2, 3]
for name, number in zip(names, numbers):
    print(f'{name}: {number}')
    
Output:    
Haku: 1
Vincent: 2
G: 3
```
Python 2中`zip()`返回包含`tuple` 的`list`，有可能占用过多内存，Python 3中返回的是产生`tuple` 的生成器。`zip`对象可以直接转换成`list` 或者`dict`。  
```python
z = zip(names, numbers)
list(z)     # 转换成list [('Haku', 1), ('Vincent', 2), ('G', 3)]
dict(z)     # 转换成dict {'Haku': 1, 'Vincent': 2, 'G': 3}
```
如果两个可迭代对象长度不同，`zip()`会自动截断较长的一个，以较短的为准。


# 紧跟在循环后面的`else`
Python 的循环有一个其他语言没有的特性，它后面可以紧跟一个`else`代码块，除非在循环体中执行到`break`语句，否则`else`代码块会被执行，有点反直觉，上代码：
```python
for i in range(3):
    print(i)
else:
    print('Else block.')
```
循环体中没有执行到`break`语句，所以会输出`Else block.`;
```python
for i in range(3):
    print(i)
    if i == 2:
        break
else:
    print('Else block.')
```
循环体中执行了`break`语句，所以不会输出`Else block.`;  
除非知道自己在做什么，否则尽量不用 `for...else...` `while...else...` 这种用法。

# 上下文管理器
with语句支持在另一个被称为*上下文管理器*的对象的控制下执行一系列语句，适用于系统资源或执行环境相关的对象，用来简化异常处理；执行with语句时就会调用`__enter__()`方法，只要控制流离开与with相关的语句就会调用`__exit__()`方法；

# 浅复制和深复制
```python
a = [1,2,3]
b = a
```
这个例子中， a和b引用的是同一个对象，修改其中任意一个变量都会影响另一个，为了避免这种情况应该创建对象的副本，而不是新引用；对于列表和字典这样的容器对象，可以使用两种复制操作：深复制和浅复制；浅复制将创建一个新对象，但他包含的是对原始对象中包含的项的引用（）；深复制将创建一个新对象，并且递归地复制它所包含的对象，用标准库的`copy.deepcopy()`函数可以实现深复制；

# 垃圾回收
Python通过引用计数机制实现自动垃圾回收功能，Python中的每个对象都有一个引用计数，用来计数该对象在不同场所分别被引用了多少次。无论是给对象分配一个新的名称，还是将其放入一个容器中，该对象的引用计数就会增加1，每当消毁一次Python对象，则相应的引用就减1，只有当引用计数为零时，才真正从内存中删除Python对象。

# 参数
## 值传递和引用传递
python的参数属于引用传递还是值传递不由用户决定，而是根据参数是否可变决定，可变参数应用引用传递，不可变参数应用值传递，为了实现不可变参数的引用传递，可以先把他变成一个可变参数，如int型转换成包含一个int的list，这样list的值就可以改变，相当于改变了int；实参发生变化；或者对公用的数据使用全局变量；

## 默认参数
使用可变对象作为默认参数会导致意外情况发生，默认参数会保留前面调用产生的更改，为防止这种情况发生，最好将可变参数默认值设置为`None`， 并在函数内加上检查代码；上代码：
```python
def add_to(somelist=[]):
    somelist.append('A')
    print(somelist)

add_to()    # ['A']
add_to()    # ['A', 'A']
add_to()    # ['A', 'A', 'A']
```
上述代码中，三次执行方法得到的结果不同，与我们的预期不相符，直觉上，三次调用都没有传参，都是使用默认参数空列表`[]`，应该结果一样才是；而实际上如上一条所述，可变参数应用引用传递，函数定义的时候默认参数`somelist`的值就确定了，它指向`[]`，每次调用函数如果改变了它的值，那么下次调用的时候就会用改变后的值，**默认参数只会计算一次**；为了达到预期，可作如下修改，将默认参数的默认值设置为不可变对象`None`：
```python
def add_to(somelist=None):
    if somelist is None:
        somelist = []
    somelist.append('A')
    print(somelist)
```

## 可变参数
函数可以接受不定数量（0个或多个）的参数，称为可变参数，用法是在参数前加上星号`*`：
```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum += n
    return sum
        
disp(1, 2)          # 返回3
disp(2, 3, 4)       # 返回9
```
可变参数在传递给函数之前会首先被转换成`tuple`, 函数感知到的是接受一个`tuple`类型参数，如果已经有了一个`tuple`或`list`，要使用可变参数，只需要在调用时给该参数加上`*`前缀：
```python
num = [1, 2, 3]
calc(*num)          # 返回6
```

顺便提一句，静态语言需要用函数重载来解决可变参数类型和可变参数个数问题，python作为一种动态语言可以接受任意类型的参数，不存在第一种情况，而第二种情况可以可以通过设定默认参数和可变参数来解决；

## 关键字参数
传参的时候可以带上关键字，位置参数也可以通过关键字传入，只要位置参数都已指定，关键字参数的顺序没有限制，位置参数没有全部指定时，不能使用关键字参数：
```python
def func(name, age):
    print(f'{name}: {age}')
    
func('Haku', 18)
func('Haku', age=18)
func(name='Haku', age=18)
func(age=18, name='Haku')

func(name='Haku', 17)       # Error  不能在关键字参数后面跟着位置参数
func(17, name='Haku')       # Error  17被认为是name参数，后面关键字再次传入了name
```
关键字参数允许以dict 形式传入0个或多个参数名的参数，用`**`标识：
```python
def func(**kw):
    print(kw)

func(a=1, b=2)              # 输出{'a': 1, 'b': 2}

d = {'a': 1, 'b': 2}
func(**d)                   # 输出{'a': 1, 'b': 2}
```

## 参数顺序
按照位置参数，默认参数，可变参数，关键字参数的顺序定义参数；

## 强制关键字
强制传参是带上关键字，使函数目的更明确，避免混淆参数位置带来的意外：
```python
def to_upper(my_str, initial_only=False, limit_len=False):
    if ignore_case:
        ...
    if limit_len:
        ...
    ...
    
to_upper('haku', False, False)
```
`func()`函数将字符串`my_str`转换成大写格式，`initial_only`参数表示是否只将首字母大写， `limit_len`参数表示是否限制给定字符串长度;  函数调用如下：
`to_upper('haku', False, False)`  
如此函数可以正常调用，但是容易混淆两个`Boolean`参数，并且难以`Debug`, 更好的调用方式应该是调用者明确意图：
`to_upper('haku', initial_only=False, limit_len=False)`
重新定义`to_upper()`函数，增加一个可变参数占位，对于这两个容易混淆的函数必须带上关键字，否则调用失败：
```python
def to_upper(my_str, *, initial_only=False, limit_len=False):
    if ignore_case:
        ...
    if limit_len:
        ...
    ...
```


# 作用域
每执行一个函数时，就会创建局部命名空间，解析变量时解释器首先搜索局部命名空间，再搜索全局命名空间，最后检查内置命名空间，仍然找不到则引发`NameError`异常；函数定义时就确定了变量是全局还是局部，不能改变作用域；如果使用局部变量时还没有给它赋值，就会引发异常，所以下面的代码会引发异常：
```python
i = 0
def foo():
    i = i + 1   #i是局部变量，在赋值前尝试读取i的值会失败
```
在函数中对变量进行赋值时，这些变量始终绑定到该函数的局部命名空间中，使用`global`语句可以改变这种行为；尽管可以访问嵌套作用域中的变量，Python 2只支持在最里层的作用域和全局命名空间（global）中给变量重新赋值，因此内部函数不能给定义在外部函数中的局部变量重新赋值，解决方法时把要修改的值放在列表或者字典里；在Python 3里可以将变量声明为`nonlocal`，`nonlocal`声明不会将名称绑定到任意函数中定义的局部变量，而是搜索当前调用栈中的下一层函数定义，即*动态作用域*；

# 装饰器
## @property 装饰器
python 内置的 @property装饰器将一个方法变成属性来调用，既能避免使用set get方法的繁琐，又保留了参数检查的功能；设置property后，对属性的访问将由一系列用户定义的get，set，delete函数控制；这种属性控制可以通过描述符对象进一步扩展；

# 闭包Closure
将组成函数的语句和函数的执行环境打包在一起，成为闭包。如果需要在一系列函数调用中保持某个状态，使用闭包是一种高效的方法；把函数当数据处理时，它将显式地携带与定义该函数的周围环境相关的信息；使用嵌套函数时，闭包将捕捉内部函数执行所需要的整个环境；如果需要编写惰性求值或者延迟求值的代码，闭包将会特别有用（通过嵌套函数的方式，返回函数）；
Python 支持闭包的前提是所有对象都是*第一类*的，意思是能够命名的对象都可以当做数据处理；函数是第一类对象，可以直接引用，也可以把它们赋值给变量，把它们作为参数传递个别的函数等等；

# 协程 Coroutine





参考：
[《Effective Python》](https://effectivepython.com/)