
---
title: Express框架系列(二)之运行原理
date: 2017/04/18
tags: [node]
toc: false
---

### 底层HTTP模块
Express框架建立在node.js内置的http模块上，**框架的核心是对HTTP模块的再包装**
http模块生成服务器的原始代码如下：

```
var http = require("http");

var app = http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
    response.end(" express hahahah");
});

app.listen(4040, "localhost");
```
上面的代码关键是http模块的**createServer**方法，表示生成一个http服务器实例，该方法接受一个回调函数，回调函数的两个参数分别代表HTTP请求和HTTP响应的**request**对象和**response**对象

![](http://oucjferwh.bkt.clouddn.com/node2-1.png)

上面的代码用Express改写如下

```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello world!');
});

app.listen(7000);
```
![](http://oucjferwh.bkt.clouddn.com/node2-2.png)

可以发现两端代码特别相似，原来是用http.createServer方法新建一个app实例，现在则是用Express的构造方法，生成一个Epress实例，两种方法的回调函数都是相同的，
Express等于在HTTP模块之上，加了一个**中间层**


### 什么是中间件
中间件就是处理HTTP请求的函数，
特点：一个中间件处理完再传递给下一个中间件，APP实例在运行中会调用一系列的中间件

每个中间件可以从APP实例接收三个参数 request(代表HTTP请求)，response(代表HTTP响应),next回调函数(代表下一个中间件)，每一个中间件都可以对HTTP请求(request对象)进行加工，并且决定是否调用next方法，将request对象再传给下一个中间件

最简单的中间件


```
function uselessMiddleware(req,res,next){
next()
}

上面代码的next就是下一个中间件。如果它带有参数，则代表抛出一个错误，参数为错误文本。

function uselessMiddleware(req, res, next) {
  next('出错了！');
}
抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止。


```
##### use方法
use是express注册中间件的方法，使用app.use方法，注册了两个中间件，收到HTTP请求后，先调用第一个中间件，根据next()确定是否把request对象传递到下一个中间件

```
var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response, next) {
  console.log("In comes a " + request.method + " to " + request.url);
  next();
});

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

http.createServer(app).listen(1337);
```
use方法内部通过`request.url`的属性可以根据访问路径进行判断，据此就能实现简单的路由，

```
var express = require("express");
var http = require("http");
var app = express();

app.use(function(request,response,next){
     if(request.url == "/"){
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Welcome to the homepage!\n");
     }else{
     next();
     }
})

app.use(function(request, response, next) {
  if (request.url == "/about") {
    response.writeHead(200, { "Content-Type": "text/plain" });
  } else {
    next();
  }
});
app.use(function(request, response) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!\n");
});

http.createServer(app).listen(1337);

```
另外一种比较清晰的方式（上面代码表示，只对根目录的请求，调用某个中间件。
）
`
app.use('/path', someMiddleware);
`
按照这个思想，改造中间件

```
var express = require("express");
var http = require("http");

var app = express();

app.use("/home", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Welcome to the homepage!\n");
});

app.use("/about", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Welcome to the about page!\n");
});

app.use(function(request, response) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!\n");
});

http.createServer(app).listen(1337);
```
