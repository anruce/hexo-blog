
---
title: Express框架系列(一)之概述
date: 2017/04/17
tags: [node]
toc: false
---

参考文档：http://javascript.ruanyifeng.com/nodejs/express.html
Express是目前最流行的基于Node.js的Web开发框架，可以快速地搭建一个完整功能的网站。

创建测试项目
`mkdir hello-world`

进入该目录，新建一个package.json文件，内容如下。


```
{
  "name": "hello-world",
  "description": "hello world test app",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "express": "4.x"
  }
}

```
安装 express
`npm install`
执行上面的命令以后，在项目根目录下，新建一个启动文件，假定叫做`index.js。`


```
var express = require('express');
var app = express();

app.use(express.static(__dirname + '/public'));

app.listen(8080);
```
运行启动脚本
`node index`
访问 http://localhost:8080
它会在浏览器中打开当前目录的**public**子目录（严格来说，是打开public目录的index.html文件）。如果public目录之中有一个图片文件my_image.png，那么可以用http://localhost:8080/my_image.png访问该文件。

目录结构
![屏幕快照 2017-03-28 下午3.55.13](media/14906850474605/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-28%20%E4%B8%8B%E5%8D%883.55.13.png)

执行结果
![屏幕快照 2017-03-28 下午3.54.14](media/14906850474605/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-28%20%E4%B8%8B%E5%8D%883.54.14.png)
也可以http://localhost:8080/WX.jpeg

也可以生成动态网页


```var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.listen(3000);
```
 运行启动脚本
`node index`

![屏幕快照 2017-03-28 下午3.57.07](media/14906850474605/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-28%20%E4%B8%8B%E5%8D%883.57.07.png)

启动脚本index.js的**app.get**方法，用于指定不同的访问路径所对应的回调函数，这叫做“**路由**”（routing）。上面代码只指定了根目录的回调函数，因此只有一个路由记录。实际应用中，可能有多个路由记录。

```
比如：
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello world!');
});
app.get('/customer', function(req, res){
  res.send('customer page');
});
app.get('/admin', function(req, res){
  res.send('admin page');
});

app.listen(3000);

```
比较庞大的时候可以单独存放
eg：

```
// routes/index.js

module.exports = function (app) {
  app.get('/', function (req, res) {
    res.send('Hello world');
  });
  app.get('/customer', function(req, res){
    res.send('customer page');
  });
  app.get('/admin', function(req, res){
    res.send('admin page');
  });
};
```
原先的index引入

```
// index.js
var express = require('express');
var app = express();
var routes = require('./routes')(app);
app.listen(3000);
```
