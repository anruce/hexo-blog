---
title: Node.js实战-技术预研
date: 2020-03-23 00:25:24
tags:
categories:
---

# 前言

以一种要开发Node.js实战项目为最终目标
进行一系列的技术预研过程

有特点，有针对性，有目标

培养Node领域的全局观

# 1 关于Nodejs

## 1.1 什么是Node.js

官网的话：

- Node.js是基于ChromeV8执行引擎的JS运行时环境
- Node.js使用了一个事件驱动，非阻塞式I/O的模型，使其轻量又高效


每一个字其实都看得懂，聚合到一起就有点懵了

![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1584894283204&di=6e87b25c91207a1929250f11285d3a2c&imgtype=0&src=http%3A%2F%2Fimg2.biaoqingjia.com%2Fbiaoqing%2F201608%2Fa70d12bb1f409850857c8d930cf2d6d1.gif)

我们先不来说nodejs是什么，先根据以往的经验抛出问题

### 1.1.1 在Node.js里运行Js跟在Chrome运行Js有啥不同？

已知Chrome浏览器用的是同样的Javascript引擎和模型

> 其实，在Node.js里写Js和在Chrome里写Js，<font color=red>几乎一样</font>


晃眼的<font color=red>几乎一样</font> 那就是有不一样的地方呗！

- Nodejs没有浏览器API，即(Document,window等)
- 相应的，也增加了它专属的API，比如文件系统，进程.

有了这些差别，其实就不难理解了

#### 对于开发者来说
- 你在chrome里写js**控制浏览器**
- Node.js让你用类似的方式，**控制整个计算机**


Node.js的真谛，也就是官方抽象的释义，我们完全可以在不断深入的过程中慢慢理解～

<!-- more -->
## 1.2 Node.js可以用来做什么？

### 1.2.1 提供Web服务
- 搜索引擎优化 + 首屏速度优化 = 服务端渲染
- 服务端渲染 + 前后端同构 = Node.js

### 1.2.2 构建工作流

在`gulp webpack`之间，前端是如何做构建工具呢？

可能用java,ruby等

但
- 构建工具不会永远不出问题
- 构建工具不会永远满足需求

前端同学很难对这些工具进行修改或者升级

所以
> 用Node.js做js的构建工具，是最保险的选择

### 1.2.3 开发工具

VScode

在nodejs的基础上封装了chrome的内核，使nodejs具有控制计算机得到能力

### 1.2.3 可扩展性较强大的沙盒游戏

需要给使用者自定义模块的能力

使用Nodejs做复杂的本地应用
- 可以利用js大的灵活性实现外部扩展
- Js庞大的的开发者基数让他们的灵活性得到利用

### 1.2.4 客户端应用

在已有网站的基础上需要开发新的客户端应用
使用Node.js客户端技术实现，可以最大限度的复用现有工程

# 2 Node.js 初探

## 2.1 实现剪刀石头布

- node运行方式游戏
- 全局变量

```
var playerAction = process.argv[process.argv.length - 1];
console.log("playerAction", playerAction);

var random1 = Math.random() * 3;

if (random1 < 1) {
  var computerAction = "rock";
} else if (random1 > 2) {
  var computerAction = "scissor";
} else {
  var computerAction = "paper";
}

if (computerAction === playerAction) {
  console.log("平局");
} else if (
  (computerAction === "rock" && playerAction === "paper") ||
  (computerAction === "scissor" && playerAction === "rock") ||
  (computerAction === "paper" && playerAction === "scissor")
) {
  console.log("你赢了！");
} else {
  console.log("你输了！");
}

```

## 2.1 使用Node.js模块规范改造游戏

### 2.1.1 如何加载js

浏览器端
- 使用`<script/>`标签
- 脚本变多时，需要手动管理加载顺序
- 不同脚本之间的逻辑调用需要全局变量


Node端
- 没有html文件，无法使用`<script/>`标签

![image](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=45975645,3909583844&fm=11&gp=0.jpg)

所以Node.js 要重新去搞一个模块管理机制来管理js的加载，就是现在我们熟悉的<font color=red>CommonJS规范</font>


### 2.1.2 重构剪刀石头布游戏

games.js

```
module.exports = function(playerAction) {
  if (["rock", "scissor", "paper"].indexOf(playerAction) == -1) {
    throw new Error("invalid playerAction");
  }
  // 计算电脑出的东西
  var computerAction;
  var random = Math.random() * 3;
  if (random < 1) {
    computerAction = "rock";
    console.log("电脑出了石头");
  } else if (random > 2) {
    computerAction = "scissor";
    console.log("电脑出了剪刀");
  } else {
    computerAction = "paper";
    console.log("电脑出了布");
  }

  if (computerAction == playerAction) {
    console.log("平局");
    return 0;
  } else if (
    (computerAction == "rock" && playerAction == "scissor") ||
    (computerAction == "scissor" && playerAction == "paper") ||
    (computerAction == "paper" && playerAction == "rock")
  ) {
    console.log("你输了");
    return -1;
  } else {
    console.log("你赢了");
    return 1;
  }
};

```
index.js

```
const game = require("./game.js");

var winCount = 0;
// 获取进程的标准输入
process.stdin.on("data", buffer => {
  // 回调的是buffer，需要处理成string
  const action = buffer.toString().trim();
  const result = game(action);
  if (result == 1) {
    winCount++;
    if (winCount == 3) {
      console.log("我不玩儿了！哼！");
      process.exit();
    }
  }
});

```

# 3 Node内置模块

[内置模块合集](http://nodejs.cn/api/)

## 3.1 Node.js系统架构图

![image](http://cdn.anruence.com/node-system-1.png)

## 3.2 理解Node.js精髓
> Node.js是基于ChromeV8执行引擎的JS运行时环境

<font color=red>ChromeV8执行引擎的JS运行时环境</font>：架构图的左侧部分就是其体现

- application 代表你写的nodejs的代码
- 通过V8引擎来来运行，里面会涉及到一些关于操作系统调用，这部分就由V8引擎帮你转发到操作系统层面
- 从操作系统层面得到返回结果之后再通过V8引擎返回到Js里去

1. 从Js到V8再到操作系统的能力，大部分都是通过Node.js的内置模块来提供的
2. 还有一些数据是从操作系统底层通知到我们的Node.js层

示例

```
// 将进程设置为长期存在并且监听用户的输入
process.stdin.on('data',e=>{
  const playerAction = e.toString().trim();
})
```

此时依赖的是Node的内置模块
- EventEmitter

process实际上是EventEmitter的实例，继承了EventEmitter使其具备了向上抛事件的能力

引出

## 3.3 EventEmitter
### 3.3.1 解决了什么问题
- 解决两个对象之间的通信问题
    - 函数调用
    - 观察者模式（事件收发模式）- 抛事件
        - addEventListener
        - removeEventListener

### 3.3.2 普通调用应用场景
- 老板通知秘书
- 说是通知，但是直接调用比较合适
### 3.3.3 观察者模式应用场景
- 通知消息的人并不知道被通知者的存在（极客时间并不知道我的存在）
- 没有人接收事件，它还能继续下去（今天没有接收到Geek上新课的消息，但是它还是可以上新课）


lib.js

```
const EventEmitter = require('events')
class Geektime extends EventEmitter{
  constructor(){
    super();
    setInterval(() => {
      this.emit('newlesson',{price:Math.random()* 100}) //触发事件
    }, 3000);
  }
}

const geektime = new Geektime;

module.exports = geektime
```

index.js

```
const geektime = require('./lib.js')
geektime.addListener('newlesson',(res)=>{
  if(res.price < 50){
    console.log('buy!当前价格为---',res)
  }
})

```

# 4 Nodejs非阻塞I/O及异步编程
值得拿出来单独说，[戳此一览](http://maying.ink/2019/03/19/promise/)


# 5 实现网页版石头剪刀布游戏

技术前置

## 5.1 什么是HTTP服务
一个网页请求，包含两次HTTP包交换
- 浏览器向HTTP服务器发送请求HTTP包
- HTTP服务器向浏览器返回HTTP包

## 5.2 HTTP服务要做什么事情
- 解析进来的HTTP请求报文
- 返回对应的HTTP返回报文 

## 5.3 实现一个简单的HTTP服务器

```
const http = require('http')
const fs = require('fs')
http
.createServer((req,res)=>{
    res.writeHead(200);
    res.end('hello')

})
.listen(8888)
```
## 5.4 server端加载模版

```
const http = require('http')
const fs = require('fs')
http
.createServer((req,res)=>{
    res.writeHead(200);
    res.end('hello')
    fs.createReadStream(__dirname + '/index.html')
    .pipe(res)

})
.listen(8888)
```
### 5.5 游戏逻辑 

index.js  [源码点击](https://github.com/maying2020/nodejs-in-action/tree/master/http/nodeNative)

```
const http = require("http");
const fs = require("fs");
const url = require("url");
const querystring = require("queryString");
const game = require("./game.js");

let playerLastAction = null; //玩家上次出的
let playerWon = 0; //玩家赢得次数
let sameCount = 0; //统计相同操作统计次数

http
  .createServer((req, res) => {
    // 通过内置模块url，转换发送到该http服务上的http请求包的url，
    // 将其分割成 协议(protocol)://域名(host):端口(port)/路径名(pathname)?请求参数(query)
    const parsedUrl = url.parse(req.url);
    if (parsedUrl.pathname == "/game") {
      const query = querystring.parse(parsedUrl.query);
      // 玩家出的
      const playerAction = query.action;

      // 需求2:如果玩家赢了三次或者玩家作弊，则电脑不给他玩了
      if (playerWon >= 3 || sameCount == 9) {
        res.writeHead(500);
        res.end("我再也不和你玩了！");
        return;
      }
      // 需求1:如果玩家操作连续三次相同，视为玩家作弊
      if (playerLastAction & (playerLastAction == playerAction)) {
        sameCount++;
      } else {
        sameCount++;
      }

      playerLastAction = playerAction;

      if (sameCount >= 3) {
        res.writeHead(400);
        res.end("你作弊");
        // 将sameCount设置为9
        sameCount = 9;
        return;
      }
      // 执行游戏逻辑
      var gameResult = game(playerAction);
      res.writeHead(200);
      if (gameResult == 0) {
        res.end("平局！");
      } else if (gameResult == 1) {
        res.end("你赢了！");
        // 玩家胜利次数统计+1
        playerWon++;
      } else {
        res.end("你输了！");
      }
    }
    // 如果请求url是浏览器icon，比如 http://localhost:3000/favicon.ico的情况
    // 就返回一个200就好了
    if (parsedUrl.pathname == "/favicon.ico") {
      res.writeHead(200);
      res.end();
      return;
    }
    // 如果访问的是根路径，就把游戏页面读出来返回出去
    if (parsedUrl.pathname == "/") {
      fs.createReadStream(__dirname + "/index.html").pipe(res);
    }
  })
  .listen(6001);

```

# 6 使用express优化石头剪刀布游戏

## 6.1 了解express

要了解一个框架，最好的方法是 
1. 了解它的关键功能
1. 推导出它要解决的问题是什么

核心功能

- 路由
- request/response 简化
    - request:pathname、query等
    - response:send()、json()、jsonp()等
- 中间件
    - 更好地组织流程代码
    - 异步会打破Express的洋葱模型

## 6.2 游戏逻辑
index.js [源码点击](https://github.com/maying2020/nodejs-in-action/tree/master/http/express)

```
const fs = require("fs");
const game = require("./game");
const express = require("express");

// 玩家胜利次数，如果超过3，则后续往该服务器的请求都返回500
var playerWinCount = 0;
// 玩家的上一次游戏动作
var lastPlayerAction = null;
// 玩家连续出同一个动作的次数
var sameCount = 0;

const app = express();

// 通过app.get设定 /favicon.ico 路径的路由
// .get 代表请求 method 是 get，所以这里可以用 post、delete 等。这个能力很适合用于创建 rest 服务
app.get("/favicon.ico", function(request, response) {
  // 一句 status(200) 代替 writeHead(200); end();
  response.status(200);
  return;
});

// 设定 /game 路径的路由
app.get(
  "/game",

  function(request, response, next) {
    if (playerWinCount >= 3 || sameCount == 9) {
      response.status(500);
      response.send("我不会再玩了！");
      return;
    }

    // 通过next执行后续中间件
    next();

    // 当后续中间件执行完之后，会执行到这个位置
    if (response.playerWon) {
      playerWinCount++;
    }
  },

  function(request, response, next) {
    // express自动帮我们把query处理好挂在request上
    const query = request.query;
    const playerAction = query.action;

    if (!playerAction) {
      response.status(400);
      response.send();
      return;
    }

    if (lastPlayerAction == playerAction) {
      sameCount++;
      if (sameCount >= 3) {
        response.status(400);
        response.send("你作弊！我再也不玩了");
        sameCount = 9;
        return;
      }
    } else {
      sameCount = 0;
    }
    lastPlayerAction = playerAction;

    // 把用户操作挂在response上传递给下一个中间件
    response.playerAction = playerAction;
    next();
  },

  function(req, response) {
    const playerAction = response.playerAction;
    const result = game(playerAction);

    // 如果这里执行setTimeout，会导致前面的洋葱模型失效
    // 因为playerWon不是在中间件执行流程所属的那个事件循环里赋值的
    // setTimeout(()=> {
    response.status(200);
    if (result == 0) {
      response.send("平局");
    } else if (result == -1) {
      response.send("你输了");
    } else {
      response.send("你赢了");
      response.playerWon = true;
    }
    // }, 500)
  }
);

app.get("/", function(request, response) {
  // send接口会判断你传入的值的类型，文本的话则会处理为text/html
  // Buffer的话则会处理为下载
  response.send(fs.readFileSync(__dirname + "/index.html", "utf-8"));
});

app.listen(6001);

```

# 7 使用koa优化石头剪刀布游戏

## 7.1 了解koa

核心功能:

- 比 Express 更极致的 request/response 简化
    - ctx.status=200
    - ctx.body='helloworld'
- 使用 async function 实现的中间件
    - 有“暂停执行”的能力
    - 在异步的情况下也符合洋葱模型
- 精简内核，所有额外功能都移到中间件里实现

##  7.2 Express vs Koa
- Express 门槛更低，Koa 更强大优雅。
- Express 封装更多东西，开发更快速，Koa 可定制型更高

## 7.3 孰“优”孰“劣”
- 框架之间其实没有优劣之分
- 不同的框架有不同的适用场景

## 7.4 游戏逻辑

index.js [源码点击](https://github.com/maying2020/nodejs-in-action/tree/master/http/koa)

```
const fs = require("fs");
const game = require("./game");
const koa = require("koa");
const mount = require("koa-mount");

// 玩家胜利次数，如果超过3，则后续往该服务器的请求都返回500
var playerWinCount = 0;
// 玩家的上一次游戏动作
var lastPlayerAction = null;
// 玩家连续出同一个动作的次数
var sameCount = 0;

const app = new koa();

app.use(
  mount("/favicon.ico", function(ctx) {
    // koa比express做了更极致的response处理函数
    // 因为koa使用异步函数作为中间件的实现方式
    // 所以koa可以在等待所有中间件执行完毕之后再统一处理返回值，因此可以用赋值运算符
    ctx.status = 200;
  })
);

const gameKoa = new koa();
app.use(mount("/game", gameKoa));
gameKoa.use(async function(ctx, next) {
  if (playerWinCount >= 3) {
    ctx.status = 500;
    ctx.body = "我不会再玩了！";
    return;
  }

  // 使用await 关键字等待后续中间件执行完成
  await next();

  // 就能获得一个准确的洋葱模型效果
  if (ctx.playerWon) {
    playerWinCount++;
  }
});
gameKoa.use(async function(ctx, next) {
  const query = ctx.query;
  const playerAction = query.action;
  if (!playerAction) {
    ctx.status = 400;
    return;
  }
  if (sameCount == 9) {
    ctx.status = 500;
    ctx.body = "我不会再玩了！";
  }

  if (lastPlayerAction == playerAction) {
    sameCount++;
    if (sameCount >= 3) {
      ctx.status = 400;
      ctx.body = "你作弊！我再也不玩了";
      sameCount = 9;
      return;
    }
  } else {
    sameCount = 0;
  }
  lastPlayerAction = playerAction;
  ctx.playerAction = playerAction;
  await next();
});
gameKoa.use(async function(ctx, next) {
  const playerAction = ctx.playerAction;
  const result = game(playerAction);

  // 对于一定需要在请求主流程里完成的操作，一定要使用await进行等待
  // 否则koa就会在当前事件循环就把http response返回出去了
  await new Promise(resolve => {
    setTimeout(() => {
      ctx.status = 200;
      if (result == 0) {
        ctx.body = "平局";
      } else if (result == -1) {
        ctx.body = "你输了";
      } else {
        ctx.body = "你赢了";
        ctx.playerWon = true;
      }
      resolve();
    }, 500);
  });
});

app.use(
  mount("/", function(ctx) {
    ctx.body = fs.readFileSync(__dirname + "/index.html", "utf-8");
  })
);
app.listen(6001);

```
