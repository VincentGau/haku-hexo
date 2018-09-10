---
title: Identity用户体系和授权认证
date: 2018-06-22 09:30:35
tags:
- asp.net
- WIF
- identity
- authentication
- authorization
---

大多数Web 应用都有用户的存在，有用户就必然涉及到授权和认证来控制用户行为，一些不允许匿名访问的资源（如webapi）也需要授权和认证，系统验证了用户身份的合法性之后用户才可以访问资源。这里分两步：认证（Authentication）验证用户的身份；授权（Authorization）验证用户是否有正确的权限获取指定的资源；认证在先，授权在后。授权、认证过程可以直接集成到单个应用程序中，对于由众多应用程序形成的生态系统，为其中每一个应用程序基于声明的是一个可行的方案。
# 基于声明的授权
在联合安全模型中，认证通过Security Token Service（STS）实现，STS颁发安全令牌
Windows Identity Framework（WIF）是一组.Net Framework 类库，实现身份感知的（Identity-aware）、基于声明（Claim-based）的应用程序和服务，WIF 原本是作为独立下载的类库发布的，现已经集成到.Net 4.5 中，
Identity 是微软在ASP.NET 应用程序中管理用户的一个API。


# 如何建立STS

应用场景 流程
