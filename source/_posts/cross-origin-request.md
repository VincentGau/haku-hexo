---
title: 如何解决跨域请求问题
tags:
  - cors
categories:
  - programming
date: 2018-12-22 19:16:53
---


## 介绍
同源策略是**浏览器端**web应用安全的一个重要概念，在这个策略下，浏览器禁止页面中的脚本向其他域发送请求（如Ajax），以防止恶意站点读取其他站点的敏感信息；

## 场景分析
- 假设有一个Web API地址为`http://localhost:9000/api/values/`  
该API返回一段简单的json数据，如下：
{% asset_img api-response.png %}
- 有一个Web页面地址为`http://localhost:8080`
页面中仅包含一段JavaScript脚本，请求上述api并在控制台显示返回结果，如果出错则显示错误信息，script内容如下：
```html
<script type="text/javascript">
    $.ajax({
        type: "GET",
        url: "http://localhost:9000/api/values/",
    }).done(function (data) {
        console.log(data);
    }).error(function (jqXHR, textStatus, errorThrown) {
        console.log(jqXHR.responseText || textStatus);
    });
</script>
```
当我们打开页面，发现在控制台出现如下错误：
{% asset_img cors-error-log.png %}
提示请求被cors策略限制，被请求的资源没有提供`Access-Control-Allow-Origin`头；出现此错误是由于同源策略的存在。

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

`http://localhost:9000/api/values/` 与 `http://localhost:8080` 不同源，因此从8080端口请求得到的JavaScript脚本中无法向9000端口提供的API发送请求。

## 解决跨域请求的几种方法
### JSONP
在介绍JSONP之前，首先明确一点，即html可以通过script标签从不同的域获取JavaScript脚本，体现为在一个html页面中，可以从abc.com引入script1.js，同时从xyz.com引入script2.js，如下：
```html
<script type="text/javascript" src="http://abc.com/jquery.min.js"></script>
<script type="text/javascript" src="http://xyz.com/bootstrap.min.js"></script>
```
实际上，对于支持src属性的标签，都是可以从不同的域引入资源的，比如通过img标签从不同的域获取图片资源等。在此前提下，是否可以将json数据放在JavaScript脚本中被获取呢？考虑一个特定场景，假设html中仅包含一段JavaScript脚本，如下所示，脚本中定义并调用了一个方法mycallback：
```html
<!DOCTYPE html>
<html>
<head>
	<title>JSONP</title>
</head>
<body>
	<script type="text/javascript">
		let mycallback = function(data){
			console.log(data);
		};

		mycallback("Hello Haku.");
	</script>
</body>
</html>
```
当打开页面，会在控制台看到`Hello Haku.`的输出；
当然也可以把调用`mycallback`的脚本放到单独的data.js文件，调整为如下：
```html
<!DOCTYPE html>
<html>
<head>
	<title>JSONP</title>
</head>
<body>
	<script type="text/javascript">
		let mycallback = function(data){
			console.log(data);
		}
	</script>
	<script type="text/javascript" src="data.js"></script>
</body>
</html>
```
data.js仅包含调用mycallback语句：
```javascript
mycallback([{"name": "haku"}, {"name": "chihiro"}])
```
当刷新页面，会在控制台看到`[{"name": "haku"}, {"name": "chihiro"}]`的输出；

JSONP（JSON with padding）是一种通过注入`<script>`标签的方式请求数据的JavaScript模式，使得可以绕开同源策略限制共享数据，此处padding实际上是指一个回调函数（如上例中的mycallback）。由于同源策略的存在返回纯JSON数据的service无法跨域共享数据，但是在`<script>`元素中可以执行从其他源获取的内容；于是可以在页面中增加一个src为所请求url的`<script>`元素，JSONP返回的数据不是JSON数据，而是一段script，以JSONP响应对象作为参数的回调函数，这就是为什么JSONP请求中会包含一个callback参数。若需要服务端返回一段JavaScript脚本而不是json格式数据，需要修改服务端代码，以C#为例，为了方便可以直接引入第三方库`WebApiContrib.Formatting.Jsonp`，在api的配置函数中定义Fomatter，返回一段被回调函数包裹的脚本，代码如下
```asp
var jsonpFormatter = new JsonpMediaTypeFormatter(config.Formatters.JsonFormatter);
config.Formatters.Insert(0, jsonpFormatter); 
```
然后将前端页面修改为：
```html
<!DOCTYPE html>
<html>
<head>
	<title>JSONP</title>
</head>
<body>
	<script type="text/javascript">
		let mycallback = function(data){
			console.log(data);
		}
	</script>
	<script type="text/javascript" src="http://localhost:9000/api/values?callback=mycallback"></script>
</body>
</html>
```
此时刷新页面，会在控制台看到`[{"name":"abc","value":"cba"},{"name":"xyz","value":"zyx"}]`的输出，并且可以发现返回头中显示的返回类型为JavaScript而不是json，返回的内容如下所示：
{% asset_img response_js.png %}

如果引入了JQuery，则不必手动创建`script`标签，jQuery会自动创建和插入，只需要在ajax请求找那个设置`jsonp` 作为`dataType`属性的值，将前端页面调整为如下所示，即我们常见的jsonp使用方法：
```html
<!DOCTYPE html>
<html>
<head>
	<title>JSONP</title>
</head>
<body>
	<script type="text/javascript">        
        $.ajax({
            type: "GET",
            url: "http://localhost:9000/api/values",
            dataType:"jsonp",
        }).done(function (data) {
            console.log(data);
        }).error(function (jqXHR, textStatus, errorThrown) {
            console.log(jqXHR.responseText || textStatus);
        });        
    </script>
</body>
</html>
```
此时刷新页面，会在控制台看到`[{"name":"abc","value":"cba"},{"name":"xyz","value":"zyx"}]`的输出。

JSONP的使用有一定局限性，并有潜在安全风险：
- 只支持GET方法；
- 存在跨站请求伪造风险；
- 在Chrome浏览器中可能会无法生效，

### CORS
Cross Origin Request Sharing(CORS) 是W3C标准，允许**服务器**端放松同源策略；通过CORS服务器可以显式地允许一些跨域请求，体现为在response header增加`Access-Control-Allow-Origin`等属性。CORS在服务器端设置，浏览器端不需要其他修改。
以ASP.NET WebAPI为例，引入`Microsoft.AspNet.WebApi.Cors`库；在VS 中`Ctrl + Q` 快速启动搜索cors，在NuGet窗口中安装最新版本`Microsoft.AspNet.WebApi.Cors`；或者在Package Manager Console窗口通过执行命令：
{% codeblock %}
Install-Package Microsoft.AspNet.WebApi.Cors
{% endcodeblock %}
此命令安装最新版本的包并更新依赖，包括`System.Web.Cors` 和 `System.Web.Http.Cors`；

在Web API的配置函数中，增加`config.EnableCors();`以启用CORS，并在Controller类或者方法上增加`[EnableCors]`特性，如下：
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
        public IEnumerable<object> Get()
                {
                    var resultList = new List<object>();
                    resultList.Add(new { name = "abc", value = "cba" });
                    resultList.Add(new { name = "xyz", value = "zyx" });
                    return resultList;
                }
}
{% endcodeblock %}
前端页面的ajax请求中一处dataType，调整回：
```html
<!DOCTYPE html>
<html>
<head>
	<title>JSONP</title>
</head>
<body>
	<script type="text/javascript">        
        $.ajax({
            type: "GET",
            url: "http://localhost:9000/api/values",
        }).done(function (data) {
            console.log(data);
        }).error(function (jqXHR, textStatus, errorThrown) {
            console.log(jqXHR.responseText || textStatus);
        });        
    </script>
</body>
</html>
```
此时刷新页面，会在控制台看到`[{"name":"abc","value":"cba"},{"name":"xyz","value":"zyx"}]`的输出，并且响应头中显示返回类型为json，以及`Access-Control-Allow-Origin: http://localhost:8080` ，如下图所示：
{% asset_img response-header.png %}

CORS 支持在Action级别，Controller级别以及全局级别设置；如需在全局应用CORS策略，只需向`EnableCors`方法传递一个`EnableCorsAttribute`实例，如下：
{% codeblock lang:CSharp %}
var cors = new EnableCorsAttribute("http://localhost:8080", "*", "*");
config.EnableCors();
{% endcodeblock %}

### 请求转发
正如前文介绍，同源策略是**浏览器**安全策略，服务端与服务端的交互不受同源策略限制；
在无法控制请求资源所在的远端服务器，不能设置返回头`Access-Control-Allow-Origin`的情况下，可以通过请求转发间接获取资源，即在前端页面发送请求至同源的后端地址，由同源后端与目标服务器交互获取资源，再将数据返回给浏览器端；前端页面所在的web应用中新增一个接口，如下：
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

WebAPI应用一处JSONP和CORS配置，将前端页面调整如下，发送请求至同源的`http://localhost:8080/forward/getvalues/`，由`getValues()`方法在服务器之间获取数据：
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
此时刷新页面，会在控制台看到`[{"name":"abc","value":"cba"},{"name":"xyz","value":"zyx"}]`的输出。

## 小结
JSONP方式的本质是通过下载JavaScript脚本的方式获取数据，因此只支持HTTP GET方法，CORS更灵活也更安全，应尽量选择使用CORS而避免JSONP。

参考资料：
[JSONP](https://en.wikipedia.org/wiki/JSONP)
[Enable cross-origin requests](https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api)
