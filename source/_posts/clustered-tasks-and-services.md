---
title: 任务计划与windows服务的集群化
tags:
  - scheduled task
  - Windows service
  - clustered
date: 2017-12-26 17:30:53
---

我们常常会使用任务计划定时执行一些日常任务，若需要程序长时间执行又不需要用户登录，可能还会用到Windows Service，一般情况单台服务器就能满足要求，但对一些要求比较高的任务可能需要集群化来保证高可用性。在故障转移集群上实现任务计划的集群化和Windows Service的集群化有一些差异。

<!-- more -->

### 任务计划集群化
Windows Server 2012 之前，我们可以在集群的节点上部署任务，但是任务计划程序对整个集群是不感知的，手动管理各个节点上的任务计划比较容易出错，尤其是在节点多的情况下，Windows Server 2012 以后，通过Cluster Scheduled Tasks 可以十分简单地通过PowerShell 来管理集群上的任务计划，只需要在任意一个节点上注册任务。比较常用的是AnyNode 模式，任务计划在任意一个节点上启用，在其他节点都处于禁用状态。

对于已存在的任务计划，可以先导出xml，以该文件名作为参数进行任务注册，这样就不用单独配置触发器了：
```bash
Register-ClusteredScheduledTask -TaskName MyTask -Cluster MyCluster -Xml $xmlFile 
```
> 注意： 使用-Xml参数的时候需要保证任务计划用户在各个节点上都存在

详见 [How to Configure Clustered Tasks with Windows Server 2012](https://blogs.msdn.microsoft.com/clustering/2012/05/31/how-to-configure-clustered-tasks-with-windows-server-2012/)

### Windows service集群化
Windows Service 不能感知到整个集群，因此Windows Service集群化需要首先在各个节点上安装服务，然后再在故障转移集群管理器中添加服务。

参考 [Creating a Windows Cluster: Part 5 – Adding Applications and Services to the Cluster](https://www.1e.com/blogs/2014/11/17/creating-a-windows-cluster-part-5-adding-applications-and-services-to-the-cluster/)