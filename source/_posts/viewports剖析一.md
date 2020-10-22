---
title: viewports剖析一
date: 2017/08/3
tags: [mobile]
toc: false
---
因为工作的原因，很少有机会接触移动web开发，一直都还挺遗憾的，偶尔写几个页面也都是“按照套路”出牌，最近终于有空了解一些概念性的东西，记录一下
文中的图片为了说明问题均为`盗图`，具体出处会在文末注明

本文大概将以下几个概念以做对比
- PC
  - [ <font face="STCAIYUN" size=3 color=blueGreen>设备的pixels和CSS的pixels</font>](#line1)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>所谓的100%缩放</font>](#line2)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>屏幕尺寸和浏览器尺寸</font>](#line3)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>页面的滚动移位</font>](#line4)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>viewport以及度量viewport</font>](#line5)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>html元素以及度量html</font>](#line6)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>关于事件坐标</font>](#line7)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>媒体查询width/height与 device-width/height</font>](#line8)
- mobile
  - [ <font face="STCAIYUN" size=3 color=blueGreen>mobile浏览器</font>](#line9)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>两种viewport</font>](#line10)
      - <font face="STCAIYUN" size=3 color=red>layoutviewport</font>
      - <font face="STCAIYUN" size=3 color=red>visualviewport</font>
  - [ <font face="STCAIYUN" size=3 color=blueGreen>缩放 Zooming</font>](#line11)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>屏幕尺寸</font>](#line12)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>页面的滚动移位</font>](#line13)
 - [ <font face="STCAIYUN" size=3 color=blueGreen>html元素以及度量html</font>](#line14)
 -  [ <font face="STCAIYUN" size=3 color=blueGreen>关于事件坐标</font>](#line15)
  - [ <font face="STCAIYUN" size=3 color=blueGreen>媒体查询width/height与 device-width/height</font>](#line16)
 - [ <font face="STCAIYUN" size=3 color=blueGreen>viewport的meta标签</font>](#line17)

<p id = "line1" >
<font face="STCAIYUN" color=#883958 size=3>设备的pixels和CSS的pixels</font>
</p>
**设备的pixels**
    设备像素是我们直觉上觉得「靠谱」的像素。这些像素为你所使用的各种设备都提供了正规的分辨率
    大多数情况下能从`screen.width/height` 取出具体值
    当然了 设备的pixels对web开发人员几乎毫无用处
    这里只需要知道它的概念即可
**CSS的pixels**
这些就是那些控制你的样式表如何被渲染的像素

现代浏览器上的缩放是基于`伸展`pixels
所以 html元素上的宽度不会因为你缩放了200%而变成了两倍宽，它在形式上还是一倍宽 只不过占用了两倍的设备pixels
如下图1-1 有4个1pixels，缩放为100%的html元素 此时css pixels和设备的pixels完全重合
![](/images/viewports/1.jpg)
此时我们如果缩小浏览器 css的pixels开始收缩，导致1单位的设备pixels 上重叠了多个css的pixels 如下图1-2
![](/images/viewports/2.jpg)
如果放大浏览器 css的pixels就会放大 导致1单位的css pixels上重叠了多个设备pixels 如图1-3
![](/images/viewports/3.jpg)
总而言之 你只需要关注CSS的pixels 这些pixels将指定你的样式被如何渲染
就像上面所说的 设备的pixels对开发人员无用 但是对用户有用，因为用户回手动缩放页面，这些开发人员不用关注 <font face="STCAIYUN" color=red size=3>浏览器会自动保证你的css pixels会被伸展还是被收缩</font>
<p id = "line2" >
<font face="STCAIYUN" color=#883958 size=3>所谓的100%缩放</font>
</p>
<font face="STCAIYUN" color=red size=3>100% 缩放的情况下 1单位的的CSS pixels 严格等于1单位的设备pixels</font>
<p id = "line3" >
<font face="STCAIYUN" color=#883958 size=3>屏幕尺寸和浏览器尺寸</font>
</p>
**屏幕尺寸（Screen size）**
含义：用户的屏幕的完整大小
度量：设备的pixels 不会因为缩放而改变 是显示器的特征
对我们来说<font face="STCAIYUN" color=red size=4>没用</font>
获取方式
如下图1-4
![](/images/viewports/4.jpg)

**浏览器尺寸（Window size）**
含义：包含`滚动条尺寸`的浏览器完整尺寸
度量：CSS pixels
浏览器内部尺寸，它定义了当前用户有多大区域。可供你的css布局占用
如下图1-5
![](/images/viewports/5.jpg)

<p id = "line4" >
<font face="STCAIYUN" color=#883958 size=3>页面的滚动移位</font>
</p>
含义：页面的移位
度量：CSS pixels
定义了页面（document）的相对于窗口远点的位移，可以利用这个特性获取用户滚动了多少的滚动条距离
如下图1-6
![](/images/viewports/6.jpg)

<p id = "line5" >
<font face="STCAIYUN" color=#883958 size=3>viewport以及度量viewport</font>
</p>
**viewport**
啊啊啊 终于提到viewport了 鸡冻
<font face="STCAIYUN" color=red size=3>划重点</font> `viewport`是控制`<html>`元素的容器  是`<html>`的爹

你发现了么？
百分比布局时 你定义的一个侧边栏宽度为10% 当你改变大小时 它的宽度会自动扩张和收缩  原理是啥
当然了 它的宽度是依赖父元素 假如它父元素就是`<body>` 那么`<body>`多宽？
向上类推 `<body>`的宽度取决于它的父元素`<html>`
呃.. 废话好多 `<html>`宽度取决于它的父元素
 `<html>`恰好等于浏览器的宽度 所以你的10%会占用浏览器宽度的10% 我们都是这么用的  今天深扒发现
`<html>`宽度受`viewport`限制 ，等于`viewport`宽度的100%
也就是说
`viewport`严格等于浏览器窗口
需要注意的是：`viewport`不是一个html的概念 所以不能通过CSS修改它

**真实页面宽度概念**
如果你放大页面几倍 如何标识页面宽度（此时已经有横向滚动条了，也就是说页面的内容溢出了`<html>`元素）
使用 document width
如图1-7
![](/images/viewports/7.jpg)
如图1-8
![](/images/viewports/8.jpg)
**度量viewport**
含义：viewport尺寸
度量：CSS pixels
如下图1-9
![](/images/viewports/9.jpg)
document.documentElement 代表<font face="STCAIYUN" color=red size=3>HTML文档根元素</font>`<html>`
来 先看张图
![](/images/viewports/10.jpg)
这张图是在为`<html>`元素赋值25% 但是`document.documentElement.clientWidth`值仍然不变
说明<font face="STCAIYUN" color=red size=3>document. documentElement. clientWidth/Height只会给出viewport的尺寸，而不管<html>元素尺寸如何改变</font>
<font face="STCAIYUN" color=#886 size=4>那么问题来了</font>
我是不是也可以用`window.innerWidth`来定义`viewport`
呃.
他与`document.documentElement.clientWidth`有一点细微的差别
前者不包含滚动条
<p id = "line6" >
<font face="STCAIYUN" color=#883958 size=3>html元素以及度量html</font>
</p>
**html**
ta爹(`viewport`)如果`document.documentElement.clientWidth`表示那么`<html>`这样获取 `document.documentElement.offsetWidth`
![](/images/viewports/11.jpg)
如果给`<html>`元素赋值了宽度 那么`offsetWidth`就会真实的反映出来


![](/images/viewports/12.jpg)

<p id = "line7" >
<font face="STCAIYUN" color=#883958 size=3>关于事件坐标</font>
</p>
**pageX/Y, clientX/Y, screenX/Y**

* pageX/Y：从<html>原点到事件触发点的CSS的 pixels
* clientX/Y：从viewport原点（浏览器窗口）到事件触发点的CSS的 pixels
* screenX/Y：从用户显示器窗口原点到事件触发点的设备 的 pixels。
上图
![](/images/viewports/13.jpg)
![](/images/viewports/14.jpg)
![](/images/viewports/15.jpg)

<p id = "line8" >
<font face="STCAIYUN" color=#883958 size=3>媒体查询width/height与 device-width/height</font>
</p>
* `device-width/height`使用`screen.width/height`来做为的判定值。该值以设备的pixels来度量
* `width/height`使用`documentElement.clientWidth/Height`即viewport的值。该值以CSS的pixels来度量
![](/images/viewports/16.jpg)
桌面浏览器上使用<font face="STCAIYUN" color=red  size=4>width</font>
<p id = "line9" >
<font face="STCAIYUN" color=#883958 size=3>mobile浏览器</font>
</p>
移动设备的屏幕宽度比桌面浏览器小（好多废话.）
试想一下 如果我们只是copy桌面的样式到移动设备 该有多丑
如下图
移动设备浏览器在初始默认打开以最小缩放模式打开网站。（即在手机屏幕上展示完整宽度的页面）

![](/images/viewports/viewport-21.jpg)


<font face="STCAIYUN" color=#285853 size=3>假设当前设备的宽度是400px 还是之前说过的10%侧边栏，如果移动设备上做同样的处理，会显示40px的宽 太窄了，布局会变得非常可怕</font>
那么 如何处理?

<p id = "line10" >
<font face="STCAIYUN" color=#883958 size=3>两种viewport</font></p>
因为viewport太窄，最显然的解决方式就是将它变宽
由此 引出了 虚拟视口  （`viewportvisualviewport`） 与      布局视口（`viewportlayoutviewport`）
关于它们 有一个很好的解释

```
  想象一下viewportlayoutviewport是一张大的不能改变大小和角度的图片 现在你有个更小的框来观看这张大图片
这个框被不透明的材料包围 因而你只能看到大图片的一部分 你通过这个框子看到的大图片的那部分就叫做虚拟视口（viewportvisualviewport）

你拿着这个框拿着站的离大图原点（用户的缩小页面功能）以一次性看到这个大图片
你站的离的近一点（用户的放大页面功能）以看到一部分
你能改变这个框框的远近 但是这张大图片的大小和形状都不会改变
```
visualviewport是当前显示在屏幕上的部分页面。用户会滚动页面来改变可见部分，或者缩放浏览器来改变visualviewport的尺寸。
如下图
![](/images/viewports/20.jpg)
但是CSS布局 通常是按照<font face="STCAIYUN" color=red size=3>layoutviewport定义的，这要比visualviewport宽很多</font>
<html>元素的宽度继承于layoutviewport
<p id = "line11" >
<font face="STCAIYUN" color=#883958 size=3>缩放 Zooming</font>
</p>
两种viewports都以CSS的 pixels来度量。当你通过缩放改变visualviewport时，layoutviewport保存不变。
<p id = "line12" >
<font face="STCAIYUN" color=#883958 size=3>屏幕尺寸</font>
</p>
**理解layout viewport**
许多移动设备浏览器在初始默认打开以最小缩放模式打开网站（也就是在手机屏幕上展示完整宽度的页面）

![](/images/viewports/21.jpg)
此时浏览器已经选择好他们的layoutviewport的尺寸 它完整覆盖了最小缩放模式下的移动浏览器的屏幕，这个时候layoutviewport的宽度高度和最小缩放模式下能在页面上显示的内容的宽度和高度一致。
<br/>
<font face="STCAIYUN" color=red size=3>那么移动端如何计算layoutviewport的尺寸？</font>
`document. documentElement. clientWidth/Height`

![](/images/viewports/25.jpg)
**理解visual viewport**
<font face="STCAIYUN" color=red size=3>那么移动端如何计算visualviewport的尺寸？</font>
`window.innerWidth/Height` 随着用户的缩放浏览器 值会改变 更多 更少的CSS pixels放进了屏幕
![](/images/viewports/27.jpg)
**屏幕尺寸 screen**
和pc浏览器一样 screen.width/height标示了设置屏幕的尺寸 以设备的pixels 显然 这跟开发人员没有什么关系

<p id = "line13" >
<font face="STCAIYUN" color=#883958 size=3>页面的滚动移位</font>
</p>
你同样需要知道当前的虚拟视口相对于布局视口的距离 这叫做`滚动位移` ，它像在pc端获取一样
使用**window.pageX/YOffset**

![](/images/viewports/29.jpg)

<p id = "line14" >
<font face="STCAIYUN" color=#883958 size=3>html元素以及度量html</font>
</p>
html元素的整体尺寸，和pc端一致 使用`document.documentElement.offsetWidth/Height`，元素以CSS pixels度量

![](/images/viewports/30.jpg)
<p id = "line15" >
<font face="STCAIYUN" color=#883958 size=3>关于事件坐标</font>
</p>
同pc浏览器 只需要关注 pageX/Y

![](/images/viewports/32.jpg)

<p id = "line16" >
<font face="STCAIYUN" color=#883958 size=3>媒体查询width/height与 device-width/height</font>
</p>
也如同pc浏览器
 `width/height `使用css的pixels度量layoutviewport 即`document. documentElement. clientWidth/Height `
`device-width/height  `使用设备的pixels 即 `screen.width/height. `
所有浏览器都遵循这个原理
<p id = "line17" >
<font face="STCAIYUN" color=#883958 size=3>viewport的meta标签</font>
</p>
```
<meta name="viewport" content="width=320">
```
最初这是apple的一个html扩展标签，被很多浏览器复用
设置 `虚拟视口`的宽度

假设你现在创建一个页面  并不为它设置宽度 那么它会伸展开来占据100%的viewlayout的宽度 绝大多数浏览器缩小这个页面在一屏的宽度上显示这个layoutviewport

![](/images/viewports/33.jpg)
当用户放大页面  绝大多数会保存元素的宽度（保持元素的定位不变）而导致文字超出屏幕
![](/images/viewports/34.jpg)
当你设置
```
<meta name="viewport" content="width=320">
```
你网站的layoutviewport变成了320px。页面的初始状态就很正确了
![](/images/viewports/36.jpg)

- Part1:http://www.quirksmode.org/mobile/viewports.html
- Part2:http://www.quirksmode.org/mobile/viewports2.html
