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
### 空格使用
经常看到用“游标卡尺”的梗来调侃Python，可见Python对缩进的严格要求，不正确的缩进格式甚至会导致代码无法正确编译，
如果使用PyCharm ，通过快捷键Ctrl + Alt + L 迅速格式化代码，默认遵循pep8 规范。
更多规范见 [PEP8](https://www.python.org/dev/peps/pep-0008/)

除了PEP8，Google 也有一套关于python的[语言和风格规范](http://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_language_rules/)。

