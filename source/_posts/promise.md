---
title: 异步编程方案的演进
date: 2019-03-19 18:28:57
tags:
- promise
categories:
- js
---





程序中**现在运行的部分**和**将来运行的部分**就是**异步编程的核心**

异步编程的演进大致分以下几个时期

* 回调函数时期
* promise时期
* 生成器(ES6) + promise时期
* async/await时期(ES7)

<!-- more -->
# 你必须要知道的基本概念
## 异步、并行，并发的区别

* 异步：是关于现在和将来的事件间隙
* 并行：是关于同时完成多个任务的概念
* 并发：是指分别由任务a和任务b 在一段时间内通过任务间的切换完成了这两个任务，**单线程事件循环是并发的一种形式**

```
并发是指两个或者多个事件随着时间发展交替执行，以至于从更高的层次上看是同时在运行（尽管在任意时刻只处理一个事件）

实际的并发场景
比如社交网站，随着用户向下滚动列表加载更多资源 （一边触发ajax一边响应数据）..
```
## js里的完整运行特性
js从不跨线程共享数据，并且由于js单线程的特性，函数块儿中的代码具有完整运行机制，也就是说，一旦`foo()`开始运行，它的所有代码都会在`bar()`中的任意代码运行之前完成
## 事件循环队列与任务队列的区别
ES6中，有一个任务队列的概念，它是挂载在事件循环队列的每个`tick`之后的一个队列

js引擎运行在宿主环境中（浏览器，node端等），这些环境都有线程的概念，他们都提供了一种机制来处理程序中多个块儿的执行，且执行每块儿时调用js引擎，这种机制被称之为**事件循环**

一旦有事件需要运行，事件循环就会运行，直到队列清空，事件循环的每一轮称为一个tick，用户交互 IO和定时器会向事件队列中加入事件，任意时刻，一次只能从队列中处理一个事件，执行事件的时候，可能直接或者间接地引发一个或者多个后续事件

一个比较形象的比喻

* 事件循环队列：类似于一个游乐场游戏，玩过了一个游戏之后，你需要重新到队尾排队才能再玩一次
* 任务队列：玩过了游戏之后，插队接着玩

# 第一阶段(回调函数时期)

任何时候，只要把一段代码包装成一个函数，并指定它在响应某个事件时执行，你就是在代码中创建了一个将来执行的块儿，也由此在这个程序中引入了异步机制。

## 回调实现异步的特性

* 回调函数是`js`异步的基本单元，但是随着js越来越成熟，对于异步编程的发展，回调已经不够用了以至于产生可怕的**回调地狱**，嵌套函数存在耦合性，一大有所改动就会牵一发而动全身，而且嵌套过多导致错误难以处理
* 回调表达异步流程的方式是非线性的，非顺序的，这使得正确推理这样儿的代码难度很大，难以理解，我们需要一种更同步更顺序更阻塞的方式来表达异步，就像我们的大脑一样。
* 回调会受到**控制反转**的影响，因为回调函数中把控制权交给第三方比如`ajax()`，会造成一系列麻烦的**信任问题** 

<font color=red>可能产生的信任问题</font>

```
1. 调用回调过早
2. 调用回调过晚（或者不被调用）
3. 调用回调次数过多或者过少
4. 未能传递所需的环境和参数
5. 吞掉可能出现的错误和异常
```

## 小结
我们需要一个通用的方案来解决这些信任问题，不管我们创建多少回调，这一方案都可以复用，且没有重复代码的开销 引出了**Promise**
# 第二阶段(Promise时期)
直到`ES6 Promise`的引入JS才真正有内建的异步概念

我们开篇就了解到了 异步编程分现在运行部分 和将来运行部分的概念
* 回调函数的模式是关于**如何处理将来值**
* `Promise` 是把现在和将来归一化了 把他们都变成了将来，也就是说 它把所有的操作都变成了异步的

## Promise的特点
* 我们通过某种方式在函数完成时候得到通知，以便我们可以继续下一步
* 类似于事件订阅 我们不需要关注谁订阅了这些事件，实现了关注点分离
* Promise 封装了依赖时间的状态-它等待底层值的完成或者拒绝 所以promise本身是与时间无关的
* 它可以按照可预测的方式组合 而不用关心底层代码如何结束
* 一旦 `promise`决议，它就永远保持在这个状态
* promise是一种封装和组合未来值的易于复用的机制

## Promise基本用法

```
var p = new Promise(function(resolve,reject){
 //resolve()用于完成
 //reject() 用于拒绝
})

 他有三个状态 
 pending,fulfilled，rejected 
 
 两个过程
 
 pending -> fulfilled
 pending -> rejected
 
resolve：会将传入的真正的promise直接返回，对传入的thenable会展开,如果这个thenable展开是一个拒绝状态，那么从promise.resolve()返回的promise实际上就是这同一个拒绝状态
所以resolve实际上的结果可能是完成或拒绝

rejected ：拒绝状态
```

实例的调用

```

promise.then(resolveFn,rejectFn)
如下：
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

### 如何确定某个值是不是真正的promise（或者说thenable）
利用鸭子模型
如果一个函数或对象具有.then()方法，我们认为这样儿的值就是`Promise`一致的`thenable`

所以不要给函数或者对象添加`.then`
方法，否则这个值就会误认为是一个`thenable` 导致难以追踪的`bug`


既然我们已经知道亟待解决的问题，把回调的缺陷解决了，否则引入`Promise`没有任何意义

## 解决控制反转问题
之前 我们用回调函数封装程序中的代码，然后将其交给第三方等（比如ajax），接着期待其能调用回调实现功能
现在我们要能够把控制反转再反转回来
我们希望第三方**给我们提供其任务何时结束的能力**，然后由我们自己的代码来决定下一步做什么

<font color=red>老实说 绝大多数JS/DOM新增的异步API都是基于Promise构建的</font>
## 解决信任问题
### promise解决调用过早
对一个`promise`调用then的时候，即使这个`promise`已经决议，提供给`then()`的回调总会被异步调用
### promise解决调用过晚
`promise`对象创建`resolve`或者`reject`时，这个promise的then注册的观察回调就会被自动调度，可以确信，这些被调度的回调在下一个异步事件点上依次被立即调用，这些回调中的任意一个都无法影响或延误对其他回调的调用，这是promise的运作方式
### 回调未调用
没有任何东西（甚至是js错误）能阻止promise向你通知他的决议，promise在决议时总是会调用其中一个

如果promise本身永远不被决议。promise也提供了解决方案

```
用于一个超时的promise
function timeoutpromise(){
return new promise(function(){
 setTimeout(functiion(){
  reject('timeout')
 },delay)
})
}
```
### 调用次数过多或过少
promise定义方式使得它只能被决议一次
如果试图调用多次或者resolve和reject都调用，那么这个promise只接受第一个决议，并默默的忽略任何后续调用
当然了，如果你把同一个回调注册了不止一次(p.then(f);p.then(f))那么它被调用的次数就会和注册次数相等

### 未能传递参数/环境值
promise至多只能有一个决议值（完成或拒绝）
如果使用多个参数调用resolve或者reject 第一个参数之后的所有参数都会默默忽略

如果要传递多个值，就必须把他们封装在单个值中传递 比如一个数组或者对象

### 吞掉错误或者异常
如果在`Promise`的创建过程中或者查看决议结果过程中的任何时间点上出现了js的异常错误，那么这个异常就会被捕捉，并且会使这个`Promise`拒绝`reject`

promise甚至把js的异常也变成了异步行为，进而极大降低了静态条件出现的可能

但是如果promise完成后的回调中出现了js异常

因为p.then()本身返回了另外一个promise 正是这个promise(下一个promise)将会因TypeError异常而被拒绝

<font color=red>注意：为什么它不是简单的调用我们的错误处理函数呢？</font>
如果这样儿的话就违背了promise的基本原则：promise一旦决议就不可改变
也会造成有些回调会调用，有些回调不会调用情况会非常不透明

### 是可信任的Promise

**promise并没有完全摆脱回调，他们只是改变了传递回调的位置**

* 如果你向`promise.resolve()`传递一个`非promise` 就会得到用这个值填充的Promise

* 如果你向promise.resolve()传递一个真正的promise 就会返回同一个promise

* `promise.resolve()`可以接受任何`thenable`，得到的是一个真正的Promise 是一个可信任的值，如果你传入的已经是真的Promise 那么就更值得信任了
* 对于用promise.resolve()为所有函数的返回值（不管是不是thenable）都封装一层，这样儿做很容易把函数调用规范为定义良好的异步任务

* `Promise`这种模式通过可信任的语义把回调当参数传递，使得这种行为更加可靠合理，通过把回调的控制反转回来，我们把控制权放在了一个可信任的系统，这种系统的设计目的就是为了使得异步编码更清晰


-------
以上 promise解决了回调函数的致命问题
接下来 我们将展示基于promise的链式流作用


## Promise的链式流
Promise并不是一个单步遵循`this-then-that`操作的机制，我们可以将多个Promise链接在一起表示一系列异步步骤

这种方式实现的有以下特性

* 每次你对promise调用then()它会创建并返回一个新的promise,我们可以将其链接起来
* 不管从then()调用的完成回调（第一个参数）返回的值是什么，他都会被自动设置为被链接promise（第一点中的）的完成
* 调用promise的then()会自动创建一个新的promise从调用返回
* 在完成或拒绝处理函数内部，如果返回一个值或者抛出一个异常。新返回的promise（可链接的）就相应的决议
* 如果返回或拒绝处理函数返回一个promise,它将会被展开，这样儿一来，不管它的决议值是什么，都会成为当前then()返回的链接promise的决议值

如下

```
var p = Promise.resolve(21);
 p.then(function(v){
  console.log(v * 2)  //42
  return v + 2 // ***
})
.then(function(v){
  console.log('ceshi',v) //23
})
```

第一个then就是异步序列的第一步

第二个then是第二步,**只要保证把先前的then(..)连到自动创建的每一个promise即可**

在这个demo 中 我们用了立即返回的return语句

但是我们如果需要步骤二等待步骤一异步来完成一些事情怎么办？ 
也就是说我们想要使promise序列真正能够在每一步有异步能力？


```
我们可以给promise.resolve()传递非（最终值）即 **真正的promise或thenable**，Promise会直接返回真正的promise 或展开接收到的thenable值，并在持续展开`thanable`的同时递归前进

```
如下

```
var p = Promise.resolve(21);
 p.then(function(v){
  return new Promise(function(resolve,reject){
    resolve(v*2) //42
  })
})
.then(function(v){
  console.log(v) //42
})
```
我们把42封装到了返回的promise中，但是它仍然会被展开并最终成为链接的promise的决议，因此第二个.then函数中的到的仍然是42

此时，如果向封装的promise中引入异步，仍然会正常工作

```
var p = Promise.resolve(21);
 p.then(function(v){
  return new Promise(function(resolve,reject){
    setTimeout(function(){
      resolve(v*2)
    },200)
  })
})
.then(function(v){
  console.log(v) //42
})
```
完美 我们可以实现一系列个异步步骤

在这些例子中，一步一步传递的值是可选的，不传的话就是隐式返回`undefined`并且这些`Promise`仍然会以同样的方式链接到一起 每一个`Promise`的决议就成了继续下一个步骤的信号

### 存在默认的resolve和reject回调
默认的reject

如果你调用.then()函数并且只传入一个完成处理函数，一个默认拒绝处理函数就会顶替上来，默认拒绝处理函数只是把错误重新抛出，这使得错误可以继续沿着Promise链传播下去，直到遇到显式定义的拒绝处理函数

```
var p = new Promise(function(resolve,reject){
  reject('ooPs');
})

var p2 = p.then(
  function fulfilled(){
    //不会执行到这里
  },
  //默认的拒绝处理函数 当你没有传时
  function (err){
    throw err;
  }
);
```

默认的reject

```
var p = Promise.resolve(2)

var p2 = p.then(
  function (v){
    //不会执行到这里
    return v
  },
  //默认的拒绝处理函数 当你没有传时
  function reject(err){
    //..
  }
)
.then(function(v){
  console.log('ss',v) //2
})
```
默认的完成处理函数只是把接受到的任何传入值传递给下一个步骤的`promise`而已


## 关于错误处理

### 之前同步的错误处理

-------

我们通常使用try catch处理异常
但是它只能是同步的，**无法用于异步代码模式**
即使你在异步代码 比如setTimeout中谁用try catch 仍然是有问题的，他们采用error-first回调风格，无法很好的组合，多级error-first回调交织，导致了回调地狱的风险

```

function foo(cb){
  setTimeout(function(){
    try {
      var x = baz.bar();
      cb(null,x)
    }
    catch(err){
      cb(err)
    }
  })
}

只有在 baz.bar()调用会同步地立即成功或失败的情况下，这里的try catch才能工作
```

### promise的错误处理 增加 .catch(..)方法
.catch(..)会创建并返回新的promise，这个promise可用于实现promise链式流程控制
它没有采用error-first回调设计风格，而是使用了分离回调风格
`一个回调用于完成情况
一个回调用于拒绝情况`

我们了解到 在完成或拒绝处理函数内部，如果返回一个值或者抛出一个异常。新返回的promise（可链接的）就相应的决议，默认情况下，如果你没有捕捉`.then`的异常，它假定你想要promise状态吞掉所有的错误，如果你忘记查看这个状态，这个错误就会默默地在暗处凋零

所以为了避免被忽略的错误，`promise`链的最佳实践就是最后总以一个`catch`结束

诸如

```
var p = Promise.resolve(42);
p.then(function fulfilled(msg){
  console.log(msg.toLowerCase())
})
.catch((error) => {
  console.log('捕获错误',error)
})
```
这样儿可以成功的捕获错误

但是 `reject()`函数的任何异常都会被作为一个全局未处理的错误抛出
## Promise其他的API

### Promise.done(..)
标示promise链的结束
done()不会创建和返回promise

### Promise.all([])
在异步序列中，任意时刻都只能有一个异步任务正在执行
但是我们如果想要同时执行两个或更多步骤（“并行执行”）的时候，`Promise.all([])`的魅力就体现出来
了

Promise.all([]) 接收一个数组，值可以是（promise，thenable，甚至是立即值），列表里的每个值都要经过Promise.resolve()过滤，以确保要等待的是一个真正的promise，
从返回的promise数据也是一个数组，与传入的顺序一致

如果返回的主promise在且仅在所有成员promise都完成后才会完成 如果有热和一个被拒绝，主promise就会立即被拒绝，并丢弃来自其他所有promise的结果

所以 要为每个promise关联一个错误处理函数

传入空数组时，它会立即完成

### Promise.race([])
与promise.all类似
它一旦有人黑一个promise决议为完成，promise.race就会完成，一旦有任何一个promise决议为拒绝，它就会拒绝

它的完成值是单个消息，并不像promise.all那样儿是一个数组，其他的promise会被丢弃或者忽略

两者都会创建一个promise作为他们的返回值，这个promise的决议完全由传入的promise数组控制

传入空数组时，它会挂住，且永远不会决议

### Promise.finally()

从行为的角度上 有些开发者提出，promise需要一个`finally()`的回调注册，这个回调在`promise`决议后总是会被调用，并且允许你执行任何必要的清理工作

如下

```
var p = Promise.resolve(42);
p.then(()=>{})
.finally(cleanup)
.then(()=>{})
.finally(cleanup)
```
finally会创建并返回一个新的promise以支持链接继续



### new Promise(..)构造器

构造器 Promise必须和new一起使用，并且必须提供一个函数回调，这个回调是同步的，这个函数接收到两个函数回调，用以支持promise的决议

```
var p = new Promise(function(resolve,reject){
})
```
#### 创建两种决议的快捷方式

1. Promise.resolve //用于创建一个已完成的promise
2. Promise.reject //用于拒绝这个promise

以下是等价的

```

var p = new Promise(function(resolve,reject){
reject('oop');
})
var p = Promise.reject('oop')
```


##promise的局限性

* 顺序错误处理

```
他们链接的方式 promise链中的错误容易被无意中忽略掉
```
* 单一值

```
promise只能有一个完成值或一个拒绝理由 对于复杂的场景信息有点局限
```
*  单决议
* 无法取消的promise

```
一旦创建了一个promise并为其注册了完成和拒绝处理函数，如果出现某种情况使得这个任务悬而未决的话 你也没有办法从外部停止它的进程
```
* promise性能

```
更多的工作更多的保护 promise与回调相比 会慢一点
但是作为交换你得到的是大量内建的可信任性
```
## 小结
* promise非常好 他们解决了我们因只用回调的代码而备受困扰的控制反转的问题
* 它并没有摒弃回调，只是把回调的安排转交给了一个位于我么和其他工具之间的可信任的中介机制

* promise也开始提供（尽管不完美）以顺序的方式表达异步流的一个更好的办法，这有助于我们的大脑更好的几乎是和维护异步js代码
 

-------

#第二阶段(生成器 Generator时期)

先来回顾一下 回调表达异步控制流程的两个关键缺陷

1. 基于回调的异步不符合大脑对任务步骤的规划方式
2. 由于控制反转回调并不是可信任的

然后我们用`Promise`解决了如何把回调的控制反转 反转回来,恢复了可信任性
但是它不会暂停
现在我们寻求一种顺序，看似同步的异步流程控制表达这个(.then()钱嵌套多了也受不了)，引出了<font color=red>ES6 生成器</font>的概念

Generator最大的特点就是可以控制函数的执行
它会创建出一个迭代器
Generator 函数是一个状态机，封装了多个内部状态。
调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象（Iterator Object）

它打破了完整执行 我们不再依赖**一个函数一旦开始执行，就会运行到结束，期间不会有其他代码能够打破它并插入其间**的假定

ES6中指定暂停点的语法是`yield` 这样礼貌的表达了一种合作式的控制放弃

如下

```
var x = 1;
function *foo(){
  x++;
  yield;
  console.log('x:',x);
}

function bar(){
  x++
}

var it = foo(); //构建了一个迭代器 并没有执行foo
it.next(); //启动生成器 *foo 并运行了第一行的 x++  *foo在yield处暂停
console.log('sws',x) //2
bar();
console.log('二',x) //3
it.next(); //最后从暂停处恢复了生成器 *foo的执行 运行了  console.log('x:',x); //3
```
生成器是一种特殊的函数，可以一次或者多次的启动和停止 构建生成器是作为异步流程控制的代码模式的基础构件之一

我们可以看到 我们在暂停之后做了我们想做的操作 还执行了我们想要执行的函数`bar`


### 生成器的输入和输出
`var it = foo();` 这行只是创建了一个生成器对象，把它赋给了变量it，用于控制生成器，它是特殊的函数，也具有函数的特质，可以传递参数
`it.next()` 指定生成器从当前位置开始继续运行，停在下一个`yield`处或者直到生成器结束，它调用的结果是一个对象，有一个value属性，持有从`*foo`返回的值（如果有的话）也就是说`yield`会导致生成器在执行过程中发送出一个值(类似`return`)

如下

```
function *foo(x,y){
  return x * y;
}

var it = foo(6,7); 
var obj = it.next(); 
console.log(obj) //{value: 42, done: true}
```

### 迭代消息传递

通过`yield`和`next`实现的内建消息输入输出能力

`next()`调用要比`yield`语句多一个
因为第一个`next()`总是启动一个生成器，并运行到第一个`yield`处，执行第一次`next`时候，传递参数值会被忽略


第一个yield基本上是提出了一个问题：我的值是多少？
谁来回答这一个问题 显然第一个next已经执行，因此由第二个next调用回答第一个yield提出的这个问题

从迭代器的角度看问题

消息是双向传递的 next也可以向暂停的yield表达式发送值 yield作为一个表达式可以发出消息响应next的调用

这里有一个例子能帮助你理解generator的执行

```
//yield基本上就是提出了个问题 我的值等于什么 然后由下一个next()传递的参数回答

function *foo(x){
  let y = 2 *(yield(x+1));
  let z = yield(y/3);
  return x + y + z 
}


let it = foo(5);

console.log(it.next()) //{value: 6, done: false}
console.log(it.next(12)) //{value: 8, done: false}
console.log(it.next(13)) //{value: 42, done: true}


/*
解析
generator函数调用和普通函数不同 它会返回一个迭代器

执行第一次next时候，传递参数值会被忽略，并且函数暂停在yield(x+1)处 所以返回 5+1
执行第二次next时，传入的参数等于上一个yield的返回值 如果你不传参，yield永远返回undefined，此时  let y = 2 * 12 = 24，所以第二个z是24/3 =8
当执行第三次next时 传递的参数会传递给z，所以z=13 x = 5 y = 24 相加等于42
*/

```

**多个迭代器**
每次构建一个迭代器实际上就是隐式构建生成器的一个实例 通过这个迭代器控制的是这个生成器的实例

同一个生成器的多个实例可以同时运行它们甚至可以彼此交互


#### 生产者与迭代器
假设你要产生一系列值，其中每个值都与前面一个有特定的关系  需要一个有状态的生产者能够记住其生成的最后一个值
在此之前 我们可以使用函数闭包来实现

迭代器是一个定义良好的接口，用于从一个生产者一步步得到一系列值，js迭代器的接口与多数语言类似，就是每次想要从生产者得到下一个值的时候调用next()
next调用返回一个对象有两个属性

```
done是一个布尔值 表示迭代器的完成状态
value中放置迭代值
```

#### 同步错误处理
类似

```
try{
    var text = yield foo(11,31);
    console.log(text)
}
catch(err){
    console.log(err)
}
```
生成器`yield`暂停的特性意味着我们不仅能从异步函数调用的到看似同步的返回值，还可以同步捕获来自这些异步函数调用的错误 它使得生成器能够捕获错误是一个很大的进步

# 第四阶段（async/await时期 ES7）
ES6中最完美的时间就是生成器（看似同步的代码）和promise（可信任可组合）的结合

我们不再`yield`出`Promise`而是用`await`等待它决议
它其实就是把前面的经验写进规范


## ES7的esync函数对于ES6的generator函数的改进体现了哪些方面

* 内置执行器

```
generator函数的执行必须依赖于执行器，而async函数自带执行器，也就是说async函数的执行，与普通函数一模一样，只需要一行
asyncReadFile();

像这样 直接调用函数 就可以直接得出结果 不像generator函数，需要调用next方法或者co模块 才能真正执行得到最后的结果

```
* 更好的语义

```
async和await比起 * 和 yield 语义更清楚了async表示函数有异步操作，await表示紧跟在后面的表达式需要等待的结果

```
* 更广的适用性

```
co模块规定 yield命令后面只能是thunk函数或者promise对象，而async函数的await后面 可以是promise对象和原始类型的值（数值，布尔等但这等同于同步操作）
```
* 返回值是promise

```
async函数返回的是promise对象，这比generator函数的返回值是 Iterator对象方便多了，你可以用then方法指定下一步的操作

进一步说 async函数可以看作是多个异步操作包装成的一个promise对象，而await命令就是内部then命令的语法糖
```





## 特点

* 一个函数如果加上async那么它就会返回promise
* async 就是将函数返回值使用 Promise.resolve()包裹了下，和then中处理返回值一样
* await是异步操作，它内部实现了generator,如果后来的表达式不返回promise的话，它就会被包装成Promise.resolve(返回值)，然后去执行函数的同步代码

看下面这个等价的例子

```
async function async1(){
    await async2();
    console.log('async1 end')
}


async function async2(){
  console.log('async2 end')
}
async1();

等价于

new Promise((resolve,reject)=>{
  console.log('async2 end')
  resolve(Promise.resolve()) //Promise.resolve()决议之后将then代码插入到微任务队列的尾部
})
.then(()=>{
  console.log('async1 end')
})
```


*  async及await 配合使用
*  await就是`generator`加上`promise`的语法糖
*  `async/await`是异步的终极解决方案


 
###  优缺点
 <font color=red>优势</font> 
 
*  处理then的调用链，能更清晰的写出来代码，毕竟写一堆then也很
*  能解决回调地狱的问题
 
<font color=red>缺点</font>

*  因为`await`将异步代码改造成了同步代码,如果多个异步代码没有依赖性却使用了`await`导致性能上的降低

如下

```
async function test(){
 await fetch(url)
 await fetch(url2)
 await fetch(url3)
}
这样儿的代码 如果没有依赖最好用promise.all
```
 


#总结
* 生成器是ES6的一个新的函数类型 它并不像普通函数那样总是运行到结束，取而代之的是生成器可以在运行当中（完全保持其状态）暂停 并且将来再从咱题ing的地方恢复运行

* 这种简体的暂停和恢复是合作型的不是抢占型的，这意味着生成器具有独一无二的能力来暂停自身，这是通过关键字`yield`来实现的 不过 只有控制生成器的迭代器具有恢复生成器的能力（next(..)）
* yield/next 不只是控制机制 实际上也是一种双向消息传递机制 yield表达式备注上是暂停下来等待某个值 接下来的next调用则是会向被暂停的yield表达式传回一个值

* 在异步控制流程方便生成器的关键优点是 生成器内部的代码是以自然的同步/顺序方式表达任务的一系列步骤，其技巧在于 我们把可能的异步隐藏在关键字yield的后面 把异步移动到控制生成器的迭代器的代码部分
* 换句话说 生成器为异步代码保持了顺序同步 阻塞的代码模式 这使得大脑可以更自然的追踪代码 解决了基于回调的异步的缺陷

