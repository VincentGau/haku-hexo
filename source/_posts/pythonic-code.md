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
目前python有两个大版本，Python 2 和Python 3， 这两个版本互不兼容，除了语法上的差异，一些Python 2 的类库没有对应的Python 3 版本，在Python 3中无法使用，Python 3类库也可能不能用在Python 2 中。鉴于python 社区更关注Python 3 的特性和提升，Python 2 除了修复Bug已经不会有发展，并且越来越多的开发者企业逐渐放弃对Python 2 的支持，对于新项目Python 3 是一个更长远的选择。不过有虚拟环境的支持，使用不同版本的python也不是问题。
安装python的时候默认的解释器是使用C语言开发的CPython，它编译python代码生成字节码然后执行，同样流行的还有JPython（Java），IronPython（.Net），PyPy（python）这些实现，其中CPython的使用更广泛。

<!-- more -->

# 遵循PEP8 编码规范
Python Enhancement Proposal #8（PEP8）是一套python 的编码规范，包括命名规范，缩进使用等，使用统一的编码规范对自己编写代码和方便别人阅读代码都是很有益的。遵循PEP8 来写出更pythonic 的代码。
### 命名规范
- 函数、变量、属性使用小写字母加下划线格式（eg. my_func）；
- 受保护的实例属性以下划线开头（eg. _protected_attr）；
- 私有属性以双下划线开头（eg. __private_attr）；
- 类和异常使用大写格式（eg. MyClass）；

### 空格使用
经常看到用“游标卡尺”的梗来调侃Python，可见Python对缩进的严格要求，不正确的缩进格式甚至会导致代码无法正确编译，
- 缩进使用空格，不用Tab；
- 每一层语法上的有效缩进是4个空格；
- 每行最多79个字符；
- 长表达式在新一行的延续部分额外使用4个空格；
- 类中的方法用一个空行隔开；   
...

如果使用PyCharm ，通过快捷键Ctrl + Alt + L 迅速格式化代码，默认遵循pep8 规范。

### 更Pythonic的表达式和声明
- 使用内部否定（if a is not b），而不用肯定表达式的否定（if not a is b）；
- 使用 if not somelist 来检查是否为空，而不通过检查长度来判定（if len(somelist) == 0）；
- import 语句始终放在文件顶部；
- import 模块的时候使用绝对导入，如from foo import bar，而不是只 import foo；如果要做相对导入，也应该使用 from . import foo；
- 导入模块的时候按照既定的顺序：先导入标准库模块，然后第三方库，最后导入自己的模块；

更多规范见 [PEP8](https://www.python.org/dev/peps/pep-0008/)

[Pylint](https://www.pylint.org)是一款静态Python 源代码分析工具，检查代码中不合规范的地方，个人认为其作用不止于语法规范，对编码思路也有提升，强烈推荐。

除了PEP8，Google 也有一套关于python的[语言和风格规范](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_language_rules/)。  
建议使用一致的风格。

# 弄清 `str` `bytes` `unicode`
在写Python代码的时候，常被编码问题困扰，不胜其烦又无可避免，彻底理解编码就是这一节的目的。  
- Python 2默认的编码方式是ASCII，而Python 3默认编码格式是Unicode，命名参数`encoding`在Python 3 中默认都是 `utf-8`；
- 在Python 2 中有两种类型来表示字符序列：`str` 和 `unicode`，`str`实例包含二进制数据，`Unicode`实例包含Unicode 字符；Python 3中有两种类型表示字符序列： `str` 和`bytes`，`str`实例包含Unicode字符，`bytes`实例包含二进制数据，二者不能通过运算符直接操作；
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


# 切片
切片是Python的一个重要特性，提供简洁的语法方便获取子集，不仅可用于`list` `str`等内建数据类型，还可扩展至所有实现了`__getitem__` `__setitem__`方法的类。  

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
切在start end 之外还提供stride 参数，表示每隔stride 取一个元素，`somelist[start: end: stride]`，通过切片可以方便地完成反转列表、取列表中奇数位元素等操作：
```python
l[::-1]     # 翻转列表
l[::2]      # 取下标为偶数的元素
l[1::2]     # 取下标为奇数的元素

```
当stride 为负数的时候表示从最后一个元素开始，并向前移动；如无必要尽量不要使用负数的stride，为了避免理解困难尽量不要同时指定三个参数，可以分成两步实现，一步切片，一步步进。

# 列表推导式
List Comprehensions 是Python最强大的特性之一，它提供了紧凑的语法从一个list 生成另一个list，比用`map()`方式更清晰，也更灵活，先来感受一下：
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
通过列表推导式我们处理输入序列的每一个元素得到一个新的列表，对于元素个数不多的序列这样没有问题，一旦元素多了就可能消耗过多内存导致程序崩溃，生成器解决了这个问题，它返回一个迭代器iterator， 而不是整个列表，需要的时候才进行运算，像lazy calculate。感受一下，创建generator 的方法很简单，只需要把列表推导式的`[]` 换成`()`，
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

另一种定义生成器的方式是使用`yield`关键字，

# `enumarate` & `range`
内建方法`range()`遍历一组数字，在Python 2中，`range()`返回列表，`xrange()`迭代器，Python 3中取消了`xrange()`只保留`range()`，返回迭代器。
```python
for i in range(10):
    print i**2
```
有时我们在遍历一个列表的时候还希望知道当前元素的下标，这时可以使用索引迭代`enumerate()`，它和`range()`功能相仿，只是还会额外提供下标：
```python
a = [1, 2, 3]
for i, value in enumarate(a):
    print(f'{i}:{value**2}')
    
0:1
1:4
2:9
```
这里用到了`f前缀`，这时Python 3.6的新特性，方便格式化字符串；  
可以给`enumerate()`提供第二个参数指定`i`从哪个数字数起，默认是0， 比如：
```python
a = [1, 2, 3]
for i, value in enumarate(a, 1):
    print(f'{i}:{value**2}')
    
1:1
2:4
3:9
```

# zip
`zip()`同时遍历多个可迭代对象，将他们打包成tuple 返回，直接看代码：
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
Python 2中`zip()`返回包含tuple 的list，有可能占用过多内存，Python 3中返回的是产生tuple 的生成器。zip对象可以直接转换成list 或者dict。  
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
除非知道自己在做什么，尽量不用 `for...else...` `while...else...` 这种用法。



参考：
[《Effective Python》](https://effectivepython.com/)