---
title: 搭建服务器亲自亲自体验跨域
date: 2019-01-18 19:07:13
tags:
- 跨域
categories:
-前端
---

## 背景
跨域这两个字就像狗皮膏药一样儿粘在每一个前端er身上 我遇见了很多开发者一般都是为了应付面试 随便背几个方案 知道概念 但是不知道为什么要这么干
到了真正的工作 开发环境有 `webpack-dev-server `搞定 线上有运维大哥会配好，配什么我不管 反正不会跨域就是了
但是.. 这样儿混日子 你的良心不会痛吗？

痛定思痛 决心不定时更新 不要再问我 XX 的问题系列 之 <font color="red">不要再问我跨域的问题了</font>

其实团队的小伙伴分享过类似的 但是不动手试一下 跟你面试前的死记硬背本质上没有任何区别

## 你需要了解的几个概念
* 什么是跨域？

**官方解释**
跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域、协议或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。

比如，站点 http://domain-a.com 的某 HTML 页面通过 <img> 的 src 请求 http://domain-b.com/image.jpg。网络上的许多页面都会加载来自不同域的CSS样式表，图像和脚本等资源。



* 为什么会产生跨域？

<font color=red>出于安全原因，浏览器限制从脚本内发起的跨源HTTP请求（也可能跨站请求可以正常发起，但是返回结果被浏览器拦截了）</font>

跨域的产生来源于现代浏览器所通用的`同源策略`，所谓同源是指`"协议+域名+端口"`三者相同的情况下，才允许访问相同的`cookie`、`localStorage`或是发送`Ajax`请求等等

**常见的跨域场景**

```
URL                                      说明                    是否允许通信
http://www.domain.com/a.js
http://www.domain.com/b.js         同一域名，不同文件或路径           允许
http://www.domain.com/lab/c.js

http://www.domain.com:8000/a.js
http://www.domain.com/b.js         同一域名，不同端口                不允许
 
http://www.domain.com/a.js
https://www.domain.com/b.js        同一域名，不同协议                不允许
 
http://www.domain.com/a.js
http://192.168.4.12/b.js           域名和域名对应相同ip              不允许
 
http://www.domain.com/a.js
http://x.domain.com/b.js           主域相同，子域不同                不允许
http://domain.com/c.js
 
http://www.domain1.com/a.js
http://www.domain2.com/b.js        不同域名                         不允许
```

* 现代的跨域解决方案

1. 通过jsonp跨域
2. document.domain + iframe跨域
3. location.hash + iframe
4. window.name + iframe跨域
5. postMessage跨域
6. <font color=red>跨域资源共享（CORS）</font>
7. nginx代理跨域
8. nodejs中间件代理跨域
9. WebSocket协议跨域

## 搭建服务尝试还原跨域过程

**通过koa搭建两个本地server 两个server都定义了一个GET请求接口/ajax。除监听port不同外，app.js还设置了静态服务。**



![koa1](/images/cors/koa1.png)

**app.js port:8000**

```
const Koa = require('koa');
const app = new Koa();
const index = require('./routes/index')
const views = require('koa-views')
const serve = require('koa-static');
const path = require('path');

// 引入静态资源
const staticPath = path.resolve(__dirname, '/public');

// 设置静态服务
const staticServe = serve(staticPath, {
  setHeaders: (res, path, stats) => {
    if (path.indexOf('jpg') > -1) {
      res.setHeader('Cache-Control', ['private', 'max-age=60']);
    }
  }
});

app.use(staticServe);


// 增加模版引擎 默认直接渲染html文件
app.use(views(__dirname + '/views'));
// 引入路由配置文件
app.use(index.routes(), index.allowedMethods())


router.get('/ajax', async (ctx, next) => {
  console.log('get request', ctx.request.header.referer);
  ctx.body = 'received';
});

app.listen(8000,()=>{
     console.log('app1 server is listening port 8000');
});
console.log('demo in run.....')

// route.js

 router.get('/ajax', async (ctx, next) => {
    console.log('get request', ctx.request.header.referer);
    ctx.body = 'received';
  });
  
```
**app2.js port:3000**


```
const koa = require('koa');
const app = new koa();
const app2Route = require('./routes/app2Route')
const cors = require('koa2-cors');

app.use(app2Route.routes(), app2Route.allowedMethods())


const main = async function(ctx,next) {
    ctx.response.body = '3000端口';
await next();
}

app.use(main)

app.listen(3000);
console.log('app2 server is listening port 3000');

// route.js

 router.get('/ajax', async (ctx, next) => {
    console.log('get request', ctx.request.header.referer);
    ctx.body = 'received';
  });
```
**前端模版**


```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>cross-origin test</title>
</head>
<body style="width: 600px; margin: 200px auto; text-align: center">
  <button onclick="getAjax()">GET 简单请求</button>
  <button onclick="getJsonP()">JSONP</button>
  <button onclick="corsWithJson()">POST 非简单请求</button>
</body>
<script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
<script type="text/javascript">

  var baseUrl = 'http://localhost:3000';
function getAjax() {
    var xhr = new XMLHttpRequest();            
    xhr.open('GET',  baseUrl + '/ajax', true);
    xhr.onreadystatechange = function() {
      // readyState == 4说明请求已完成
      if (xhr.readyState == 4 && xhr.status == 200 || xhr.status == 304) { 
        // 从服务器获得数据  
        alert(xhr.responseText);
      } else {
        console.log(xhr.status);
      }
    };
    xhr.send();
  }
</script>
</html>

```

很简单 大概长这样儿

![koa2](media/15478002097574/koa2.png)

**AJAX**

测试case

1. 同域下请求ajax 不涉及跨域
  
 请求接口：`baseUrl = 'http://localhost:8000';`
 测试结果👇
 ![koa3](media/15478002097574/koa3.png)


2. 跨域ajax请求
请求接口：`baseUrl = 'http://localhost:3000';`
测试结果👇
![koa4](media/15478002097574/koa4.png)
很明显 跨域了

### 针对浏览器的Ajax请求跨域的主要解决方案有：JSONP、CORS。


* **JSONP**

    <font color="red">原理</font>
    
    虽然浏览器同源策略限制了XMLHttpRequest请求不同域上的数据。但是，在页面上引入不同域的js脚本是可以的，而且script元素请求的脚本会被浏览器直接运行
    
    <font color="red">测试</font>
    
    `origin.html`添加
    
```
  function getJsonP() {
    var script = document.createElement('script');
    script.src = baseUrl + '/jsonp?type=json&callback=onBack';
    document.head.appendChild(script);
}

function onBack(res) {
    alert('JSONP CALLBACK:  ' + JSON.stringify(res) + ''); 
}
```
getJsonP方法会在当前页面添加一个script，src属性指向跨域的GET请求
通过query格式带上请求的参数。callback是关键，用于定义跨域请求回调的函数名称，这个值必须后台和脚本保持一致

在`app2.js`添加路由

```
router.get('/jsonp', async (ctx, next) => {
  const req = ctx.request.query;
  console.log(req);
  const data = {
    data: req.type
  }
  ctx.body = req.callback + '('+ JSON.stringify(data) +')';
})

app.use(router.routes());

```
针对jsonp请求，后台要做的是：

获取请求参数中的callback值，如本例中的onBack
将callback的值以function(args)的格式作为response。

重启服务 触发页面的 `JSONP`🔘
![koa5](media/15478002097574/koa5.png)


<font color="red">优点</font>
JSONP方案的兼容性好，IE浏览器也支持。

<font color="red">缺点</font>
  
```
 因为是利用的<script>元素，所以只支持GET请求。
 缺乏错误处理机制
```

* CORS

CORS即跨域资源分享，是W3C制定的标准。

1. 特性
CORS需要浏览器和服务器同时支持。

```
大多主流浏览器都支持，IE 10以下不支持。
只要服务器端实现了CORS接口，浏览器就能自动实现基于CORS的跨域请求。
```

2. 两种请求

浏览器将CORS请求分成两类：简单请求和非简单请求。

* 简单请求
满足条件：请求类型为 <font color="red">`HEAD，GET，POST之一`</font>；
请求头信息不超出以下几种：

```
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
```
对于简单请求，浏览器会直接发出，同时在请求头中添加Origin字段。

Origin用来说明请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。
如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

回顾下直接Ajax测试跨域的请求报文：

![koa6](media/15478002097574/koa6.png)
浏览器为这个简单的GET请求添加了Origin，而响应头信息中没有Access-Control-Allow-Origin，浏览器判断请求跨域，给出错误提示。

* 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

在<font color="red">origin.html</font>中添加一个post请求：

```
function corsWithJson() {
    $.ajax({
      url: baseUrl + '/cors',
      type: 'post',
      contentType: 'application/json',
      data: {
        type: 'json',
      },
      success: function(data) {
        console.log(data);
      }
    })
  }

```
通过设置Content-Type为appliaction/json使其成为非简单请求：

启动服务
![koa7](media/15478002097574/koa7.png)
"预检"请求的方法为OPTIONS，服务器判断Origin为跨域

除了Origin字段，"预检"请求的头信息包括两个特殊字段。

（1）**Access-Control-Request-Method**
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。
（2）**Access-Control-Request-Headers**
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是content-type。

<font color="red">服务端设置CORS</font>

在app2.js引入koa2-cors：

```
app.use(cors({
  origin: function (ctx) {
      if (ctx.url === '/cors') {
          return "*"; // 允许来自所有域名请求
      }
      return 'http://localhost:3201';
  },
  exposeHeaders: ['WWW-Authenticate', 'Server-Authorization'],
  maxAge: 5,
  credentials: true,
  allowMethods: ['GET', 'POST', 'DELETE'], //设置允许的HTTP请求类型
  allowHeaders: ['Content-Type', 'Authorization', 'Accept'],
}));

```
重启服务后，浏览器重新发送POST请求。可以看到浏览器发送了两次请求。

![koa8](media/15478002097574/koa8.png)

（1）**Access-Control-Allow-Methods**
该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
（2）**Access-Control-Allow-Headers**
如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
（3）**Access-Control-Allow-Credentials**
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
（4）**Access-Control-Max-Age**
该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

现在为止 默认你已经完全理解跨域了哦

示例中的源代码 

































