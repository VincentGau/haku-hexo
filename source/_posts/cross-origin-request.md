---
title: 如何解决跨域请求问题
tags:
  - cors
categories:
  - programming
date: 2018-12-22 19:16:53
---


## 介绍
**浏览器**安全策略禁止页面向其他域发送Ajax 请求，即同源策略，以防止恶意站点读取其他站点的敏感信息；

## 何为同源
只有协议，地址，端口均完全相同的URL才称为同源。
以下两个URL同源：
- `http://example.com/foo.html`
- `http://example.com/bar.html`
以下四个URL和上两个URL均不同源：
- `http://example.net` - Different domain
- `http://example.com:9000/foo.html` - Different port
- `https://example.com/foo.html` - Different scheme
- `http://www.example.com/foo.html` - Different subdomain

{% blockquote %}
注：IE在判断同源的时候不考虑端口差异；
{% endblockquote %}

## 解决跨域请求的几种方法
假设Web API地址为`http://localhost:9000/api/values/`,
请求发出页面地址为`http://localhost:8080`
如果直接在页面中发送ajax请求，如下
```javascript
$.ajax({
    type: "GET",
    url: http://localhost:9000/api/values/,
}).done(function (data) {
    alert(data);
}).error(function (jqXHR, textStatus, errorThrown) {
    alert(jqXHR.responseText || textStatus);
});
```
由于同源策略的存在，会发现无法获取资源，并能在浏览器的控制台发现如下错误：
{% asset_img cors-error-log.png %}
提示请求被cors策略限制，被请求的资源没有提供`Access-Control-Allow-Origin`头。
### JSONP
JSONP（JSON with padding）是一种通过注入`<script>`标签的方式请求数据的JavaScript模式，使得可以绕开同源策略限制共享数据。返回纯JSON数据的service由于同源策略的存在无法跨域共享数据，但是在`<script>`元素中可以执行从其他源获取的内容；于是可以在页面中增加一个src为所请求url的`<script>`元素，JSONP返回的数据不是JSON数据，而是一段script，以JSONP响应对象作为参数的回调函数，这就是为什么JSONP请求中会包含一个callback参数（有时是jsonp参数）；jQuery会自动去创建和插入script标签，只需要设置jsonp 作为dataType属性的值，即在ajax请求中增加一行`dataType:"jsonp",`:
```javascript
$.ajax({
    type: "GET",
    url: http://localhost:9000/api/values/,
    dataType:"jsonp",
}).done(function (data) {
    alert(data);
}).error(function (jqXHR, textStatus, errorThrown) {
    alert(jqXHR.responseText || textStatus);
});
```
JSONP的使用有一定局限性，并有潜在安全风险：
- 只支持GET方法；
- 存在跨站请求伪造风险；
- 在Chrome浏览器中可能会无法生效，

### CORS
Cross Origin Request Sharing(CORS) 是W3C标准，允许**服务器**端放松同源策略；通过CORS服务器可以显式地允许一些跨域请求，在response header增加`Access-Control-Allow-Origin`等属性。CORS在服务器端设置，浏览器端不需要其他配置；
以ASP.NET WebAPI为例，在服务端程序找那个启用CORS依赖`Microsoft.AspNet.WebApi.Cors`；在VS 中`Ctrl + Q` 快速启动搜索cors，在NuGet窗口中安装最新版本`Microsoft.AspNet.WebApi.Cors`；或者在Package Manager Console窗口通过执行命令：
{% codeblock %}
Install-Package Microsoft.AspNet.WebApi.Cors
{% endcodeblock %}
此命令安装最新版本的包并更新依赖，包括`System.Web.Cors` 和 `System.Web.Http.Cors`；

打开Startup.cs，增加`config.EnableCors();`
{% codeblock lang:CSharp %}
using System.Web.Http;
namespace APIs
{
    public class Startup
    {
        // This code configures Web API. The Startup class is specified as a type
        // parameter in the WebApp.Start method.
        public void Configuration(IAppBuilder appBuilder)
        {
            // Configure Web API for self-host. 
            HttpConfiguration config = new HttpConfiguration();

            config.EnableCors();
            
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );

            appBuilder.UseWebApi(config);
        }
    }
}
{% endcodeblock %}
然后在Controller类或者方法上增加`[EnableCors]`属性：
{% codeblock lang:CSharp %}
using System.Web.Http;
using System.Web.Http.Cors;

namespace APIs
{
    //[EnableCors(origins: "http://localhost:8080", headers: "*", methods: "*")]
    public class ValuesController : ApiController
    {
        // GET api/values
        [EnableCors(origins: "http://localhost:8080", headers: "*", methods: "*")]
        public IEnumerable<string> Get()
        {
            return new string[] { "value1", "value2" };
        }   
}
{% endcodeblock %}
此时在`http://localhost:8080`发出跨域资源请求即可，并可以在返回头中发现`Access-Control-Allow-Origin: http://localhost:8080` ，如下图所示：
{% asset_img response-header.png %}

### 请求转发
正如前文介绍，同源策略是**浏览器**安全策略，服务端与服务端的交互不受同源策略限制；
在无法控制请求资源所在的远端服务器，不能设置返回头`Access-Control-Allow-Origin`的情况下，可以通过请求转发间接获取资源，即在前端页面发送请求至同源的后端地址，由后端逻辑与目标服务器交互获取资源，再将数据返回给浏览器端；
{% codeblock lang:java %}
@RestController
@RequestMapping(value = "/forward")
public class ForwardController {

    @RequestMapping(value = "/getvalues", produces = "application/json")
    public String getValues()
    {
        final String uri = "http://localhost:9000/api/values/";

        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.getForObject(uri, String.class);
    }
}
{% endcodeblock %}

此时在前端页面发送请求至同源的`http://localhost:8080/forward/getvalues/`，由`getValues()`方法在服务器之间获取数据；
```javascript
$.ajax({
    type: "GET",
    url: http://localhost:8080/forward/getvalues/,
}).done(function (data) {
    alert(data);
}).error(function (jqXHR, textStatus, errorThrown) {
    alert(jqXHR.responseText || textStatus);
});
```

## 小结
相较于JSONP只支持HTTP GET方法，CORS更灵活也更安全，应尽量选择使用CORS而避免JSONP。

参考资料：
[JSONP](https://en.wikipedia.org/wiki/JSONP)
[Enable cross-origin requests](https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api)
