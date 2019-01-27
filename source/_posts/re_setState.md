---
title: 揭秘setState机制
date: 2019-01-27 19:34:45
tags:
- setState
categories:
- 前端,react
---


## 开篇
要说`React`设计体现了响应式编程思想
> UI=f(state) 

[程墨Morgan](https://www.zhihu.com/people/morgancheng/activities)的总结真是恰到好处

`state`于`React`很重要，`setState`作为管理`state`的重要方法自然也是头等公民

当然了，如果只是简单用法，API足够了，你知道如何设置，如何更新，或许能解决眼前的需求，但是需求稍微复杂一点，可能会被动陷入`setState`怪圈

在哪里跌倒就把哪里买下来，被坑了之后，痛定思痛决定研究下`setState`怪象，最后发现 react真是博大精深

<font color="red">文章太太太长了，不太感兴趣的话，可以拉到最后总结，也能避免入坑</font>

## 引出问题

**如果面试官问你**

1. setState 是同步的还是异步的？
2. 它可能是同步的吗？如果可能的话如何实现同步更新state？


如果是看这篇文章之前 我的答案可能是这样儿
1. 它是异步的 不能立马拿到结果
2. emm....

## 解决问题
<!--最好的方式就是带着疑问看源码（基于版本<font color="red">16.4.1</font>），相信我 调试大法好-->

可能会有疑问的几个问题.
### 1. 为什么要用setState更新state不能直接修改？
先更正一个观点 是可以通过`this.state`对象修改state的 也确实能改变状态，但是不能驱动ui更新（不走render）那.. 有什么意义
### 2. setState是同步的还是异步的？
setState是同步执行的 但是state并不一定会同步更新（异步更新）,这是结论

针对这个问题 让我们带着问题去源码（基于版本<font color="red">16.4.1</font>）中寻找答案

-------

#### <font color=green>**场景一（合成事件中调用setState）**</font>

点击按钮实现加一的操作

```
...
onClick = ()=> {
   this.setState({
       val: this.state.val + 1
   })
   console.log('onClick', this.state.val);
 }
 ...
 render(){
console.log('render', this.state.val);
 <button onClick={this.onClick}>加1</button>
 }
 

```
首先 这是<font color=red>合成事件</font>中操作`state`,

**了解什么是合成事件？**
> react为了解决跨平台，兼容性问题，自己封装了一套事件机制，代理了原生的事件，像在jsx中常见的onClick、onChange这些都是合成事件

盯着这张网络盗图，画的完全符合我的预期，先有个概览，下文会跟踪解释

![setState的更新过程](/images/setState/setState__.jpg)

当你点击`onClick`时

* 第一阶段 （对合成事件的前期处理）

react会在正式`setState`前执行一堆pre钩子函数(<font color="red">大部分都不需要关心</font>),你可以在chrome调试台或者`console.trace()`追踪到
像这样儿👇
![trace](/images/setState/trace.png)
这些都是react的前期处理 这个过程你只需要关心👇

![pre钩子](/images/setState/pre11.png)

得到的信息
isBatchingUpdates默认为false,在这里(某一个pre钩子函数中被置为true了)，划重点，以后要考的 此时程序走的是<font color="red">try分支</font>

至此，前期处理结束

* 第二阶段（setState事件的处理）

红框部分是`setState`逻辑的调用栈，执行到`requestWork`方法时会二次判断 如下图👇
![sasas](/images/setState/sasas.png)

二阶段完

* 第三阶段（重置post钩子,更新state 渲染ui操作）

接上文，那么我想知道它会return到哪里呢
通过追踪发现 会走你主进程里的`console`语句
![state延迟更新](/images/setState/state_delay.png)
**这也就是为什么说setState更异步的**
上文中提到我们现在仍然处在try分支中，而在 finally 中才会更新`state`并且渲染到UI上，此时我们得到的仍然是`更新前的 state` 值，这就导致了`所谓的"异步"`,为什么要用`""`号,因为这并不是我们的本意，是`react为了性能着想私自做的处理`

-------

接着往下走：
![ssssww](/images/setState/sssswww.png)
最终还会走到这个方法
这个方法里面有个 try finally 语法, 到这里 我们知道了原来return到了这个fn
继续👇
![post钩子](/images/setState/post0.png)
![ui](/images/setState/ui.png)

render之后 合成事件中setState的逻辑就结束了

-------

#### <font color=green>**场景二（生命周期中调用setState）**</font>

实际上与场景一只在第一阶段不同（没有对合成事件的各种处理）我们可能想在不同的生命周期中聊下state的更新状况

我们照例先来个小demo 再来分析为什么会有这样儿的结果

```
state = {
   val:0,
   name:'yishu',
   age:18
}
 componentWillMount(){
      this.setState({val:this.state.val+1});
      console.log('componentWillMount第一次输出',this.state.val)
      this.setState({val:this.state.val+1});
      console.log('componentWillMount第二次输出',this.state.val)
    }
     componentDidMount(){
      this.setState({
            val: this.state.val + 1,
            name:'🐶'
        });
        console.log('componentDidMount第一次执行', this.state.val)
        this.setState({
            val: this.state.val + 1,
            name:'🐱',
            age:81
        });
        console.log('componentDidMount第二次输出', this.state.val)
     }
     
     componentDidUpdate(){
        console.log('componentDidUpdate执行',this.state.val)
    }
render(){
      if(this.state.val !==0){
            console.log('render执行',this.state)
        }
  <p>当前的值:{this.state.val}</p>
  <p>你的名字:{this.state.name}</p>
  <p>芳龄:{this.state.age}</p>
}

```

直接给出结果

![component_state](/images/setState/component_state.png)
其实经过各种API的熏陶，不运行也能知道结果,既然摊开来讲，就比较想知道为什么。

按照流程 ，先来一张调用栈鸟瞰图（网图）

![钩子函数中setState的调用栈](/images/setState/gouzihanshu.jpg)
还是三个阶段


* 第一阶段
![did](/images/setState/did.png)

* 第二阶段
![2121233434](/images/setState/2121233434.png)
到现在就比较清晰了，与场景一不同的是 走的是`requestWork`中的第一个`if`分支,场景一（第二个if分支）不管哪种情况 结果都是一致的 那就是不会走到下面的同步更新分支 并且 两种情况都还只是在**try**模块中执行 再往下的流程参考场景一

* 第三阶段
同场景一

关于这一场景 背着
<font color="red">`render`之前的生命周期拿到的都是更新之前的值，render之后执行的才能拿到最新的值 比如 `componentDidUpdate`</font>

-------

### 3. setState可以变成同步更新吗？ 如果可以的话 怎么做才能获取最新的值？

面试官这么问的话 通常都是可以 😂
那我们如何做
实践中一般每个人都有一到两种解决方案，这里列举中处理方案 并将在下面的文章中深入研究 为什么 它 可以.

* 在**原生事件**中调用`setState`函数
* 利用setState回调函数
* setTimeout等异步操作中调用`setState`函数
* 最近刚被种草的函数式setstate用法
* componentDidUpdate中获取

名词释义：
**原生事件**：指非react合成事件，原生自带的事件监听 `addEventListener` ，或者也可以用原生`js、jq`直接 `document.querySelector().onclick` 这种绑定事件的形式

-------

#### 在**原生事件**中调用`setState`函数


一个小demo

```
state = {
   val:0
}
componentDidMount = () =>{
 document.querySelector('#btn-raw').addEventListener('click', this.onClick);
}

onClick = ()=> {
   this.setState({
       val: this.state.val + 1
   });
   console.log('onClick', this.state.val);
 }
 
render(){
 if(this.state.val !==0){
            console.log('render执行',this.state)
        }
 <button id="btn-raw">Increment </button>
}
```
运行结果
![js](/images/setState/js.png)
意料之中

按理来说还需要张鸟瞰图 在这里 👇


![原生事件的setState](/images/setState/js0setState.jpg)

其实应该已经挺清晰了，我们还是在调试器里走一下

* 第一阶段（setState逻辑之前的处理）

![js_xzs](/images/setState/js_xzs.png)
画风突变，调用栈内不再执行一堆看不懂的pre钩子函数了
简单的理解为 用原生事件调用时 不受react控制了 也就没办法针对它执行一堆函数，直接走setState的逻辑了

* 第二阶段（setState执行逻辑）

![js999](/images/setState/js999.png)

走了第三个`if`分支 同步更新了（执行了render 再在click函数中打印的 所以拿到了最新值）

* 第三阶段

参考场景一

-------

#### 利用setState回调函数
API就不多说了
 
```
 this.setState({
       val:this.state.val + 1
        },()=>{
          console.log('cllback',this.state.val);
})
```
原理：回调函数被调用的时候，其实render函数已经被调用过了 参考`demo`的执行结果

-------
#### setTimeout等异步操作中调用`setState`函数

实际上 这并不是一个单独的场景，你可以在合成事件中 调用 `setTimeout` ，可以在生命周期中调用 `setTimeout` ，也可以在原生事件`setTimeout`，但是不管是哪个场景下，基于 `event loop`的模型下， setTimeout 中里去 setState 总能拿到最新的state值

一个demo 👇

```
this.state = {
       val:0,
       name:'yishu',
       age:18
   }
       componentDidMount(){
        // // debugger;
        // // 生命周期中调用---------------
        this.setState({
            val: this.state.val + 1,
            name:'🐶'
        });
        console.log('componentDidMount第一次执行', this.state.val)
        this.setState({
            val: this.state.val + 1,
            name:'🐱',
            age:81
        });
        console.log('componentDidMount第二次输出', this.state.val)

        setTimeout(() => {
            console.log('开始setTimeout', this.state.val)
            this.setState({
                val: this.state.val + 1
            });
            console.log('setTimeout第一次执行', this.state.val)
            this.setState({
                val: this.state.val + 1
            });
            console.log('setTimeout第二次执行', this.state.val)
        }, 0)

        this.setState({
            val: this.state.val + 1
        });
        console.log('componentDidMount第三次输出', this.state.val)        
    }
render(){
if(this.state.val !==0){
            console.log('render执行',this.state)
        }
 <p>当前的值:{this.state.val}</p>
}
```
![setTimeout](/images/setState/setTimeout.png)
上述结果，如果你了解<font color="red">js事件循环（ event loop）</font>的机制 就太简单了

默认你已经阅读过全文的上半部分 了解了 `setState`逻辑都是在**try**模块中执行的是在 try 代码块中,当你 try 代码块执行到 `setTimeout` 的时候，把它丢到**定时器触发线程(浏览器提供的线程)**去维护，并没有立即执行，先执行的 `finally` 代码块，等 finally 执行完了， `isBatchingUpdates` 又变为了 `false` ，导致最后去执行队列里的 setState 时候， requestWork 走的是和原生事件一样的 `expirationTime === Sync if`分支，所以表现就会和原生事件一样，可以同步拿到最新的`state`值。

关于js事件循环 又是另一个模块儿的东西 值得单开一章去研究，这里就不再细说了 感兴趣的同学 可移步
[js到底是如何工作的](http://maying.ink/2018/11/17/%E5%BD%BB%E5%BA%95%E7%90%86%E8%A7%A3js%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84/#more) 寻找答案

-------

### 4. 函数式setState用法
我在想 怎么才能更好引出这个大彩蛋，那便又涉及到另一个问题
以上文章中我们为了说明`setState执行的大流程` 有一个实际存在却没提及的问题，那就是`setState的批量更新`

类似这样儿的代码

```
      //...
      this.setState({val: this.state.val + 1});
      console.log('componentDidMount第一次执行', this.state.val)
      this.setState({val: this.state.val + 1});
      console.log('componentDidMount第二次执行', this.state.val)
      this.setState({val: this.state.val + 1});
      console.log('componentDidMount第三次执行', this.state.val)
      //...
```
得到的执行结果
![rende](/images/setState/render.png)

API告诉我可以接受，但是有点点懵 发生了什么... emm
假如这么写 会不会更清晰👇

```
  const currentCount = this.state.count;
  this.setState({count: currentCount + 1});
  this.setState({count: currentCount + 1});
  this.setState({count: currentCount + 1});
```
对的 你每次设置的都是同一个值
setState多次,re-render一次，多次同步执行的`setState`，会进行合并，类似于`Object.assign`,相同的`key`会被覆盖 所以结果为<font color="red">1</font>

为了不偏题 实现过程暂时省略,接受这个结论,让我们看函数式setState用法

<font color="red">如果传递给this.setState的参数`不是一个对象而是一个函数` 那就完全不一样儿了</font>

一个小🌰体验下神奇的效果

```
componentDidMound = () =>{
        console.log('参数为函数第一次执行',this.state.val)
        this.setState(this.increment);
        console.log('参数为函数第二次执行',this.state.val)
        this.setState(this.increment);
        console.log('参数为函数第三次执行',this.state.val)
        this.setState(this.increment);
        console.log('参数为函数第四次执行',this.state.val)
}
   /**
     * @description increment函数并不去修改组件状态，只是把“希望的状态改变”返回给React，维护状态这些苦力活完全交给React去做。
     * params  state：当前的state
     * params  props：当前的props
     * @return 对象代表之前你想给this.setState传递的参数 比如你认为第一次传0 第二次传1 
     * @memberof SetState
     */
    increment = (state, props) => {
        // 计算这个对象的方法有些改变，不再依赖于this.state，而是依赖于输入参数state。
        console.log('---state',state) 
        return {val: state.val + 1}; //同样是把状态中的count加1，但是状态的来源不是this.state，而是输入参数state。
      }
```
![fn_state](/images/setState/fn_state.png)
仔细看下代码中的注释部分
结论：
对于多次调用函数式setState的情况，React会保证调用每次increment时，state都已经合并了之前的状态修改结果。
但是 在`increment`函数被调用时`，this.state`并没有被改变，依然要等到`render`函数被重新执行时才被改变，也没有推翻上文的结论 真好。

API的设计符合函数式编程思想，开发者编写无副作用的函数 increment 并不会去改变组件状态，只是把“希望的状态改变”返回给React，维护状态这些苦力活完全交给React去做。
也正是 流程的控制权交给了React，所以React才能协调多个setState调用的关系。

### 4. 手动避免setState的不当调用带来的性能问题
到这里基本上就清楚了setState执行机制，那么 我们应该怎样儿利用自己的知识手动优化页面性能呢

这里列举了几条

* 除了特殊需求 尽量不要在非合成事件中调用`setState` 这样儿会失控 变成真的同步更新了 每次更新都会走render 走render就不可避免的涉及到 `diff对比`.. 更新过程很频繁，也就会导致性能问题。
* 不要在 `shouldComponentUpdate` 和 `componentWillUpdate` 中调用 setState 会造成循环
![xunhuan](/images/setState/xunhuan.jpg)


## 总结
* setState不会立刻改变React组件中state的值；
* setState通过引发一次组件的更新过程来引发重新绘制；
* 多次setState函数调用产生的效果会合并。
* setState 只在合成事件和钩子函数中是“异步”的，在原生事件和setTimeout 中都是同步的，批量更新的策略是基于"异步"之上的，在setTimeout和原生事件中是没有的，因为此时时不受控的
* setState 的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形成了所谓的“异步”，但是如果需要，也有解决方案可以直接拿到最新的值
* setState 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次setState，setState的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时setState多个不同的值，在更新时会对其进行合并批量更新。
* 像写受控组件那样儿去操作setState













