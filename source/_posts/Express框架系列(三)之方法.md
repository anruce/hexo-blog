---
title: Express框架系列(三)之方法
date: 2018/04/18
tags: [node]
toc: false
comments: true
categories: 前端
---

### all方法和HTTP动词方法
针对不同的请求，Express提供了use方法的一些别名。比如，上面代码也可以用别名的形式来写。


```
var express = require("express");
var http = require("http");
var app = express();

app.all("*", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  next();
});

app.get("/", function(request, response) {
  response.end("Welcome to the homepage!");
});

app.get("/about", function(request, response) {
  response.end("Welcome to the about page!");
});

app.get("*", function(request, response) {
  response.end("404!");
});

http.createServer(app).listen(1337);
```
除了get方法以外，Express还提供post、put、delete方法，即HTTP动词都是Express的方法，express允许模式匹配
### set方法
用于指定变量的值

```
app.set('views',_dirname+'/views')
app.set("view engine", "jade");

```
### response 对象
response.redirect()允许网址的重定向
`response.redirect("/hello/anime");`

response.sendFile()用于发送文件
`response.sendFile("/path/to/anime.mp4")`

response.render() 用于渲染网页模版


```
app.get("/", function(request, response) {
  response.render("index", { message: "Hello World" });
});
```
使用render方法，将message变量传入index模版，渲染成HTML网页

### request对象

request.ip:用于获取HTTP请求的IP地址
request.files 用于获取上传的文件

### 搭建[HTTPS](http://baike.baidu.com/link?url=RDLn4MhlPev2MMCPjJvGa0aEf2Fg2DzGyz-Eqo7AwmYdYDjPTvyZuku-svVMjAlHcvsZm9PQ4bGPcjFW7VPcDK)服务器
使用express搭建https加密服务器


```
var fs = require('fs');
var options = {
  key: fs.readFileSync('E:/ssl/myserver.key'),
  cert: fs.readFileSync('E:/ssl/myserver.crt'),
  passphrase: '1234'
};

var https = require('https');
var express = require('express');
var app = express();

app.get('/', function(req, res){
  res.send('Hello World Expressjs');
});

var server = https.createServer(options, app);
server.listen(8084);
```
