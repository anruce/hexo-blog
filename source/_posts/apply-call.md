---
title: 模拟实现apply和call方法
date: 2019-03-13 16:12:37
tags:
- call
- apply
categories:
 - js
---


## 通过MDN认识下call和apply
### 语法
`func.apply(thisArg, [argsArray])`
### 参数
**thisArg**：可选的，`func`函数运行的时使用的`this`值

⚠️ 

* 如果这个函数处于<font color=red>非严格模式下</font> 指定其为`null` 或者 `undefined`时 this绑定会应用`默认规则`（<font color=red>这在[分析js指向问题](http://maying.ink/2019/03/11/this/#more)时有提到</font>）
* 如果thisArg是原始值会被包装称对象 `.apply(2)`会被包装成`.apply(Number(2))`


-------
**argsArray**：可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 func 函数。如果该参数的值为 null 或  undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象

-------
**返回值**：
调用有指定this值和参数的函数的结果

-------
几个游泳的例子感受下apply的魔力

**求数组最大最小值**

聪明的apply用法允许你在某些本来需要写成遍历数组变量的任务中使用内建的函数

使用`Math.max/Math.min`来找出一个数组中的最大/最小值

```
var numbers = [5, 6, 2, 3, 7];
var max = Math.max.apply(null, numbers);
console.log(max);

var min = Math.min.apply(null, numbers);
console.log(min);
```

**apply设置的this值**

```
var doSth = function(a, b){
    console.log(this);
    console.log([a, b]);
}
doSth.apply(null, [1, 2]); // this是window  // [1, 2]
doSth.apply(0, [1, 2]); // this 是 Number(0) // [1, 2]
doSth.apply(true); // this 是 Boolean(true) // [undefined, undefined]
doSth.call(undefined, 1, 2); // this 是 window // [1, 2]
```
**用apply将一个数组添加到另一个数组**

如果我们传递一个数组来推送，它实际上会将该数组作为单个元素添加，而不是单独添加元素，因此我们最终得到一个数组内的数组
`concat`确实具有我们想要的行为，但它实际上并不附加到现有数组，而是创建并返回一个新数组
用`apply`就能简单实现

```
var array = ['a', 'b'];
var elements = [0, 1, 2];
array.push.apply(array,elements) // ["a", "b", 0, 1, 2]
//array.push(elements) //) ["a", "b", 0, 1, 2, Array(3)]
console.log(array);
```



call()与apply()非常相似

`fun.call(thisArg, arg1, arg2, ...)`

**call和apply的不同点**

* `apply`只接收两个参数，第二个参数可以是`数组`也可以是`类数组`，其实也可以是对象，后续的参数忽略不计
* `call`接收第二个及以后一系列的参数


### 小结 
重新认识了call和apply会发现
它们作用都是一样的，改变函数里的this指向为第一个参数`thisArg`，如果明确有多少参数，那可以用`call`，不明确则可以使用`apply`。也就是说完全可以不使用`call`，而使用`apply`代替，我们只需要模拟实现`apply`，`call`可以根据参数个数都放在一个数组中，给到`apply`即可

-------
## 模拟实现的准备工作

模拟之前 我们先得看看[ES5规范](http://yanhaijing.com/es5/#book) 关于apply 摘抄以下几条

```
Function.prototype.apply (thisArg, argArray)

当以 thisArg 和 argArray 为参数在一个 func 对象上调用 apply 方法，采用如下步骤：

1.如果 IsCallable(func) 是 false, 则抛出一个 TypeError 异常。

2.如果 argArray 是 null 或 undefined, 则返回提供 thisArg 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。

3.返回提供 thisArg 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。

4.如果 Type(argArray) 不是 Object, 则抛出一个 TypeError 异常。
...
9.提供 thisArg 作为 this 值并以 argList 作为参数列表，调用 func 的 [[Call]] 内部方法，返回结果。

apply 方法的 length 属性是 2。

10.在外面传入的 thisArg 值会修改并成为 this 值。thisArg 是 undefined 或 null 时它会被替换成全局对象，所有其他值会被应用 ToObject 并将结果作为 this 值，这是第三版引入的更改
```
结合上文和规范 ，明确了要解决的问题，**我们如何将函数里的this（一般指向window）指向第一个参数thisArg呢**
不由得想起来了[介绍this指向那一篇文章](http://maying.ink/2019/03/11/this/)
那就采用隐式绑定呀，也就是说 既然他现有的上下文环境是window（全局作用域）,那我们就手动给他创建一个`非全局上下文`

看看这个熟悉的例子

```
var doSth = function(){
    console.log(this);
    console.log(this.name);
    console.log(arguments);
}
var student = {
    name: 'yishu',
    doSth: doSth,
};
student.doSth(1, 2); // this === student // true // 'yishu' // [1, 2]

doSth.apply(student, [1, 2]); // this === student // true // 'yishu' // [1, 2]
```
你能看出来什么？
 在对象`student`上加一个函数doSth，再执行这个函数，这个函数里的`this`就指向了这个对象
 
 那我们就模拟这个对象，给他添加一个函数,使用函数调用之后再删除它
 
 第一版本
 
```
// 浏览器环境 非严格模式
function getGlobalObject(){
    return this;
}

Function.prototype.applyFn = function apply(thisArg,argsArray){
   // 1.如果 `IsCallable(func)` 是 `false`, 则抛出一个 `TypeError` 异常。
  if(typeof this !='function'){
    throw new TypeError(this + 'is not function')
  }
  // 1.如果 `IsCallable(func)` 是 `false`, 则抛出一个 `TypeError` 异常。
  if(typeof argsArray === 'undefined' || argsArray === null){
    argsArray = [];
  }

  //3.如果 Type(argArray) 不是 Object, 则抛出一个 TypeError 异常 .
   if(argsArray !== new Object(argsArray)){
        throw new TypeError('CreateListFromArrayLike called on non-object');
    }
  //4.改变this的指向 在外面传入的 thisArg 值会修改并成为 this 值 如果传入的是 undefined或者null 则this指向应用默认绑定
    if(typeof thisArg === 'undefined' || thisArg === null){
        // ES3: thisArg 是 undefined 或 null 时它会被替换成全局对象 浏览器里是window
        thisArg = getGlobalObject();
    }
    //开始表演
    thisArg = new Object(thisArg);
    thisArg.fn = this;
    //接收返回值
    var fnResult = thisArg.fn(...argsArray);
    delete thisArg.fn;
    return fnResult;
}


var doSth = function(){
    console.log(this);
    console.log(this.name);
    console.log(arguments);
}
var student = {
    name: '马小莹',
    //doSth: doSth, //我们主要模拟了这个函数
};
doSth.applyFn(student, [1, 2]); 

// {name: "马小莹", doSth: ƒ, fn: ƒ}
// 马小莹
// Arguments(2) [1, 2, callee: ƒ, Symbol(Symbol.iterator): ƒ]
```
看起来很完美，那它有没有问题呢？ 其实是有的

**.fn函数同名覆盖问题，`thisArg`对象上有`fn`，那就被覆盖了然后被删除了**

那我们就找一个唯一值的函数名，那映入眼帘的就是`Math.ramdom`函数了

```
    thisArg = new Object(thisArg);
    var _fn = '__fn' + new Date().getTime();
    thisArg[_fn] = this;
    //接收返回值
    var fnResult = thisArg[_fn](...argsArray);
    delete thisArg[_fn];
    return fnResult;
```
到现在 简单版本的`apply`已经实现了，现实业务场景不需要去模拟实现`call和apply`,毕竟是`ES3`就提供的方法

既然实现了`apply`,`call`也就简单了，**原理就是**拿到`call`的参数 转换成数组，然后调用`applyFn`

```

Function.prototype.applyFn = function apply(thisArg){
  var argsArray = [];
  var argumentsLength = arguments.length;
  for(var i = 0; i < argumentsLength - 1; i++){
    argsArray.push(arguments[i + 1]);
    }
    return this.applyFn(thisArg, argsArray);
}
```
## 总结

1. 通过MDN认识call和apply，阅读ES5规范，到模拟实现apply，再实现call




 
 




