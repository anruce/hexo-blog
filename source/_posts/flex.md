---
title: 深入理解Flexbox
date: 2019-03-14 19:08:59
tags:
- flex
categories:
- CSS
---

## 前言
过去，我们总是不得不忍受`float、display:table`这些布局方式带来的痛苦，不过现在是时候去拥抱一个更简洁的制作智能布局的现代语法 `Flexbox`
## Flexbox是什么
根据规范中的描述可知道，`Flexbox`模块提供了一个有效的布局方式，即使不知道视窗大小或者未知元素情况之下都可以智能的，灵活的调整和分配元素和空间两者之间的关系


<!-- more -->
## 如何开始使用Flexbox
幸运的是，入门超级简单
你要做的第一件事就是声明一个`Flex`容器

就像这样儿，声明了`Flex`容器之后，一个`Flexbox`格式化上下文就立即启动了

```
<ul class="oul">
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```
正常情况下div在CSS中垂直堆栈的，也就是说从上到下排列显示

图一
![flex2](/images/flex/flex2.png)

声明`Flex容器`

```
.oul {
    //...
    display: flex
    //...
}
```

现在已经是一个Flexbox格式化上下文
图二
![flex1](/images/flex/flex1.png)

**很简单 一行代码就能看到布局改变了子元素就像你使用了float一样是水平排列的**

拿这个例子来说此时`ul`自动变成了`Flex`，而 `li`变成了`Flex`项目

记住这些名词，它们是`Flexbox`模块的基础

## Flex容器属性
`flex-direction || flex-wrap || flex-flow || justify-content || align-items || align-content
`
解释这些属性之前，先来看一张flex世界比较重要的概念

![flex3](/images/flex/flex3.png)

* `Main-Axis`就是水平方向，从左到右，这也是默认方向
* `Cross-Axis`是垂直方向，从上往下

### flex-direction
属性值

```css
ul { flex-direction: row || column || row-reverse || column-reverse; }
```
默认值是`row`它让Flex项目沿着Main-Axis排列（从左向右，水平排列） 这也解释了图二的效果
### flex-wrap
属性值

```css
ul { flex-wrap: wrap || nowrap || wrap-reverse; }

*  nowrap: Flex容器内的Flex项目不换行排列 （默认值）
*  wrap:换行排列 这种情况下，一行不能包含所有列表项的默认宽度，它们就会多行排列
*  wrap-reverse:反向换行
```

### flex-flow

```css
flex-flow是flex-direction和flex-wrap两个属性的速记属性
```
语法
```css
ul { flex-flow: row wrap; }
```

### justify-content
接下来感受来自`flex`容器的魔法

它主要定义了Flex项目在`Main-Axis`上的对齐方式

属性值

```css
ul { justify-content: flex-start || flex-end || center || space-between || space-around }
* flex-start 元素位于容器的开头。弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
* flex-end 元素位于容器的结尾，弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界
* center：居中对齐
* space-between：让除了第一个和最一个Flex项目的两者间间距相同（两端对齐）
* space-around：让每个Flex项目具有相同的空间
```

**space-between**
![between](/images/flex/between.png)
**space-around**

和space-between有点不同，第一个Flex项目和最后一个Flex项目距Main-Axis开始边缘和结束边缘的的间距是其他相邻Flex项目间距的一半
![around](/images/flex/around.png)
### align-items
它主要用来控制Flex项目在侧轴上的对齐方式

```css
ul { align-items: flex-start || flex-end || center || stretch || baseline }

* stretch 默认值 让所有的Flex项目高度和Flex容器高度一样。
* flex-start 元素位于容器的开头。弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
* flex-end 元素位于容器的结尾，弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界
* center 元素位于容器的中心
* baseline 让所有Flex项目在Cross-Axis上沿着他们自己的基线对齐
```

**baseline**
效果类似`flex-start`但略有不同
区别就在于`baseline`
![base](/images/flex/base.png)
### align-content

```
ul { align-items: stretch|center|flex-start|flex-end|space-between|space-around}

* stretch 默认值 元素被拉伸以适应容器
* flex-start 元素位于容器的开头。弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
* flex-end 元素位于容器的结尾，弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界
* center 元素位于容器的中心
* space-between：让除了第一个和最一个Flex项目的两者间间距相同（两端对齐）
* space-around：让每个Flex项目具有相同的空间
```
**stretch**
![align-content](/images/flex/align-content.png)
flex-end
![ennd](/images/flex/ennd.png)

**flex-start**

![align-content-start](/images/flex/align-content-start.png)

**center**
![cente](/images/flex/center.png)

## Flex项目属性

```css
order || flex-grow || flex-shrink || flex-basis
```

### order
允许Flex项目在一个Flex容器中重新排序。基本上，你可以改变Flex项目的顺序，从一个位置移动到另一个地方而不改变源代码，所有Flex项目的order值都是0，Flex项目会根据order值从低到高重新排序

```css
.oul li:nth-child(1) {
    order: 1;
    /*设置一个比0更大的值*/
}
```
Flex项目2、3、4的order值为0。现在Flex项目1的order值为1

![orde](/images/flex/order.png)

Flex项目`2、3和4的order`值都是`0`。HTML源代码秩序并没有修改过。如果给`Flex`项目2的`order`设置为`2`
![orde](/images/flex/order2.png)

可见 它也增加堆栈。现在代表`Flex`项目的最高的`order`值

当两个Flex项目具有相同的order值呢？在下面的示例中，把Flex项目1和3设置相同的order值。
![orde](/images/flex/order3.png)
现在仍是从低到高排列。这次`Flex`项目`3`排在`Flex`项目`1`后面，那是因为在`HTML`文档中`Flex`项目`3`出现在`Flex`项目`1`后面。

如果两个以下`Flex`项目有相同的`order`值时，`Flex`项目重新排序是基于`HTML`源文件的位置进行排序


### flex-grow 和 flex-shrink

* flex-grow ：控制`Flex`项目在容器有多余的空间如何放大（扩展）默认值是0 表示开关是关闭的，即如果存在剩余空间，也不放大

```css
.item {
  flex-grow: <number>; /* default 0 */
}
```

![flow-gro](/images/flex/flow-grow.png)
属性值为1时
![flow-gro](/images/flex/flow-grow1.png)

现在`Flex`项目扩展了，占据了Flex容器所有可用空间。也就是说开关打开了。如果你试着调整你浏览器的大小，`Flex`项目也会缩小，以适应新的屏幕宽度

如果所有项目的`flex-grow`属性都为`1`，则它们将等分剩余空间（如果有的话）
如果一个项目的`flex-grow`属性为`2`，其他项目都为`1`，则前者占据的剩余空间将比其他项多一倍

* flex-shrink：属性定义了项目的缩小比例，默认为`1`（默认开启），即如果空间不足，该项目将缩小

```css
.item {
  flex-shrink: <number>; /* default 1 */
}
```

### flex-basis
它用于设置或检索弹性盒伸缩基准值
`浏览器`根据这个属性，计算主轴是否有多余空间一般配合 `flex-wrap `一起使用，flex容器根据 `flex-basis` 计算是否需要换行


一些特性

#### 1.它的属性值可以是**长度单位(em || rem || px)**或**百分比(%)**，<font color="red">百分比是按照父元素的width为标准</font>
#### 2.默认值为 <font color="red">auto</font>  [MDN](https://drafts.csswg.org/css-flexbox/#flexibility)

 ```
取值为**auto**时，它的值就等于当前项的**width**（或者默认的大小，width没有设置的话）" 
flex-basis:auto" 的含义是 "参照我的width和height属性
 ```
#### 当flex-item没有自身宽高，其默认大小由flex-basis决定
即优先级： <font color="red">flex-basis > width(非auto)</font>

```
.oul2-li{
      flex-basis: 200px;
      width: 10px;
      margin: 0px 4px;
      background: red;
  }
```
![shiliflex1](/images/flex/shiliflex1.png)
#### 4.当元素存在默认宽高（<font color="red">input</font>）
并且设置了 `flex-basis`，那么它的初始大小`以固定宽高为下限`，如果`flex-basis `超过了固定宽高，那么以`flex-basis`设置大小为准，如果`flex-basis`比固定宽高小，那么以固定宽高为准

```
.Myinput {
     background: greenyellow;
     flex-basis: 200px;
 }
```
![gfdsdf](/images/flex/gfdsdf.png)
当将`flex-basis`设置的比默认宽度大

![swqs](/images/flex/swqs.png)

当将`flex-basis`设置的比默认宽度小 100
![jnjjjuuu](/images/flex/jnjjjuuu.png)
####  5.当元素存在 min-width[height] 或者 max-width[height]

```
如果 `flex-basis` 的值 大于 `min-width[min-height]`，`flex-item content`的值为 `flex-basis`
如果`flex-basis `的值小于` min-width[min-height]` 那么`flex-item content`以`min-width[min-height]`计算
```

![jsjjsjsjjjsjsjs](/images/flex/jsjjsjsjjjsjsjs.png)
#### 6.元素设置`width[height]: auto;`
CSS解析器对比两者的值，两者**谁大取谁**作为`item`的基本尺寸，如果一个`item`没有内容，flex-item 初始大小就会以`flex-basis`来决定
但是如果`item`有了内容，且内容撑开的尺寸比`flex-basis`大，那么`flex-item`初始大小就会以`width[height]: auto; `来决定<br/>
优先级：<font color="red">width[height]: auto == flex-basis</font>

```
 <ul class="oul2">
      <li class="li1">666666666666666666666666666666666666666666666666666666666666666666666666666666666666</li>
      <li class="li2">77777</li>
    </ul>
    
    .li1{
    flex-basis: 100px;
    margin-right: 5px;
    background: greenyellow;
    width: auto;
}
.li2{
    flex-basis: 300px;
    width: auto;
    background: red;
}

.oul2{
    display: flex;
    list-style: none;
    border: 1px solid #ccc;
}
```
![uuxsuudhdusd](/images/flex/uuxsuudhdusd.png)


## 绝对和相对Flex项目
绝对Flex项目的宽度只基于 flex 属性，而相对Flex项目的宽度基于内容大小

**相对项目**

```css
 .oul {
      display: flex;
      /*触发弹性盒*/
  }

  .oul li {
      //flex-basis: auto;
      flex: auto; /*记住这与 flex: 1 1 auto; 相同*/
      border: 2px solid red;
      margin: 2rem;
  }
```

![xianngdui](/images/flex/xianngdui.png)

Flex项目的初始宽度是被自动计算的（`flex-basis: auto`），然后会伸展以适应可用空间（`flex-grow: 1`）

像这样 当`Flex`项目因为被设置为 `flex-basis: auto`，而导致宽度被自动计算时，是基于`Flex`项目内包含的内容的大小而计算的就是相对项目

**绝对项目**


```css
.oul li {
     flex-grow: 1;
     flex-shrink: 1;
     flex-basis: 0;
     /*与 flex: 1 1 0 相同*/
    border: 2px solid red;
    margin: 2rem;
  }
```

![juedui](/images/flex/juedui.png)

`Flex`项目的初始宽度是零（`flex-basis: 0`），并且它们会伸展以适应可用空间。当有两到多个`Flex`项目的 `flex-basis` 取值为`0`时，它们会基于`flex-grow`值共享可用空间


### flex组合属性
flex是`flex-grow、flex-shrink和flex-basis`三个属性的速记（简写）顺序缩写为 GSB 

#### 一些取值规律

**当 flex 取值为一个非负数字，则该数字为 flex-grow 值，flex-shrink 取 1，flex-basis 取 0%，例如：flex: 1; 相当于**

```
li {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: 0%;
}
```
⚠️ `flex-basis的`默认值为`auto`，为什么此时是`0%`？
当你创建一个flexbox上下文而不给flex项目设置任何属性，此时的默认值
![wwwaaaa111](/images/flex/wwwaaaa111.png)
此时它是<font color="red">相对项目</font>
一旦你设置了`flex:1`简写属性

参考MDN所说
![flexjianxie](/images/flex/flexjianxie.png)
浏览器使其变成了<font color="red">绝对项目</font>

**flex: auto;**

```
li {
    
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: auto;
}
```
**当 flex 取值为一个长度或百分比，则视为 flex-basis 值，flex-grow 取 1，flex-shrink 取 1，有如下等同情况（注意 0% 是一个百分比而不是一个非负数字）**

```
.item-1 {flex: 0%;}
.item-1 {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: 0%;
}
.item-2 {flex: 24px;}
.item-1 {
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: 24px;
}
```
**当 flex 取值为两个非负数字，则分别视为 flex-grow 和 flex-shrink 的值，flex-basis 取 0%，如下是等同的：**

```
.item {flex: 2 3;}
.item {
    flex-grow: 2;
    flex-shrink: 3;
    flex-basis: 0%;
}
```
**当 flex 取值为一个非负数字和一个长度或百分比，则分别视为 flex-grow 和 flex-basis 的值，flex-shrink 取 1，如下是等同的：**

```
.item {flex: 2333 3222px;}
.item {
    flex-grow: 2333;
    flex-shrink: 1;
    flex-basis: 3222px;
}
```


## 深入flex
到这里 关于`Flex`的基础知识已经结束了,你可以用它们处理几乎任何问题
但是
<font color="red">Flexbox是如何弹性的计算子级项目的大小的，它有没有什么规则 </font>
令我费解

好了，小🌻课堂开始了

flex是应用在X轴和Y轴上的
每一根轴都包括三个东西 ` 维度、方向、尺寸`

* 维度：子元素横着排(X轴)还是竖着排(Y轴)
* 方向：子元素的顺序(顺序还是逆序)
* 尺寸：即父元素的`width`，子元素在当前轴方向所占的位置的总和

如下图所示（来自W3C规范）

![w3](/images/flex/w3c.png)

## FFC(flex formatting context)
Flexbox 布局新定义了格式化上下文，类似 BFC（block formatting context）
定义了`display: flex;` 或 `display: inline-flex`的元素，和`BFC`一样，不会被浮动的元素遮盖，不会垂直外边距坍塌等等

## 与BFC的细微区别

* vertical-align 对 Flexbox 中的子元素 是没有效果的
* float 和 clear 属性对 Flexbox 中的子元素是没有效果的，也不会使子元素脱离文档流(但是对Flexbox 是有效果的！)
* Flexbox 下的子元素不会继承父级容器的宽

## flex item（flex 子元素）
CSS解析器会把 定义了 `display: flex;` 和 `display: inline-flex;` 的 `Flexbox `下的子元素外部装进一个看不见的盒子里，我们通过排列这些盒子来达到排序、布局、 伸缩的目的

### flex-item-size 是如何计算的
子元素的尺寸为主轴方向上元素的的自身宽度 再加上自身的`margin 、 border 和 padding `

W3C规范中介绍了  `flex-item content` 的计算规则

### 隐藏属性对 items-size 的影响

针对 `display: none; visibility: hidden; visibility: collapse; transform: scale;`进行测试

结论
* 如果设置了 `visibility: hidden; | visibility: collapse; | transform: scale;`的`flex-item content `依然被算进主轴尺寸，CSS 解析器依然将可用空间分配给他们
* 如果设置了`display: none;` CSS解析器不会对该`item`的空间进行计算

### 关于position: absolute 对item影响

`position: absolute `也是适用 `Flexbox` 中的子元素的，并且，设置了`position: absolute`属性的子元素，也会受到 `Flexbox` 排列的影响


absolute 的子元素重叠在了一起，但是依然会受到 `align-items: center; `的影响而居中

![jjjjjdsd](/images/flex/jjjjjdsd.png)

并且根据一系列的实验得知

`flexbox` 下设置了`absolute`：

* flexbox 流下面的 `justify-content` 和 `align-items`
* `item`的 `top、left、right、bottom`
* `margin`自始至终都会影响`item`的位置
* 脱离了文档流的 `item` 不会影响正常的`flex` 布局

**小结**
`justify-content、align-items` 和 `top、left、right、bottom `都是位置属性 且 `top、left、right、bottom `的值会覆盖 `justify-content、align-items`的值

`margin `的优先级是和 `top、left、right、bottom` 一样的，也就是说 `margin 和 top、left、right、bottom`所设置的值会**同时生效**


## flex-basis、flex-grow、flex-shrink 以及相应的计算

这三个属性只有父级元素设置了 `display: flex | inline-flex;` 才会生效，并且只针对主轴方向生效

* 如果 主轴是水平的，即 `flex-direction: row`; 那么 `flex-basis、flex-grow、flex-shrink `控制的就是单个`item`的宽度
* 如果 主轴是垂直的，即 `flex-direction: column`; 那么 `flex-basis、flex-grow、flex-shrink` 控制的就是单个item的高度

-------


那么所有`items`都会在主轴方向上的一条线上排列，`CSS解析器`会计算` items `在主轴方向上所占的空间 相对于` Flexbox `在主轴方向的所占的空间进行比较计算

* 如果 `items` 所占的空间是小于`Flexbox`的 那么说明`Flexbox` 还没有填满，`CSS解析器`就会计算还有多少空间没有填满，根据每一个`item`所设置的`flex-grow` 设置的值，将这些空间分配按比例分配给每一个`item`

![wwwqwq](/images/flex/wwwqwq.png)

* 如果 `items` 所占的空间是大于`Flexbox`的 那么说明`Flexbox` 被填满了，`CSS解析器`就会计算超出了多少空间，根据每一个`item`所设置的`flex-shrink`设置的值，将这些空间分配按比例缩小每一个`item`
 ![fukeyongkongjian](/images/flex/fukeyongkongjian.png)


### 超出的空间是如何计算的

**`flow-grow`的计算流程**

```
可用空间 = 将flexbox-content - 每个item-size的总和
```

将元素设置的`flow-grow`值加起来设置为 `growSize`
`单位分配空间 = 可用空间/growSize`
然后真正分配的时候根据自己的比例计算增加的值
`应该增加的值 = 自己的grow值 *  单位分配空间 `

![growfencd](/images/flex/growfencdc.png)


**`flow-shrink`的计算流程**

它的流程与`flow-grow`的计算流程<font color="red">不同</font>

```css
shrink比例 = flex-shrink * item-size / 之前的总和

应该缩减的值：超出的空间 - shrink比例 * item-size
```

如图所示

![shrinks](/images/flex/shrinksw.png)


**max-width[height] 情况下 flex-grow 的计算流程**

由于可能存在某一个或多个`item` 设置了有`max-width[height]`。所以，CSS引擎会先进行一次分配，分配后，统计那些有`max-width[height]的items`, 分配后是否有超出的剩余空间，然后对这些剩余空间再分配给那些没有设置`max-width[height]`的`item`

![sandjandjnsjdwednjsedf](/images/flex/sandjandjnsjdwednjsedf.png)

**min-width[height] 情况下 flex-shrink**

由于可能存在某一个或多个item 设置了有`min-width[height]`。所以，`CSS引擎`会先进行一次 `shrink`， `shrink`后，统计那些有`min-width[height]`的`items`, `shrink`后是否有的剩余的未`shrink`空间，然后对这些剩余空间再分配给那些没有设置`min-width[height]`的`item`


![iiisjindndnjsndjsdnxjksndj](/images/flex/iiisjindndnjsndjsdnxjksndj.png)

## Flexbox的浏览器支持

让我们求助于 [caniuse](https://caniuse.com/#search=flex)


## 总结
深入理解Flex 还是挺不容易的。


## 参考链接
[理解flexbox，你需要知道的一切](https://www.w3cplus.com/css3/understanding-flexbox-everything-you-need-to-know.html
)
[探索Flexbox](https://www.w3cplus.com/css3/flexbox-adventures.html)
[理解flex](https://www.w3cplus.com/css3/flexbox-layout-and-calculation.html?from=groupmessage)






