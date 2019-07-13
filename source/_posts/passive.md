---
title: addEventListener 中的 passive 用法
date: 2018-12-21 21:40:40
categories:
  - js
---

## 引出问题
一个很简单的需求 页面有一张小图 点击是个swiper实现的图集 同时有一个灰色的蒙层 蒙层底部页面不可滑动 关闭蒙层 页面可恢复正常

实现方式

```
function bodyScroll(event){
    event.preventDefault();
}
function _switchTag(type) {
    if (type === 'on') {
        window.addEventListener('touchmove', bodyScroll);
    } else {
        window.removeEventListener('touchmove', bodyScroll);
    }
}

_switchTag(on) 页面不可滑动
_switchTag(off) 页面恢复滑动
```
移动端的效果如下
android
![jsworke](/images/passive/passive1.gif)

ios
![jsworke](/images/passive/iOSqian.gif)

我不知道为什么无效 直到我在模拟器上看到了

![jsworke](/images/passive/webqian.gif)

啊哦 报错了🦢

<font color="red">Unable to preventDefault inside passive event listener due to target being treated as passive</font>
来自 google 的解释 https://developers.google.com/web/updates/2017/01/scrolling-intervention

大概的意思是说


```
由于浏览器必须要在执行事件处理函数之后，才能知道有没有调用过 preventDefault() ，这就导致了浏览器不能及时响应滚动，略有延迟。

所以为了让页面滚动的效果如丝般顺滑，从 chrome56 开始，在 window、document 和 body 上注册的 touchstart 和 touchmove 事件处理函数，会默认为是 passive: true。浏览器忽略 preventDefault() 就可以第一时间滚动了。

举例：
wnidow.addEventListener('touchmove', func) 效果和下面一句一样
wnidow.addEventListener('touchmove', func, { passive: true })
```
这就导致了这个问题


```
如果在以上这 3 个元素的 touchstart 和 touchmove 事件处理函数中调用 e.preventDefault() ，会被浏览器忽略掉，并不会阻止默认行为
```
所以出现了以上视频中问题

### 解决方案

那么我们如何来解决这个问题 即不要让浏览器忽略掉e.preventDefault()？


1. window.addEventListener(‘touchmove’, func, { passive: false })

设置passive: false之后的结果
android
![jsworke](/images/passive/androdhou.gif)
ios
![jsworke](/images/passive/iOShou.gif)
浏览器
![jsworke](/images/passive/webhou.gif)
问题完美解决

```
你看到这里可以结束了 如果你还想再了解一点点
👇👇👇👇

#### 你可能不知道的addEventListener

很久之前addEventListener的参数是这样儿的
`addEventListener(type, listener, useCapture)`

后来也就是控制监听器是在捕获阶段执行还是在冒泡阶段执行的 useCapture 参数，变成了可选参数
`addEventListener(type, listener [,useCapture])`

再后来 DOM 规范做了修订addEventListener() 的第三个参数可以是个对象值了，也就是说第三个参数现在可以是两种类型的值了 变成这样儿式儿的

```
addEventListener(type, listener[, useCapture ])
addEventListener(type, listener[, options ])
```
扩展新的选项，从而自定义更多的行为，目前规范中 options 对象可用的属性有三个：


```
addEventListener(type, listener, {
    capture: false, 等价于 useCapture 默认值false
    passive: false, 是否让阻止默认事件失效 true:失效 false：不失效
    once: false //表明该监听器是一次性的，执行一次后就被自动 removeEventListener 掉，还没有浏览器实现它 默认值false
})
```

还想再说一点 那我设置了 passive的事件 这么移除呢 这里给出了方法

```
你可以直接省略第三个参数
window.removeEventListener(‘touchmove’, func)

如果添加了 第一个参数 capture 可以这样移除

window.removeEventListener(‘touchmove’, func, true)
window.removeEventListener(‘touchmove’, func, {capture :true})

### 为什么会有 passive这个概念

像这样儿的代码

```
document.addEventListener("touchstart", function(e){
    ... // 浏览器不知道这里会不会有 e.preventDefault()
})

```

由于 touchstart 事件对象的 cancelable 属性为 true，也就是说它的默认行为可以被监听器通过 preventDefault() 方法阻止，那它的默认行为是什么呢，通常来说就是滚动当前页面（还可能是缩放页面），如果它的默认行为被阻止了，页面就必须静止不动。但浏览器无法预先知道一个监听器会不会调用 preventDefault()，它能做的只有等监听器执行完后再去执行默认行为，而监听器执行是要耗时的，有些甚至耗时很明显，这样就会导致页面卡顿。视频里也说了，即便监听器是个空函数，也会产生一定的卡顿，毕竟空函数的执行也会耗时。

有 80% 的滚动事件监听器是不会阻止默认行为的，也就是说大部分情况下，浏览器是白等了。所以，passive 监听器诞生了，passive 的意思是“顺从的”，表示它不会对事件的默认行为说 no，浏览器知道了一个监听器是 passive 的，它就可以在两个线程里同时执行监听器中的 JavaScript 代码和浏览器的默认行为了




