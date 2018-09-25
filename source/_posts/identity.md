---
title: ASP.NET Identity用户体系和授权认证
date: 2018-06-22 09:30:35
tags:
- asp.net
- WIF
- identity
- authentication
- authorization
---

大多数Web 应用都有用户的存在，有用户就必然涉及到授权和认证来控制用户行为，一些不允许匿名访问的资源（如webapi）也需要授权和认证，系统验证了用户身份的合法性之后用户才可以访问资源。这里分两步：认证（Authentication）验证用户的身份；授权（Authorization）验证用户是否有正确的权限获取指定的资源；授权、认证过程可以直接集成到单个应用程序中，对于由众多应用程序形成的生态系统，为其中每一个应用程序都开发自己的用户体系无疑是重复造轮子，而且会随之产生巨大的运维压力，用户信息的新增、撤销和修改需要在多处同时处理，若再有和外部系统对接的需求，则是一件更为复杂的事情，因此需要将用户授权认证模块从应用程序剥离，作为一个独立的模块供多个系统共用，基于声明（Claim-based）的联合授权认证是一个可行的方案。
# 基于声明的授权认证
基于声明的授权和认证并不是新的概念，在联合安全模型中认证和授权过程与应用程序本身分开，认证和授权是另外的单独的Web服务，即安全令牌服务（Security Token Service, STS），STS负责颁发安全令牌；信赖方应用程序（Relying Party，RP）不关心认证方如何认证，在认证方认证成功之后返回一个令牌，令牌中包含了信赖方需要的用户名、角色、权限等用来认证用户身份的信息。这种方式避免了针对一个用户需要管理多个副本的情况，密码同步的问题也不复存在，并且能满足单点登录（SSO）的场景，用户在登录一个应用程序之后无需再次进行身份认证即可获得访问另一个应用的权限。
基于声明的联合认证方式具有以下特点：
- 将身份验证机制从应用程序和服务中分离出来，实现与应用程序的松耦合；
- 使用声明（Claim）代替角色（Role），声明是一种更灵活、更精确的对象，它可以包括角色及其他信息；
- 安全令牌服务STS可以仅作为一个功能模块在用户管理系统中实现，并且依赖方应用程序可以很方便地与STS建立关系；

## 认证过程
在基于声明的应用场景中，通常包含三方参与者，分别是：应用程序自身，终端用户和STS。
{% asset_img wifbasicwebapp.gif %}
1. 用户访问依赖方应用程序，依赖方应用程序发现该未经认证的请求，于是重定向到STS 服务；
2. STS 服务需要用户提供凭证以验证身份，认证成功之后STS 会颁发一个令牌给用户；
3. 带着这个令牌，用户会被STS 服务重定向到依赖方应用程序；
4. 依赖方应用程序提前通过配置信任该STS 和它颁发的令牌，它会取出令牌中的声明信息，实现用户验证；  
  
也就是说，在依赖方与STS服务建立起信任关系之后，当用户请求依赖方应用的资源时，依赖方应用程序不关心登录过程，不关心用户角色，只需要把用户重定向到STS，由STS作验证和返回令牌，依赖方应用程序从令牌中获取需要的信息。

## 名词解释
**声明**是标识信息描述，如：用户名，部门，角色信息等。身份提供者验证成功后会返回一组声明，信赖方应用程序根据声明进行授权；建立基于声明的应用程序后，应用程序中的用户身份将通过一组声明来表示，一个声明可能是用户名，也可能是邮箱地址等等；  
**令牌**是声明传输的载体，是发行机构（STS）签名的一组序列化的声明；  
**STS**，安全令牌服务，一个负责认证用户并颁发令牌的服务，它把声明打包进令牌，对令牌进行加密；  
**依赖方应用程序**即依赖声明的应用程序，它将用户认证逻辑代理出去给STS，提取STS 颁发的令牌中的声明并将它们用于身份验证；  

## 应用场景
下面以一个具体场景，进一步理解基于声明的认证模型：
场景中涉及两个应用：一个是依赖方应用（`http://localhost:19851/`），需要验证用户身份；一个是用户管理应用，用户数据保存在这里，也在这里对用户信息进行增删改查，同时STS也包含在这个系统中（`http://localhost:62398/`）；  

1. 用户访问依赖方应用；
2. 如果已有登录凭据则直接进入访问对应资源；如果没有登录凭据，则跳转向STS服务请求安全令牌，并在请求中附带来源信息以便在登录之后重新跳转回去；
3. 用户在用户管理系统登录页面输入用户名和密码，登录成功后跳转回指定页面；  


- 请求依赖方地址`http://localhost:19851/`  
**request**：GET / HTTP/1.1   
**response**：HTTP/1.1 302 Found
Location: `http://localhost:62398/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`  
以上重定向地址的query string中：  
`wa` 取值`wsignin1.0`表示登录请求；  
`wct` 记录时间，判断请求是否超时；


- STS要求登录
**request**：GET `/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
**response**：HTTP/1.1 302 Found
Location: `/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`

- 请求登录页面
request：GET `/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
response：HTTP/1.1 200 OK
显示STS登录页面；

- 用户登录
request：POST `/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3a%2f%2flocalhost%3a19851%2f&wctx=rm%3d0%26id%3dpassive%26ru%3d%252F&wct=2018-09-25T06%3a25%3a12Z HTTP/1.1`

{% asset_img post-data.png %}

response：HTTP/1.1 302 Found
Location: `/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`

Set-Cookie: `.ASPXAUTH=D263346CD7291C49B8D8316FD7145BAD99D91D6AF0E8A768691987EACD7971437AFFF9D5AE3FE51BFE681EFE26A7957B69E7E2893F1A991ECE0C906D6E32D71290CC63DB5ED39074F65EB81D12949EC94710322AC420C6A383300AFFDC9D033F8DEC28D615EF7E6A9CF7CE9EF2CA91A9FFBFA234837378B59477EE3874788976; path=/; HttpOnly`

- 颁发Token
request：GET `/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
response：HTTP/1.1 200 OK
{% asset_img response-body.png %}
响应页面是包含一个自动提交的form，将token提交到依赖方地址；

- 自动提交表单
request：POST / HTTP/1.1 POST数据到http://localhost:19851/，带cookie

	`.ASPXAUTH=D263346CD7291C49B8D8316FD7145BAD99D91D6AF0E8A768691987EACD7971437AFFF9D5AE3FE51BFE681EFE26A7957B69E7E2893F1A991ECE0C906D6E32D71290CC63DB5ED39074F65EB81D12949EC94710322AC420C6A383300AFFDC9D033F8DEC28D615EF7E6A9CF7CE9EF2CA91A9FFBFA234837378B59477EE3874788976`

response：HTTP/1.1 302 Found
Location: /

Set-Cookie: `FedAuth=77u/PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48U2VjdXJpdHlDb250ZXh0VG9rZW4gcDE6SWQ9Il9hZjE2YzdmYi1iMzMxLTQ2MjgtYjQzMC0zZGRjNTkxNmEwMGUtMERFRkRGMEY5NDQ5MUZDRDZEQjcwMEQyQzNCNjM3NUUiIHhtbG5zOnAxPSJodHRwOi8vZG9jcy5vYXNpcy1vcGVuLm9yZy93c3MvMjAwNC8wMS9vYXNpcy0yMDA0MDEtd3NzLXdzc2VjdXJpdHktdXRpbGl0eS0xLjAueHNkIiB4bWxucz0iaHR0cDovL2RvY3Mub2FzaXMtb3Blbi5vcmcvd3Mtc3gvd3Mtc2VjdXJlY29udmVyc2F0aW9uLzIwMDUxMiI+PElkZW50aWZpZXI+dXJuOnV1aWQ6OGQ3NDhiYTItMTAwYi00MTI5LThlZTktZDM1ZTZmNmRiYWMxPC9JZGVudGlmaWVyPjxDb29raWUgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwNi8wNS9zZWN1cml0eSI+QVFBQUFOQ01uZDhCRmRFUmpIb0F3RS9DbCtzQkFBQUE1clRVeWN2Y3BreWRIeDdiQ1F2ODlBQUFBQUFDQUFBQUFBQVFaZ0FBQUFFQUFDQUFBQUQrUXY0TlFTMENMTjJUbGZIL0U5SzRYVjhMSXRFQzB5bUE3OHNLMnR1alZBQUFBQUFPZ0FBQUFBSUFBQ0FBQUFCVkt2emUvVE4wMUY4djhyVC94ZEtZWEdzNW4vbDU1M0JHaXRNTGRQYjg3ekFEQUFEUGN1dkQ0SUl6VjUyM3FSWldTQWJXcGFHby9UQ1l0RDhzc0dEd1pTbU9xSjJHdUZnS3p3cU1QTWJ6MjB1K2ZMbzNXK0U5WDNTaTRBc3dqczNjMllGZlJERk9FK1h5UHhWVTgrOUZmMXExMFRtdmlnUTVBQ3JBTXZwazdLMjlOdER2VnZLdEhUMVgxRjhkVFh5Wmh0Q2tDZ3EzalU3eVhNdlh3YVp3UkgwR0dSUWlsa1JKQzg5S2p4bjcrcGJkTFFSWUVUMnFwWFFxRUcrRVVBK01JZUhuR25LSnVxR2NEQ3dOT0xjZEhBbHdSRHRMZzhvemR2K1F3YnpIQXp5U0FwZEhoQTZrT21DTkJ0YkloNXptOVJCOWlxOVBUTDZGVHJQWFZpS3ppZWI2d2s0cy8vQjFlc0YxMkU5NG9XZVJ3dmQ4Z3l4WlhLek9XbktCUmpJMStoN3UyR0VZeE9DSG9KQWN2SWdtT092QnVXMFRtSk1wTlpnU1JBT3dIazZWWUV5SDM0dWhmYXB0d1NleUY1czFPZ1A4V1hxSUQrWTVJbkVSeXA2eWR6a2RXdGp5SlRmR0FUeEFzNGt4OFhicWMzLzlzUzlXRC81bWNYWEFsNzExQzVvQWhFUWhRdk03NTZya29Kb2c4ak9nR056MkZpbi9JbjJrZVFLRG1xY0VaRUJhT28ySnk5RXBKNWc2NjNRbk5RdG5HTm1weDRmU2RZRWxCcmgvNEdHSXFUaUxMTko1cmV1dm0yMVRXSStlL2xEUysveElhOWxuQTF2MFF3YTBYYkNlWC9oQUJGV0pWQTZ0Y1hQYmN0REQ4c2YyUThJN3hYN3laeld0YVRnOVZWQ0orTERFU0tzS2MzSUVKUE1EWlBlRWNkcm9sWXA0RzE0UVJidHIzeGFJQno4Y0hEUlpwVUFiTllJL2o4UkhZNXAxWWdseDRmaU81cmVEamwyY2tNamlOUWZ5RDI1L0VqOEVsUGFDSHgzVklYbW84RU1ndHgzQklyL1RmQnpDcnB6M0FCVWg0a00rN1ZzSU5LNTFKNzA4S0RzYVFNZDByaTFCZzAycWlQVXhXSkNtZlRaNWFuNmNLTEp2YXZvY0tuUTJTR3VFSGVjQ3dzUTdZVlprdWFha1cxRS9HZTVxM1N1; path=/; HttpOnly`

Set-Cookie: `FedAuth1=RjVnTXZjVlFFdlhHU1luck9VWnZjL25hL0VCVjJERDRoL2tVUDZ0cUNYTkR1WG9mMkFCT091NGEwWlZ3QzNHeWV5UXY2WGdLWGxiVE5Vd1lqdVJTS0wrWDJzNDczcmdBcCs0dldpWXhFMm9WSG5SWHFXTFVCcS9UT3BQd1ZqVmxVN09ORVlUQXdUZmptMFFYZlNnLzFGVnVUV1Rpai9FMk91Wi9nY0NSQ2k2WS8vZ1B6TVBUdDliSkJHcmFrVnVObmtIZTV4VkEzUjVqcG9lSm1PSkIwWnFIb2NSQ2pRWXBBQUFBQWQ5QXBiOGtjZzZQK1BvNG9oWmYyeisrQ1NVelJOb2hXbG55N1EwSFNZZUpWV2xKM3d2bjBlazlMMkF1NVNPTnZ1UzVRT3ZvbXdsVEhFdEhXTEQ4WC9nPT08L0Nvb2tpZT48L1NlY3VyaXR5Q29udGV4dFRva2VuPg==; path=/; HttpOnly`

- 进入依赖方地址
request：GET / HTTP/1.1 请求中附带cookie
response：HTTP/1.1 200 OK



在基于角色的访问控制（Role-based Access Control，RBAC）中，用户权限通过一个基于角色的应用程序来管理和执行，如果用户拥有执行一个动作需要的角色，则该动作被允许；和基于角色的模型相比，Claim-based的方式不与角色捆绑，具有更好的灵活性和扩展性，声明并不局限于角色和权限，还能附带用户的其他信息，比如邮箱、生日，比如还可以通过isOver18 声明在不透露用户具体年龄的情况下验证用户是否有权限等等，授权的决定基于声明中的有效数据的任意逻辑，而在RBAC中，唯一使用的声明就是角色。

## Claims认证与OAuth
OAuth是用于授权的工业标准协议，基于声明的认证与OAuth有不同的适用场景：
Claim-based：
- 用于应用程序和用户身份认证解耦；
- 事前约定建立信任关系，用户声明由身份管理员维护；
- 安全令牌服务将声明包含在令牌中颁发并提交给应用；
OAuth：
- 用户可选授权；
- 用于与第三方合作，应用本身有自己的用户体系；
- 获取访问令牌去资源服务器获取；

# 基于声明的Web应用的实现
## WIF
Windows Identity Framework（WIF）是一组.Net Framework 类库，实现身份感知的（Identity-aware）、基于声明（Claim-based）的应用程序和服务，WIF 原本作为独立下载的类库发布，现已经集成到.Net 4.5 中。有了WIF，我们可以自定义安全令牌服务，更容易地开发基于声明的Web 应用而无需再安装其他组件。

Identity 是微软在ASP.NET 应用程序中管理用户的一个API。



## 建立自定义STS

# 小结

应用场景 流程
