---
title: Go into PyCharm
date: 2018-01-17 15:53:01
tags:
- pycharm
- programming
---
开发Python项目首选PyCharm，PyCharm有很多好用的强大的特性，略作记录以备后用。

# Automatic Upload
除了集成Git 等版本控制工具，pycharm 还提供自动上传文件到远程主机，以及调用远程主机解释器的功能。配置好SSH连接到远程主机之后，在本地工作区所做的修改可以被自动同步到远程主机上.

<!-- more -->

- 因为安全原因，我在远程主机禁止用户名密码方式，只允许私钥登录，因此需要提前准备Private key 文件以在IDE 中连接远程主机；

> **PS:** 在远程主机生成的ppk私钥文件不被pycharm 支持，需要通过pyttygen 工具进行转换：打开puttygen.exe， Conversation -> Import 导入已生成的ppk 私钥文件，密码可选，然后 Conversation -> Export OpenSSH Key 导出备用。

- PyCharm菜单栏进入Tools -> Deployment -> Configuration，新增配置，类型选择SFTP，配置远程主机及端口，选择验证方式为Key Pair，私钥文件选择上一步骤导出的文件，测试SFTP 连接是否成功。

{% asset_img deployment_config.png %}

- 在Mapping 标签页填上本地路径和远程主机发布路径。
- 远程主机配置完成之后可以继续在Deployment 下的Options 选项中做一些个性化的配置，比如排除哪些文件，是否自动上传等。

# TODO
PyCharm 和其他jetBrain 的IDE 都有TODO 功能，迅速定位注释中出现TODO 关键字的位置，帮助我们快速回到上次工作遗留的地方。  

{% asset_img todo.png %}

TODO 窗口包含四个标签页，显示不同范围的TODO。 此外，TODO 功能还支持自定义模板。

[More about TODO](https://www.jetbrains.com/help/pycharm/using-todo.html)





[PyCharm 帮助文档](https://www.jetbrains.com/help/pycharm/meet-pycharm.html)