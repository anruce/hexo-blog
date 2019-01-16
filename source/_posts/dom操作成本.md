---
title: Dom操作成本 浏览器的重排和重绘
date: 2018/05/16
tags: [js]
toc: false
---

操作Dom的成本很高 不要轻易去操作Dom 这句话从开始入门就听说，那么这里说的成本是指什么？
由此引出今天的问题

首先我们要清楚几个概念

### 什么是DOM？
   
* DOM全称 Document Object Model 文档对象模型
* 它是为HTML（XML）提供的API
* HTML是一种标记语言 HTML在DOM模型标准中被视为对象
* DOM只提供编程接口却无法实际操作HTML里面的内容 
* 在浏览器中 前端工程师可以通过脚本语言（js）通过DOM去操作HTML内容
（不只js能调用DOM这个API Python也可以）
*  ps：也存在CSSOM：CSS Object Model 浏览器将CSS代码解析成树形的数据结构与DOM是两个独立的数据机构
 
### 浏览器渲染过程
讨论DOM操作成本 首先要了解下该成本的来源 那么就离不开浏览器渲染
浏览器渲染前需要先构建DOM和CSS树 因此我们需要尽快将HTML和CSS都提供给浏览器

这里只讨论浏览器拿到HTML之后开始解析 渲染 

<font color=red>之前的一些另开一篇</font>
1. 解析HTML 构建DOM树 （这里遇到外链 会发起请求）
2. 解析CSS 生成CSS规则树
3. 合并DOM树和CSS规则 生成render（渲染）树
4. 布局render树（Layout/reflow）负责各元素的尺寸，位置的计算
5. 绘制render树（paint）绘制页面像素信息
6. 浏览器会将各层的信息发送给GPU GPU将各层合成(composite) 显示在屏幕上
##### 构建DOM树
<font color=red>HTML 标记转换成文档对象模型 (DOM)</font>
DOM树构建过程：当前节点的所有子节点都构建好后才会去构建当前节点的下一个兄弟节点
##### 构建CSSOM树
<font color=red>CSS 标记转换成 CSS 对象模型 (CSSOM)</font>
在最终计算各个2节点的样式时 浏览器都会先从该节点的普遍属性（比如全局样式）开始  再去应用该节点的具体属性

每个浏览器都有自己的默认样式表因此很多时候这颗CSSOM树只是对这张默认样式表的部分替换

DOM 和CSSOM都要经过
`Bytes→characters→tokens→nodes→objectmodel`这个过程
DOM和CSSOM是独立的数据结构
此处需要一张图片


##### 生成render（渲染）树 由此 浏览器中会解析并生成两个内部数据结构
* Dom树表示页面结构
* DOM树和CSSOM合并生成render树（渲染树），渲染树表示Dom节点在页面中如何显示（宽高 位置等）


```
在dom树中每一个需要显示的节点在渲染树种至少存在一个对应的节点 渲染树中的节点被称之为“帧”或者“盒” 符合css模型的定义 一旦Dom树和渲染树构建完成  浏览器就开始 显示（绘制paint）页面元素
```


简单描述下render的过程

```
DOM树从根节点开始遍历可见节点
设置了类似 display：none （则该节点不可见） 在render过程中是被跳过的
visibility:hidden; opacity:0 这种仍旧占据空间的节点不会被跳过render  保存各个节点的样式信息及其余节点的从属关系
```

```
Layout布局
有了各个节点的信息属性 但不知道各个节点的确切位置和大小 所以要通过布局将样式信息和属性转换为实际可视窗口的相对大小和位置
（DOM 树捕获文档标记的属性和关系，但并未告诉我们元素在渲染后呈现的外观。那是 CSSOM 的责任）
```

```
Paint绘制
最后只要将确定好位置大小的各节点通过GPU渲染到屏幕的实际像素
```

TIPS：

* 在上述渲染过程中 前三点可能要多次执行 比如js脚本去操作DOM 更改CSS样式 浏览器又要重新构建DOM CSSOM树 重新render 重新layout paint
* 因为layout在paint之前 因此每次layput重新布局（reflow回流）后都要重新触发paint渲染 这时又要去消耗GPU
* paint不一定会触发layout 比如改个颜色改个背景（repaint重绘）
* 图片下载完也会重新触发Layout和paint


##### 何时触发reflow（重排）和repaint（重绘）


<font color=red>reflow(重排)：</font>当dom树的变化影响了元素的集合属性 =》 意味着元素的内容，结构 位置或者尺寸发生了变化，同样其他元素的集合属性和位置也会因此受到影响，浏览器会使渲染树（render树）中受到影响的部分失效  需要重新计算样式和渲染树，这个过程称为重排（reflow）

<font color=red>repaint(重绘)：</font> 意味着元素发生的改变只你影响了节点的一些样式（背景色 边框颜色 文字元素等）只需要应用新样式绘制这个元素就可以了 （完成重排后 浏览器会重新绘制受影响的部分到屏幕中 这个过程叫做重绘）

并不是所有的dom辩护都会影响几何属性 例如  改变元素的背景色不会影响 宽和高 这种情况下 只会发生一次重绘（不需要重排）因为元素的布局没有改变

<font color=red>重排一定会引起浏览器的重绘 重绘则不一定伴随重排</font>

<font color=red>重排</font>的成本开销要高于<font color=red>重绘</font>一个节点的重排往往导致子节点以及同级节点的重排



###### 触发重排的情况

当页面布局的几何属性改变时就需要重排 下列情况会导致重排

```
页面第一次渲染（初始化）

DOM树变化（如：增删节点）
元素位置改变
元素尺寸改变（外边距 内边距 边框厚度 宽度 高度等）
Render树变化（如：padding改变）
浏览器窗口resize
获取元素的某些属性：
当滚动条出现时，会触发整个页面的重排
```

由于每次重排都会产生计算消耗，大多数浏览器通过队列化修改并批量执行来优化重排的过程

但是  我们经常会不知不觉强制刷新队列并要求计划任务立即执行
获取布局信息的操作会到最后队列刷新  比如

```
offsetTop , offsetLeft , offsetWidth , offsetHeight

scrollTop , scrollLeft , scrollWidth , scrollHeight

clientTop , clientLeft , clientWidth , clientHeight

getComputedStyle() ( currentStyle in IE )

```
当获取以上的属性和方法时 浏览器为了获取最新的布局信息 不得不立即触发重排以返回正确的值

###### 最小化重绘和重排
重绘和重排代价很昂贵 因此一个号的提高程序响应熟读的策略就是减少此类操作的发生

###### 优化方式

1. 合并多次对样式属性的操作

```
思考
var el = document.getElementById('mydiv');
el.style.borderLeft = '1px';
el.style.borderRight = '2px';
el.style.padding = '5px';

即使有浏览器有重排机制优化 但最坏的情况也是进行三次重排

修改后

var el = document.getElementById('mydiv');
el.style.cssText = 'border-left: 1px; border-right: 2px; padding: 5px;';
或者

var el = document.getElementById('mydiv');
el.className = 'active';

```
2. 批量修改dom
当需要对dom元素进行一系列的操作时候 可以通过以下的步骤来减少重绘和重排的次数

<font color=green> * 使元素脱离文本流</font>
<font color=green> * 操作元素</font>
<font color=green> * 操作完成后 将元素带回文档中</font>
这样儿 只有第一步和第三部触发两次重排

有三种方式可以实现上面的步骤


<font color=red> 1.隐藏元素（display:none）操作元素 重新展示</font>

```
    var ul = document.getElementById('mylist');
    ul.style.display = 'none';
    appendDataToElement(ul, data);
    ul.style.display = 'block';
    
```
<font color=red> 2.使用文档片段（document fragment）在当前 DOM 之外构建一个子树，再把它拷贝回文档
</font>

```
 var fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
document.getElementById('mylist').appendChild(fragment);   
```
<font color=red> 3.将原始元素拷贝到一个脱离文档的节点中，修改副本，完成后再替换原始元素
</font>

```
var old = document.getElementById('mylist');
var clone = old.cloneNode(true);
appendDataToElement(clone, data);
old.parentNode.replaceChild(clone, old); 
```
**总结：**推荐尽可能的使用文档片段（第二个方案），因为它们所产生的 DOM 遍历和重排次数最少。唯一潜在的问题是文档片段未被充分利用，很多人可能并不熟悉这项技术。

3. 缓存布局信息

```
浏览器获取元素的offsetLeft等属性值时会导致重排 将需要获取的保护局信息的属性值 赋值给变量 然后再操作变量
```
4. 定位

```
将需要多次重排的元素，position 属性设置为 absolute 或 fixed，这样元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。
```

<font color=red>操作DOM具体的成本，说到底是造成浏览器重排和重绘，从而消耗GPU资源</font>













   

 


s