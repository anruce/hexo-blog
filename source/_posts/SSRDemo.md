---
title: 实现一个SSR同构应用
date: 2019-03-06 15:51:42
tags:
categories:
---

纸上得来终觉浅，我们来实现一个简易的服务端渲染流程，意在体会SSR带来的红利
<!-- more -->
页面源码来自[React状态管理与同构实战]()

## 几个重要的概念

实现`SSR`是依靠`React`提供的`ReactDomServer`对象

它主要提供了只能在服务端使用的`renderToString()`与`renderToStaticMarkup()`方法
### renderToString()/renderToStaticMarkup()
**使用方法：** `ReactDomServer.renderTostring(element)`/
`ReactDomServer.renderToStaticMarkup(element)`

#### 共同点：

* 都接收一个React Element 并**将此 Element 转化为HTML字符串,通过浏览器返回**，实现了在服务端将页面拼接字符串插入HTML文档中并返回给浏览器 完成初步服务端渲染的目的

#### 不同点

* renderToString(<font color=red>注：React 15</font>) 生成的HTML字符串的每个Dom节点都有`data-react-id`属性，根节点会有一个`data-react-checkSum`属性
* renderToStaticMarkup 不带`data-react-checkSum`属性 浏览器渲染时必会重新渲染组件

-------

<font color="red">关于data-react-checkSum</font>：

```
如果两个组件有相同的props和Dom结构，这个值是一样的

我们知道 服务端渲染完页面内容难过之后，浏览器端也会渲染以完成组件的交互等能力，浏览器端会生成组件的data-react-checkSum值 然后跟服务端渲染组件的值做对比，如果相等，则不再重复渲染
```

这里有一张草图能大概描述这个过程嘤嘤嘤.
![cache_detai](/images/ssr/ssrcc.png)

-------

### ReactDom.hydrate()
React 16以后通过
`renderToString`渲染的组件不再带有`data-react-*`属性，因此浏览器端的渲染方式无法简单通过`data-react-checksum`来判断是否需要重新渲染

基于这样儿的背景下`ReactDom`提供了一个新的API `ReactDom.hydrate()` 用法同`render()`在浏览器端渲染组件

当然，react是向下兼容的，浏览器端在渲染组件时使用render()仍然没有问题，但不论是面向未来，还是基于性能的考虑，都应该采用更好的模式

-------
### renderToNodeStream()/renderToStaticNodeStream()

<font color=red>React 16 </font>为了优化页面的初始加载速度缩短TTFB时间,提供了这两个方法

**概念**
该方法持续产生子节流 返回 `Readable stream` 最终**通过流形式返回的HTML字符串**
这样 服务端处理内容时是实时向浏览器端传输数据而不是一次性处理完成后才开始返回结果的

`renderToStaticNodeStream` 之于 `renderToNodeStream` 也是不会产生`data-react-*`属性，对于静态页面 可以采用此方法。


## 实际开发中可能存在的问题
1. 服务端不存在支持组件挂载的浏览器环境，所以react组件只有`componentDidMount`之前的生命周期方法有效，所以在其之前的生命周期方法中不能用到浏览器的特性，比如 `window、localStorage`.
2. 双端可能都有拉取数据的需求，所以为了实现代码的复用，一种典型的做法就是把请求数据的逻辑放到React组件的静态方法中 然后双端共用，双端请求方法不一致的问题可以通过服务端与浏览器端的判断来封装一下 **比如根据window是浏览器特有对象**


## React 16 在服务端渲染上的惊喜
前面也有混杂说过，在此总结一下

* 在浏览器渲染组件需要配合服务端使用`hydrate`方法
* 提供了`stream`方式的接口
* 与浏览器的新特性相似，除了能处理`React Element` 也能处理别的类型，比如` string number`
* 因为在返回结果Dom中废除了`data-react-checksum`等属性，所以服务端生成HTML更加高效
* 允许在渲染Dom中加入非标准Dom属性

-------

好了 测试一下，基于Node.js实现一个小demo

`Express4.15.3 进行服务端处理`

![ssrvs1](/images/ssr/ssrvs1.png)
browser: 浏览器端渲染
server：服务端逻辑
share：同构的部分

运行效果：
![ssr](/images/ssr/ssrvs100.png)
**share/app.js**

```
import React, { Component } from "react";
import logo from "./logo.svg";
import "./App.css";

class App extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    alert('我被触发辣')
  }
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React in the Server</h2>
        </div>
        <p className="App-intro">点击按钮体验</p>
        <button onClick={e => this.handleClick()}> 我是按钮 </button>  
      </div>
    );
  }
}

export default App;
```
**browser/index.js**

```
import React from "react";
import { hydrate } from "react-dom";
import App from "../shared/App";

hydrate(<App />, document.getElementById("root"));

```
**server/index.js**

```
import express from "express";
import React from "react";
import { renderToString } from "react-dom/server";
import App from "../shared/App";

const app = express();

app.use(express.static("public"));

app.get("*", (req, res) => {
  const htmlMarkup = renderToString(<App />);
  res.send(`
      <!DOCTYPE html>
      <head>
        <title>Universal Reacl</title>
        <link rel="stylesheet" href="/css/main.css">
        <script src="/bundle.js" defer></script>
      </head>

      <body>
        <div id="root">${htmlMarkup}</div>
      </body>
    </html>
  `);
});

app.listen(process.env.PORT || 3000, () => {
  console.log("Server is listening");
});
```


**server端：**使用 `renderToString`生成的字符串，使用`res.send`发送给浏览器
**client端：** id为root的Dom节点就来自服务端返回的结果，用了`React.hydrate`完成了浏览器端的逻辑处理部分

### 假设一 client端渲染仍然使用render()

**测试**

```
import React from "react";
import {render } from "react-dom";
import App from "../shared/App";

render(<App />, document.getElementById("root"));

```

**结果**
由于实现了向下兼容，所以是可以的，但是会给如下警告⚠️
![hydrate_to_rende](/images/ssr/hydrate_to_render.png)

**结论** 尽量使用新特性

-------

### 假设二 完全依赖服务端渲染会发生什么

**测试**
将`browser/index.js`代码注释掉
**结果**
页面正常显示，但是点击按钮没有不会弹窗
**结论** 需要双端一起完成页面的展示与交互

-------
### 假设三 使用React 16 renderToNodeStream渲染

**测试 更改 server/index.js**

```

import express from "express";
import React from "react";
import { renderToNodeStream } from "react-dom/server";
import App from "../shared/App";

const app = express();

app.use(express.static("public"));

app.get("*", (req, res) => {
  res.write(`
      <!DOCTYPE html>
      <head>
        <meta http-equiv="content-type" content="text/html; charset=utf-8">
        <title>Universal Reacl</title>
        <link rel="stylesheet" href="/css/main.css">
        <script src="/bundle.js" defer></script>
      </head>`
  );
  res.write("<div id='root'>"); 
  const stream = renderToNodeStream(<App/>);
  stream.pipe(res, { end: false });
  stream.on('end', () => {
    res.write("</div></body></html>");
    res.end();
  });
});
```
**说明：** 为了配合返回一个流，使用`res.write`方法代替先前的`res.end`

**好处**
使用`renderToString `页面TTFB时间


![TTFB2](/images/ssr/TTFB2.png)
使用`renderToNodeStream `页面TTFB时间

![TTFB1](/images/ssr/TTFB1.png)

**结论**
采用渐进式流渲染可以最大限度的缩短服务器响应水间，从而使浏览器可以更快的接收到信息

-------

### 假设三 同构应用与浏览器渲染优势对比

浏览器渲染：
![client_rende](/images/ssr/client_render.png)

同构应用：
![ss](/images/ssr/ssr.png)

-------

### 假设三 react16比react15渲染更加高效

React 15
![react15_rende](/images/ssr/react15_render.png)
React 16
![react16](/images/ssr/react16.png)

## 遗留问题
1. 鉴于`renderToNodeStream()/renderToStaticNodeStream()`与
`renderToString()/renderToStaticMarkup()`
React 16之后都不存在`data-react-*`了 双方还有什么区别？
2. react 16之后 如何做双端对比？ 官方说是根据`ReactDom.hydrate()`与`renderToString()`结合判断.. 一脸懵逼
 
 