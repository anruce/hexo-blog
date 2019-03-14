
---
title: DomContentLoaded 与 load
date: 2018/03/30
tags: 
toc: false
comments: true
categories: web
---

## 浏览器渲染原理

```
当我们在浏览器地址输入url时，浏览器会发送请求到服务器，服务器将请求的html文档发送回浏览器，浏览器将文档下载下来后 便开始从上到下解析，解析完成后 会生成dom，如果页面中有css 会根据css的内容 形成cssdom 然后 dom和css会生成一个渲染树 最后浏览器会根据渲染树的内容计算出各个节点在页面中的确切大小和位置，并将其绘制在浏览器上
```
![](http://oucjferwh.bkt.clouddn.com/746387-20170407181220066-2064922697.png)

```
在解析html的过程中 有时候解析会被中断，这是因为javascript会阻塞dom的解析 当解析过程中遇到script标签的时候 便会停止解析过程 抓转而去处理脚本 如果脚本是内联的 浏览器会先去执行这段内联的脚本，如果脚本是外链的  那么先去加载脚本 然后执行 在处理完脚本之后  浏览器便继续解析html文档
```
### 如何计算DomContentLoaded 加载时间
当文档中没有脚本时 浏览器解析完成文档便能触发 DomContentLoaded 事件 如果文档包含脚本 则脚本会阻塞文档的解析 而脚本需要等位于前面的css加载完才能执行 在任何情况下  DomContentLoaded 的触发不需要等待图片等其他资源加载完成

```
DOMContentLoaded不同的浏览器对其支持不同，所以在实现的时候我们需要做不同浏览器的兼容。

1）支持DOMContentLoaded事件的，就使用DOMContentLoaded事件；

2）IE6、IE7不支持DOMContentLoaded，但它支持onreadystatechange事件，该事件的目的是提供与文档或元素的加载状态有关的信息。

1) 更低的ie还有个特有的方法doScroll， 通过间隔调用：document.documentElement.doScroll("left");
可以检测DOM是否加载完成。 当页面未加载完成时，该方法会报错，直到doScroll不再报错时，就代表DOM加载完成了。该方法更接近DOMContentLoaded的实现。

```
### 如何计算load 加载时间
页面上所有的资源（图片，音频，视频等）被加载以后才会触发load事件，简单来说，页面的load事件会在DOMContentLoaded被触发之后才触发。

```
window.onload = function(){

}

```

### 我们为什么一再强调将CSS放在头部 将js放在尾部
因为浏览器生成Dom树的时候是一行一行读html代码的  script标签放在最后面就不会影响前面的页面渲染，那么问题来了
既然Dom树完全生成好页面才能渲染出来 浏览器又必须读完全部的html才能生成完成的dom树 script标签放不放在底部是不是也一样 因为dom树的生成需要整个文档解析完成

<font color=red>chrome页面渲染过程中 会有 firstpaint的概念，现代浏览器为了更好的用户体验，渲染引擎将尝试尽快在屏幕上显示的内容 他不会等到所有的html解析完成才开始构建和布局dom树 部分的内容被解析并展示 也就是说 浏览器能够渲染不完整的dom树和cssdom 尽快的减少白屏时间
</font>

假如我们将js放在header js将会阻塞解析dom dom的内容会影响到 firstpaint 导致firstpaint延后 所以说我们会将js放在后面 以减少firstpaint时间但是不会减少 DomContentLoaded 被触发的时间



