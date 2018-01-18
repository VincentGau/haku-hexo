---
title: fabric-python命令行工具库
date: 2018-01-18 17:10:34
tags:
- python
---
fabric 是一个python库，也是一个命令行工具，支持python2.5 到2.7 版本，**不用登录远程服务器，在本地执行远程命令**，这个令人兴奋的特性让我们可以通过fabric 来进行应用部署，以及执行一些系统运维自动化的任务。

> It provides a basic suite of operations for executing local or remote shell commands (normally or via sudo) and uploading/downloading files, as well as auxiliary functionality such as prompting the running user for input, or aborting execution.

安装fabric：激活python2 虚拟环境，pip install fabric

fabric 提供的API 中，常用的有local() 和run()，分别在本地和远程服务器执行命令。fabric 文件的一个函数对应一个操作。 

fabric 提供密码和密钥方式登录远程服务器，举一个简单的例子，如需要列出远程服务器"/"下的文件和目录，在本地新建fabfile.py 文件如下：

```$python
#!/usr/bin/python env
# -*- coding: utf-8 -*-

from fabric.api import *
env.hosts=['username@xxx.xxx.xxx.xxx']
env.key_filename = "id_rsa"   # 通过密钥登录远程服务器

def hello():
    with cd('/'):     # 进入目录
        run('ls -l')  # run() 执行远程操作 列出远程机器上的文件和目录
```

执行 `fab hello` 命令列出文件和目录，和在远程服务器上执行一样。

[fabric 文档](http://www.fabfile.org/index.html)