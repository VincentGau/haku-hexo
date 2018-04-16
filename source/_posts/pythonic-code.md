---
title: 写更pythonic 的代码
date: 2018-04-16 16:43:13
tags:
- python
---
# Python版本
目前python有两个大版本，python2 和python3， 这两个版本互不兼容，除了语法上的差异，一些python2 的类库没有对应的python3 版本，在python3中无法使用，python3类库也可能不能用在python2 中。鉴于python 社区更关注python3 的特性和提升，python2 除了修复Bug已经不会有发展，并且越来越多的开发者企业逐渐放弃对python2 的支持，对于新项目python3 是一个更长远的选择。不过有虚拟环境的支持，使用不同版本的python也不是问题。
安装python的时候默认的解释器是使用C语言开发的CPython，它编译python代码生成字节码然后执行，同样流行的还有JPython（Java），IronPython（.Net），PyPy（python）这些实现，其中CPython的使用更广泛。

# 遵循PEP8 编码规范
Python Enhancement Proposal #8（PEP8）是一套python 的编码规范，主要包括命名规范，缩进使用等，使用同一的编码规范对自己编写代码和方便别人阅读代码都是很有益的。遵循PEP8 来写出更pythonic 的代码。
### 命名规范
- 函数、变量、属性使用小写字母加下划线格式（eg. my_func）；
- 受保护的实例属性以下划线开头（eg. _protected_attr）；
- 私有属性以双下划线开头（eg. __private_attr）；
- 类和一场使用大写格式（eg. MyClass）；

### 空格使用
经常看到用“游标卡尺”的梗来调侃Python，可见Python对缩进的严格要求，不正确的缩进格式甚至会导致代码无法正确编译，
- 缩进使用空格，不用Tab；
- 每一层语法上的有效缩进是4个空格；
- 每行最多79个字符；
- 长表达式在新一行的延续部分额外使用4个空格；
- 类中的方法用一个空行隔开；   
...

如果使用PyCharm ，通过快捷键Ctrl + Alt + L 迅速格式化代码，默认遵循pep8 规范。

### Pythonic的表达式和声明
- 使用内部否定（if a is not b），而不用肯定表达式的否定（if not a is b）；
- 使用 if not somelist 来检查是否为空，而不通过检查长度来判定（if len(somelist) == 0）；
- import 语句始终放在文件顶部；
- import 模块的时候使用绝对导入，如from foo import bar，而不是只 import foo；如果要做相对导入，也应该使用 from . import foo；
- 导入模块的时候按照既定的顺序：先导入标准库模块，然后第三方库，最后导入自己的模块；

更多规范见 [PEP8](https://www.python.org/dev/peps/pep-0008/)

[Pylint](https://www.pylint.org)是一款静态Python 源代码分析工具，检查代码中不合规范的地方，个人认为其作用不止于语法规范，对编码思路也有提升，强烈推荐。

除了PEP8，Google 也有一套关于python的[语言和风格规范](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_language_rules/)。建议使用一致的风格。

# 弄清`str` `bytes` `unicode`
在写Python代码的时候，因为理解不透彻常被编码问题困扰，不胜其烦又无可避免，彻底理解编码就是这一节的目的。  
- Python2默认的编码方式是ASCII，而Python3默认编码格式是Unicode；
- 在Python2 中有两种类型来表示字符序列：`str` 和 `unicode`，`str`实例包含二进制数据，`Unicode`实例包含Unicode 字符；Python3中有两种类型表示字符序列： `str` 和`bytes`，`str`实例包含Unicode字符，`bytes`实例包含二进制数据；
- 使用`encode`方法将Unicode 字符串转换成二进制数据（编码），反之用`decode`方法（解码）；