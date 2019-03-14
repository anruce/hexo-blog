---
title: 揭秘setState机制
date: 2018-11-23 16:40:40
categories:
  - react
---

前言：state是react中重要的概念， react是通过管理状态来实现对组件的管理，那么 react是如何控制组件的状态 又是如何利用状态来管理组件的呢？


我们所知道的版本 大概是 通过`this.state`来访问`state`，通过`setState()`方法来更新`state`，当`this.state()`被调用的时候 `React`会重新调用`render`方法来重新渲染`UI`

那好的 先来看一道题吧

```
export default class SetState extends React.Component {
    constructor(){
        super();
        this.state = {
            val:0
        }
    }

    componentWillMount(){
        this.setState({val:this.state.val+1});
        console.log('componentWillMount第一次输出',this.state.val)
        this.setState({val:this.state.val+1});
        console.log('componentWillMount第二次输出',this.state.val)
    }
    componentDidMount(){
        // debugger;
        this.setState({val:this.state.val+1});
        console.log('componentDidMount第一次输出',this.state.val)
        this.setState({val:this.state.val+1});
        console.log('componentDidMount第二次输出',this.state.val)
        setTimeout(()=>{
            // debugger;
            console.log('开始setTimeout',this.state.val)
            this.setState({val:this.state.val+1});
            console.log('第三次输出',this.state.val) 

            this.setState({val:this.state.val+1});
            console.log('第四次输出',this.state.val) 
        },0)
    }
   
    render(){
        return null;
    }
    
  }
```
这道题的答案是 `0 0 1 1 2 3 4`

假如结果与你心中的答案并不完全相同，那么你应该感兴趣这背后究竟发生了什么.

<!-- more -->
## 了解setState

1. setState是同步执行的 但是state并不一定会同步更新（异步更新）

```
实际上react的异步更新通过一个队列机制来实现，当执行state时 需要将更新的state合并后放入状态队列而不会立刻更新 队列机制可以高效的批量更新state 如果在非构造方法里更改值  类似 this.state.name='yishu' 是不会被放到状态队列中 当下次调用setState并对状态队列进行合并时 将会忽略它而造成无法预知的错误
```
2. setState在React生命周期和合成事件中批量覆盖执行
 
```
在React的生命周期钩子和合成事件中，多次执行setState，会批量执行，多次同步执行的setState，会进行合并，类似于Object.assign
```

3. setState在原生事件，setTimeout，setInterval，Promise等异步操作中，state会同步更新

```
当执行到 setTimeout 的时候，把它丢到列队里，并没有去执行，而是先执行的 finally 主进程代码块，等 finally 执行完了， isBatchingUpdates 又变为了 false ，导致最后去执行队列里的 setState 时候， requestWork 走的是和原生事件一样的 expirationTime === Sync if分支，所以表现就会和原生事件一样，可以同步拿到最新的state的值。
```

关于setState这个方法 源码记载

```

ReactComponent.prototype.setState = function(partialState, callback) {
  //...
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};

  enqueueSetState: function(publicInstance, partialState) {
  //...
    var queue =internalInstance._pendingStateQueue ||
      (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

    enqueueUpdate(internalInstance);
  },

```

setState方法实际上会执行 `enqueueSetState`方法 通过`_pendingStateQueue`更新队列进行合并操作 最终通过`enqueueUpdate`执行state更新

## setState调用栈

![](/images/setState/setState1.png)

如图：通过变量isBatchingUpdate 来决定当前是应该走批量更新 还是立即更新 为true时 说明当前在批量更新模式 为false的话 会立即更新

为了更好的理解 涉及到部分源码

enqueueUpdate 代码如下：

```
function enqueueUpdate(component) {

  // 如果不处于批量更新模式 更新
  if (!batchingStrategy.isBatchingUpdates) {
batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
  // 如果处于批量更新模式 则将该组件保存在 dirtyComponents 中
  dirtyComponents.push(component);
}

```

那么这个`batchingStrategy`究竟是做什么的？ 其实它只是一个简单的对象，定义了 isBatchingUpdates 和 batchedUpdates 方法 其中transaction.perform 的调用 涉及到了<font color="red">事务</font>的概念

```

// batchedUpdates 方法
var ReactDefaultBatchingStrategy = {
  isBatchingUpdates: false,

   batchedUpdates: function(callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;

    ReactDefaultBatchingStrategy.isBatchingUpdates = true;

    if (alreadyBatchingUpdates) {
      callback(a, b, c, d, e);
    } else {
      transaction.perform(callback, null, a, b, c, d, e);
    }
  },
};

```

## 事务机制
事务就是将需要执行的方法使用wrapper封装起来 再通过事务提供的perform方法执行
![](/images/setState/setState2.png)
执行 perform 之前 先执行  wrapper 中的init方法 执行完 perform之后 再执行 所有的close方法
假如有一个事务 test 执行顺序表现为 

`init->test->close`

## 揭秘setState机制
那么 说了这么多，事务是怎么导致前面所述的setState的各种不同表现呢.

在整个React组件渲染到Dom中的过程就处于一个大的事务中 ，在生命周期和合成事件执行前后都会执行init 和 close，init会调用batchedUpdate方法将isBatchingUpdates变量置为true，开启批量更新，而close会将isBatchingUpdates置为false，setState的更新会被存入队列中，待同步代码执行完后，再执行队列中的state更新。

而在原生事件和异步操作中，不会执行pre钩子，或者生命周期的中的异步操作之前执行了pre钩子，但是pos钩子也在异步操作之前执行完了，isBatchingUpdates必定为false，也就不会进行批量更新

## 获取正确的state值
以下：
1. setState函数式
2. 放到setTimeout，Promise等异步中执行
3. 放到componentDidUpdate中

## 说在最后的话
所以 开篇的结果应该可以理解了吧

我们把didMount中四次调用归类，前两次一类 因为它们在同一个调用栈中执行 setTimeout中的两次属于另一类，我们重点看第一类，早在setState调用之前  ReactDefaultBatchingStrategy.isBatchingUpdates已经被设置为true，所以两次的setSate并没有生效 而是被放进了队列中
再看setTimeout中的两次state 此时的isBatchingUpdates为false，这也就导致了心的state马上生效 没有走到队列的分支（可参考调用栈图）也就是说 第一次执行setState时 值就为1 加1之后变为2 第二次打印同理
 
参考：深入react技术栈一书 希望通过setState 深入源码 知其然也知其所以然







