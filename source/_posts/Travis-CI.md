---
title: Travis 持续集成
date: 2018-04-19 15:05:04
tags:
- Travis
- Github
- CI
---
使用Travis进行持续集成，会自动配置webhook，在push完成之后执行既定操作。

1. 直接使用Github账号登录Travis， 勾选需要持续集成的代码库；  
2. 在项目根目录添加.travis.yml文件：
```python
language: python
python:
  - "3.6"
install:
  - pip install -r requirements.txt
script:
  - pytest
```
push代码到代码库之后脚本自动执行，在Travis可以看到执行日志；

异常情况：
- 运行测试出错，提示找不到模块，ModuleNotFoundError: No module named 'blockchain'
    - 在tests目录增加空的__init__.py 文件
- 运行测试出错，提示找不到文件，FileNotFoundError: [Errno 2] No such file or directory: '../src/sites.json'
    - TODO

{% blockquote %}
注：TravisCI不支持MSTest，因为MSTest只能在Windows下运行，而目前Travis只在Linux 或者OSX上通过Mono或.Net Core runtime构建C#项目；
{% endblockquote %}

在README.md文件增加以下内容，显示构建徽章；
`[![Build Status](https://www.travis-ci.org/VincentGau/pythonScripts.svg?branch=master)](https://www.travis-ci.org/VincentGau/pythonScripts)`