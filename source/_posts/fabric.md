---
title: 命令行工具库 fabric
date: 2018-01-18 17:10:34
tags:
- python
- 自动化部署
---
fabric 是一个python库（支持python2.5 到2.7 版本），也是一个命令行工具，，它的一个重要的功能是**在不登录远程服务器的情况下，在本地执行远程shell命令，上传下载文件等等**，这个特性让我们可以通过fabric 来进行应用部署，以及执行一些系统运维自动化的任务，而且是可以在多台服务器上执行。

> It provides a basic suite of operations for executing local or remote shell commands (normally or via sudo) and uploading/downloading files, as well as auxiliary functionality such as prompting the running user for input, or aborting execution.

<!-- more -->

# 安装
安装fabric：激活python2 虚拟环境，pip install fabric


# 使用SSH key
fabric 提供密码和密钥方式登录远程服务器，为了方便和安全起见，密钥是更好的选择。
## 生成SSH key
首先我们需要在远程服务器生成密钥，在远程执行：  
`$ ssh-keygen`  
在.ssh/ 目录下生成默认rsa算法加密的密钥对（id_rsa 私钥，id_rsa.pub 公钥）
## 发布公钥
密钥生成之后，需要把公钥的内容添加到.ssh 目录下的 _authorized_keys_ 文件，  
``` bash
$ cd ~/.ssh
$ cat id_rsa.pub >> authorized_keys
```  
修改权限
```bash
$ chmod 600 authorized_keys
$ chmod 700 ~/.ssh
```
## 配置fabric 环境参数
将私钥下载到本地，指定fabric 环境配置
`env.key_filename = "path/to/id_rsa"`


# 示例
fabric 提供的API 中，常用的有`local()` 和`run()`，分别在本地和远程服务器执行命令。fabric 文件的一个函数对应一个操作。 

举一个简单的例子，如需要列出远程服务器"/"下的文件和目录，在本地新建fabfile.py 文件如下：

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

fabfiles 通常放在项目的根目录。

[fabric 文档](http://docs.fabfile.org/en/1.14/)