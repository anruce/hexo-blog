---
title: 彻底理解js是如何工作的
date: 2018-11-17 21:23:40
categories:
  - js
---

曾经 你一定遇到过类似这样儿的题目

```
console.log('script start')

setTimeout(function() {
    console.log('timer over')
}, 0)

Promise.resolve().then(function() {
    console.log('promise1')
}).then(function() {
    console.log('promise2')
})
console.log('script end')

输出结果：
script start
script end
promise1
promise2

```
如果你很轻松的答对并且能说出原理 那么恭喜你，倘若有些疑问，那么读完这篇文章，你一定会彻底搞懂它的运行原理。

<!-- more -->

### 首先  先了解几个概念

> JavaScript引擎

js引擎是执行js的程序或者解释器 我们常说的V8引擎就是一种js引擎的实现，其他的还有基于`java`开发的`Rhin` `Nashorn`等

V8引擎 使用在chrome 和node中，它由两部分组成

* 内存堆 ：这是内存分配发生的地方
* 调用栈： 这是你的代码执行时的地方

-------

> JS的特性

<font color=red>单线程 异步 非阻塞</font>

**js的单线程**
由于js的单线程

```
console.log('script start')
console.log('do something...')
console.log('script end')

// script start
// do something...
// script end

```
很好理解

那再来看

```
console.log('1')

console.log('2')

setTimeout(() => {
  console.log('3')
}, 1000)

// 点击页面
console.log('4')

console.log('5')
```
那么它的输出是什么呢？ 应该是`1，2，4，5，3 `？ 而不是` 1，2，3，4，5 `，js不是一行一行从上到下执行的吗 为啥会出现这种情况？

**这就是我们接下来要说的问题**

为什么不能同步执行？

如果一个任务的处理耗时（或者是等待）很久的话，如：网络请求、定时器、等待鼠标点击等，后面的任务也就会被阻塞,可能会出现白屏的情况 用户体验极其不友好

所幸的是 浏览器给我们提供了很多有用的`webapi`

如何优化？

js的单线程指的是浏览器中负责解释和执行js代码的只有一个线程 -js引擎线程 但是浏览器的渲染进程是提供多个线程的，遇到定时器 Dom事件或者是网络请求的任务的时候 js引擎会将他们交给webapi 也就是浏览器提供的相应线程去处理 而js引擎线程继续去处理后面的任务 这样儿实现了**异步非阻塞**
以下是日常线程：

* js引擎线程
* 事件触发线程
* 定时器触发线程
* 异步HTTP请求线程
* GUI渲染线程

图示的话 大概长下面这样儿
![jsworke](/images/jsWorker/jsworker.jpg)
所以 这里的 图例中的`setTimeout`会被分配到定时器触发线程去维护 去定时，时间一到 还是会把它的回调塞到`消息队列`等待

那么 到这里 又引出了两个问题
1.什么叫消息队列？
2.js引擎什么时候处理这个定时器 怎么处理？

JavaScript 通过事件循环（ event loop）的机制来解决这些问题
猜对了吗？
事件循环 机制和 消息队列 的维护是由**事件触发线程**（浏览器渲染引擎webapi之一）控制的

JS引擎线程 会维护一个 执行栈

1. 同步和异步任务进入不同的执行场所 同步的进入**主线程** 异步的进入Event Table并注册函数
2. 当指定的事情完成时 Event Table会将这个函数移入**Event Queue**
3. 主线程内的任务执行完毕为空，会去**Event Queue**读取对应的函数，进入主线程执行
4. 上述过程会不断重复，也就是常说的**Event Loop**(事件循环)。

言语太过苍白 举个🌰～


![jswork2](/images/jsWorker/jswork2.jpg)

所以 这个时候来看
setTimeout异步函数对应的回调函数( () => {} )会在执行栈为空，主代码块执行完了后才会执行

结果就不意外了吧

那么 像这样

```
setTimeout(() => {
  console.log('timer')
}, 0)
```
0延时的情况是啥意思呢
只是 timer的回调函数会立即加入消息队列而已，回调的执行还是得等执行栈为空（JS引擎线程空闲）时执行。

还没完～～

ES5中以上标准就够用了 但是ES6中新出了一些API 引出了一些新概念

> 宏任务与微任务

先来看一段代码 你能立刻说出它的执行结果吗

```
console.log('script start')

setTimeout(function() {
    console.log('timer over')
}, 0)

Promise.resolve().then(function() {
    console.log('promise1')
}).then(function() {
    console.log('promise2')
})

console.log('script end')

// script start
// script end
// promise1
// promise2
// timer over
```
What？ timer over 会在 promise1 promise2 之后执行？

好的 不要着急 往下看👇
所有任务分为 `宏任务` 和 `微任务`

* 宏任务（macrotask）：主代码块、setTimeout、setInterval等（可以看到，事件队列中的每一个事件都是一个 macrotask，现在称之为宏任务队列
* 微任务（microtask）：Promise、process.nextTick等 在microtask中 process.nextTick 优先级高于 Promise，它用来调度应在当前执行的脚本执行结束后立即执行的任务

事件（任务）队列和宏任务和微任务的联系：

* 一个事件循环有一个或者多个任务队列；
* 每个事件循环都有一个microtask队列
* macrotask队列就是我们常说的任务队列，microtask队列不是任务队列
* 一个任务可以被放入到macrotask队列，也可以放入microtask队列
* 当一个任务被放入microtask或者macrotask队列后，准备工作就已经结束，这时候可以开始执行任务了


js的执行规则：

1. 执行一个宏任务（栈中没有就从事件队列中获取）
2. 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
3. 宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）
4. 当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染
5. 渲染完毕后，JS引擎线程继续，开始下一个宏任务（从宏任务队列中获取）

所以 promise1 与 promise1属于微任务 会在第一个宏任务结束之后立即执行 而setTimeout即使延时为0 也是要等到下个事件循环去执行的😊

再简单点的话 那就

macro-task队列包含任务: a1, a2 , a3
micro-task队列包含任务: b1, b2 , b3

执行顺序为，首先执行marco-task队列开头的任务，也就是 a1 任务，执行完毕后，在执行micro-task队列里的所有任务，也就是依次执行b1, b2 , b3，执行完后清空micro-task中的任务，接着执行marco-task中的第二个任务，依次循环。

再简单点的话 那就.. 上图吧😄

![jsworke](/images/jsWorker/jsworker3.jpg)s

好的 理解的话 再来一个栗子 你可能继续懵逼


```
setTimeout(function(){console.log(1)},0);

new Promise(function(resolve,reject){
   console.log(2);
   resolve();
}).then(function(){console.log(3)
}).then(function(){console.log(4)});

process.nextTick(function(){console.log(5)});

console.log(6);
输出 2，6，5 ，3，4，1
```
定义promise的构造部分是同步的
如下
script(主程序代码)——>process.nextTick——>promise——>setTimeout

关于 process.nextTick()
插入到事件队列尾部，但在下次事件队列之前会执行。也就是说，它指定的任务总是发生在所有异步任务之前，当前主线程的末尾。
大致流程：当前”执行栈”的尾部–>下一次Event Loop（主线程读取”任务队列”）之前–>触发process指定的回调函数。
服务器端node提供的办法。用此方法可以用于处于异步延迟的问题。
可以理解为：此次不行，预约下次优先执行。

好的 再再来一个😂


```
setTimeout(function(){console.log(1)},0); （setTimeout1）

new Promise(function(resolve,reject){
   console.log(2);  （promise1）
   setTimeout(function(){resolve()},0) （setTimeout2）
}).then(function(){console.log(3)  （then1）
}).then(function(){console.log(4)}); （then2）

process.nextTick(function(){console.log(5)}); （nextTick）

console.log(6); （log）
输出： 2 6 5 1 3 4
```

区别在于promise的构造中，没有同步的resolve，因此promise.then在当前的执行队列中是不存在的，只有promise从pending转移到resolve，才会有then方法，而这个resolve是在一个setTimout时间中完成的，因此3,4最后输出。

写到这里 想到一个某位大师的很形象的例子

事件循环队列就类似于游乐园游戏，玩过了一个游戏之后 你需要到队尾去排队才能再玩一次 而任务队列类似w玩过了这个游戏之后 插队接着玩
看到这里 文章开头的题目应该不成问题了 甚至还觉得so easy

好的 到这里 就完了 下面是两个js运行时的概念 你可以傲娇的略过

-------
> js执行时

js调用栈

拥有 LIFO（后进先出）数据结构的栈，被用来存储代码运行时创建的所有**执行上下文**，记录了我们在程序中的位置  如果我们运行到一个函数 它就会将其放到栈顶 当从这个函数返回的时候，就会将这个函数从栈顶弹出，这就是调用栈做的事情

当 `JavaScript` 引擎第一次遇到你的脚本时，它会创建一个全局的执行上下文并且压入当前执行栈。每当引擎遇到一个函数调用，它会为该函数创建一个新的执行上下文并压入栈的顶部。

引擎会执行那些执行上下文位于栈顶的函数。当该函数执行结束时，执行上下文从栈中弹出，控制流程到达当前栈中的下一个上下文

-------

> js执行上下文

执行上下文是评估和执行js代码的环境的抽象概念
js代码在执行的时候 它都是在执行上下文中运行

它分为三种类型
1. 全局执行上下文

```
它会执行两件事： 创建一个全局window对象 并且设置this的值等于这个全局对象 一个程序中只会有一个全局执行上下文
```
2. 函数执行上下文

```
函数被调用时候 会创建上下文，函数上下文可以有任意多个  每当一个新的执行上下文被创建 它会按定义的顺序 执行一系列步骤
```
3. eval函数执行上下文

```
执行在eval函数内部的代码会有他自己的执行上下文
```

**js如何创建执行上下文**

两个阶段

* 创建阶段
* 执行阶段


代码执行栈 执行上下文经历创建阶段 会发生
1. this值的制定
2. 创建词法环境组件
3. 创建变量环境组件

-------

最后的最后

JavaScript 是单线程语言，决定于它的设计最初是用来处理浏览器网页的交互。浏览器负责解释和执行 JavaScript 的线程只有一个（所以说是单线程），即JS引擎线程，但是浏览器同样提供其他线程，如：事件触发线程、定时器触发线程等












