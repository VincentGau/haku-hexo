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

大多数Web 应用都有用户的存在，有用户必然涉及到授权和认证来控制用户行为，除了页面资源以外，访问其他不允许匿名访问的资源（如webapi）也需要经过授权和认证，系统验证了用户身份的合法性之后用户才可以继续访问。这里分两步：授权和认证，认证（Authentication）验证用户的身份；授权（Authorization）决定用户是否有正确的权限获取指定的资源；授权、认证过程可以直接集成到单个应用程序中，对于由众多应用程序形成的生态系统，为其中每一个应用程序都开发自己的用户体系无疑是重复造轮子，而且会随之产生巨大的运维压力，用户信息的新增、撤销和修改需要在多处同时处理，若再有和外部系统对接的需求，则会使事情更加复杂，因此需要将用户授权认证模块从应用程序剥离，作为一个独立的模块供多个系统共用，基于声明（Claim-based）的联合授权认证是一个可行的方案。
# 基于声明的授权认证
基于声明的授权和认证并不是全新的概念，在联合安全模型中，认证和授权过程与应用程序本身分开，认证和授权是另外的单独的Web服务，即安全令牌服务（Security Token Service, STS），STS负责颁发安全令牌；信赖方应用程序（Relying Party，RP）不进行用户认证，不再保存用户名和密码，不关心认证方如何认证，在认证方认证成功之后返回一个令牌，令牌中包含了信赖方需要的用户名、角色、权限等用来认证用户身份的信息。这种方式避免了针对一个用户需要管理多个副本的情况，密码同步的问题也不复存在，并且能满足单点登录（SSO）的场景，用户在登录一个应用程序之后无需再次进行身份认证即可获得访问另一个应用的权限。在基于声明的应用程序中，用户身份将通过一组声明来表示，一个声明可能是用户名，也可能是邮箱地址、年龄等等。
基于声明的联合认证方式具有以下特点：
- 将身份验证机制从应用程序和服务中分离出来，实现与应用程序的松耦合；
- 使用声明（Claim）代替角色（Role），声明是一种更灵活、更精确的对象，它可以包括角色及其他信息；
- 安全令牌服务STS可以仅作为一个功能模块在用户管理系统中实现，并且依赖方应用程序可以很方便地与STS建立关系；

## 认证过程
在基于声明的应用场景中，通常包含三方参与者，分别是：应用程序自身，终端用户和STS。
{% asset_img wifbasicwebapp.gif %}
1. 用户访问依赖方应用程序，依赖方应用程序发现该未经认证的请求，于是重定向到STS 服务；
2. STS 服务需要用户提供凭证以验证身份，认证成功之后STS 会颁发一个令牌给用户；
3. 带着这个令牌，用户请求会被STS 服务重定向到依赖方应用程序；
4. 依赖方应用程序提前通过配置信任该STS 和它颁发的令牌，它会取出令牌中的声明信息，实现用户验证，之后会生成一个cookie，以便下次访问；  
  
也就是说，在依赖方与STS服务建立起信任关系之后，当用户请求依赖方应用的资源时，依赖方应用程序不关心登录过程，不关心用户角色，只需要把用户重定向到STS，由STS作验证和返回令牌，依赖方应用程序从令牌中获取需要的信息。

## 名词解释
**声明**是标识信息描述，如：用户名，部门，角色信息等。身份提供者验证成功后会返回一组声明，信赖方应用程序根据声明进行授权；应用程序接收到的声明越多，对用户的了解就越详细；应用程序并不会去某一个路径查询用户的声明，而是用户发送声明给应用程序，应用程序会检测这些声明，通过Issuer 字段表示知道声明的颁发者，应用程序只会接受受信任的颁发者生成的声明。
**令牌**是声明传输的载体，是发行机构（STS）签名的一组序列化的声明；用户会将一组声明和请求一起通过POST 方式发送给应用程序，在安全性要求不高的场景中也可以使用不签名的声明；  
**STS**，安全令牌服务，一个负责认证用户并颁发令牌的服务，它把声明打包进令牌，对令牌进行签名和加密；  
**依赖方应用程序**即依赖声明的应用程序，它将用户认证逻辑代理出去给STS，提取STS 颁发的令牌中的声明并将它们用于身份验证；  

## 应用场景
下面以一个具体场景为例，分析访问过程中的具体HTTP请求进一步理解基于声明的认证：
场景中涉及两个应用：一个是依赖方应用（例子中地址为 `http://localhost:19851/`），需要验证用户身份；一个是用户管理应用，用户数据保存在这里，也在这里对用户信息进行增删改查，同时STS也包含在这个系统中（例子中地址为 `http://localhost:62398/`）；  

{% asset_img process.png %}

1. 用户访问依赖方应用；
2. 如果已有登录凭据则直接进入访问对应资源；如果没有登录凭据，则跳转向STS服务请求安全令牌，并在请求中附带来源信息以便在登录之后重新跳转回去；
3. 用户被重定向到Identity用户管理的登录页面；
4. 用户登录成功后跳转回指定页面，生成Token；  
5. 经过权限验证之后用户被重定向至最初的依赖方页面；

这个过程中发生的HTTP请求有：

- 请求依赖方地址`http://localhost:19851/`：  
**request**：GET / HTTP/1.1   
**response**：HTTP/1.1 302 Found
Location:  
`http://localhost:62398/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`  
_请求依赖方地址，被重定向到STS_。以上重定向地址的query string中：  
`wa` 取值`wsignin1.0`表示登录请求，`wsignout1.0`表示登出请求；  
`wtrealm` 表示依赖方应用的URI， STS通过它来决定是否颁发令牌以及给予哪些声明；
`wct` 记录请求时间，STS用于校验判断请求是否超时；  
`wtreply` 可选项，表示依赖方希望被重定向到的地址；

  
- 请求STS`http://localhost:62398/`，被重定向到登录页面：  
**request**：GET  
`/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
**response**：HTTP/1.1 302 Found
Location:  
`/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`  
由于用户没有登录，被重定向到STS的地址后再次被重定向到登录页面；

- 请求登录页面`http://localhost:62398/`：
**request**：GET  
`/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
**response**：HTTP/1.1 200 OK
显示STS登录页面，等待用户输入登录信息；

- 用户登录`http://localhost:62398/`：
**request**：POST  
`/login.aspx?ReturnUrl=%2f%3fwa%3dwsignin1.0%26wtrealm%3dhttp%253A%252F%252Flocalhost%253A19851%252F%26wctx%3drm%253D0%2526id%253Dpassive%2526ru%253D%25252F%26wct%3d2018-09-25T06%253A25%253A12Z&wa=wsignin1.0&wtrealm=http%3a%2f%2flocalhost%3a19851%2f&wctx=rm%3d0%26id%3dpassive%26ru%3d%252F&wct=2018-09-25T06%3a25%3a12Z HTTP/1.1`
{% asset_img post-data.png %}
**response**：HTTP/1.1 302 Found
Location:  
`/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z`
Set-Cookie: `.ASPXAUTH=blablabla; path=/; HttpOnly`
_用户POST表单数据之后，被重定向回STS；_

- 请求STS地址`http://localhost:62398/`：
**request**：GET 
`/?wa=wsignin1.0&wtrealm=http%3A%2F%2Flocalhost%3A19851%2F&wctx=rm%3D0%26id%3Dpassive%26ru%3D%252F&wct=2018-09-25T06%3A25%3A12Z HTTP/1.1`
**response**：HTTP/1.1 200 OK  
{% asset_img response-body.png %}
带cookie请求STS地址，响应页面包含一个自动提交的form；

- 自动提交表单`http://localhost:19851/`
**request**：POST / HTTP/1.1 
	`.ASPXAUTH=blablabla`
**response**：HTTP/1.1 302 Found
Location: /
Set-Cookie: `FedAuth=blablabla; path=/; HttpOnly`  
Set-Cookie: `FedAuth1=blablabla; path=/; HttpOnly`  
_自动提交表单，将token发送到依赖方地址；_


- 访问依赖方地址`http://localhost:19851/`：
**request**：GET / HTTP/1.1 
**response**：HTTP/1.1 200 OK
_依赖方消费Token，用户最终访问到依赖方应用；_

## 基于声明和基于角色的区别
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

## 依赖方应用配置
依赖方Web应用程序如需使用已有的自定义STS进行身份认证，需要给项目添加引用并修改配置文件：
需要添加的引用包括`System.IdentityModel` 和`System.IdentityModel.Services`；  
需要编辑配置文件web.config，使用基于声明的认证方式：
```xml
<system.webServer>
    <modules>
      <add name="WSFederationAuthenticationModule" type="System.IdentityModel.Services.WSFederationAuthenticationModule, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" preCondition="managedHandler" />
      <add name="SessionAuthenticationModule" type="System.IdentityModel.Services.SessionAuthenticationModule, System.IdentityModel.Services, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" preCondition="managedHandler" />
    </modules>
</system.webServer>
```

配置URL地址与处理程序：
```xml
<system.identityModel>
    <identityConfiguration saveBootstrapContext="true">
      <issuerTokenResolver type="SimpleWebToken.CustomIssuerTokenResolver, SimpleWebToken">
        <AddAudienceKeyPair  symmetricKey="wAVkldQiFypTQ+kdNdGWCYCHRcee8XmXxOvgmak8vSY=" audience="http://localhost:19851/" />
      </issuerTokenResolver>
      <issuerNameRegistry type="RelyingParty.TrustedIssuerNameRegistry, RelyingParty"/>
      <audienceUris>
        <add value="http://localhost:19851/"/>
      </audienceUris>
      <securityTokenHandlers>
        <add type="SimpleWebToken.SimpleWebTokenHandler, SimpleWebToken" />
      </securityTokenHandlers>
    </identityConfiguration>
</system.identityModel>
```

```xml
<system.identityModel.services>
<federationConfiguration identityConfigurationName="">
  <serviceCertificate>
    <certificateReference x509FindType="FindBySubjectName" findValue="localhost" storeLocation="LocalMachine" storeName="My"/>
  </serviceCertificate>
  <wsFederation passiveRedirectEnabled="true" issuer="http://localhost:62398/" realm ="http://localhost:19851/" requireHttps="false" />
  <cookieHandler mode="Default" requireSsl="false">
    <chunkedCookieHandler chunkSize="2000"/>
  </cookieHandler>
</federationConfiguration>
</system.identityModel.services>
```
上述配置中：
`audienceUris` 标签指定依赖方应用的URL;
`securityTokenHandlers` 标签指定处理token 的类和方法，可以使用自定义的方法或内建的方法；
`issuerNameRegistry`标签指定token 处理程序使用的颁发者；
配置完成后，访问依赖方应用需要身份认证的页面时会跳转至STS进行登录操作。

# 小结
用户授权认证是大多数应用程序都需要经过的流程，目前各种框架种类众多，基于声明的模型中用户身份由一组声明表示，通过配置一个受信任的外部身份系统为我们自己的应用程序提供关于用户的所有必要信息，在这种模型下，单点登录也能较为简单的实现，应用程序本身不处理任何用户认证相关的逻辑，并且不需要保存用户账户和密码等数据，也不用主动查询用户的详细信息，只需从受信任的令牌发布程序接受安全令牌即可。一个新的应用依赖自定义的STS进行身份认证和授权只需要简单的配置，用户管理方便，安全性有保障，对于不适合使用我行现有的统一安全认证平台的单个或多个应用，ASP.NET 基于声明的模型是一个可选的方案。