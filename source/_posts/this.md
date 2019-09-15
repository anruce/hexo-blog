---
title: 你不知道的JS系列-this指向
date: 2019-08-26 12:53:02
tags:
- this
categories:
- js
---


this是js中最复杂的机制之一

> 任何足够先进的技术都和魔法无异  - Arthur C.Clarke

<!-- more -->
但其实this机制并没有那么先进，是我们的臆想把它想复杂了，在缺乏认知的情况下，this对你来说就是魔法
### 为什么要使用this？
它提供了一种更优雅的方式来隐式传递一个对象引用，因此可以将API设计得更加简洁并且易于使用
### 对于this的误解
1. 指向函数自身

```
常见需要指向自身的场景是递归
匿名的函数无法指向自身
传统的arguments.callee已经弃用
```
2. 指向函数的词法作用域
 
```
this在任何情况下都不指向函数的词法作用域，在js内部作用域和对象相似，可见的标识符都是它的属性,但是作用域“对象”无法通过js代码访问，它存在于JS引擎内部
```
### this到底是什么？
this是运行时进行绑定的 并不是在编写时绑定的 它的上下文取决于函数调用时的各种条件 this绑定和函数声明的位置没有任何关系 只取决于函数的调用方式

```
当一个函数调用时 会创建一个活动记录（执行上下文）这个记录会包含函数在哪里被调用（调用栈）函数的调用方法 传入的参数等 this就是记录的其中一个属性会在函数执行的过程中用到

this时函数被调用时发生的绑定 指向什么完全取决于函数在哪里被调用
```
#### 调用位置
寻找"函数被调用的位置"
需要分析调用栈（为了到达当前执行位置所调用的所有函数）

#### 绑定规则
找到调用位置 然后按照结论中的四条规则判断
#### 默认绑定
独立函数调用时，如果在严格模式下，就绑定到undefined 否则绑定到全局对象 
#### 隐式绑定(对象上的函数调用)

当函数引用有上下文对象时 隐式绑定规则会把函数中的this绑定到这个上下文对象

**对象属性引用链中只有最顶层或者说最后一层会影响调用位置**

```
function foo(){
  console.log(this.a)
}

var obj2 = {
  a:42,
  foo:foo
};

var obj1 = {
  a:2,
  obj2:obj2
}

obj1.obj2.foo(); //42
```

```
var a = 20;
var obj = {
    a: 10,
    c: this.a + 20,
    fn: function () {
        return this.a;
    }
}

console.log(obj.c); //40
console.log(obj.fn()); //10
```
单独的{}是不会形成新的作用域的，因此这里的`this.a`，由于并没有作用域的限制，所以它仍然处于全局作用域之中。所以这里的`this`其实是指向的`window`对象。


**被隐式绑定的函数会丢失绑定对象而执行默认绑定规则 下面是两个场景**

1.

```
function foo(){
  console.log(this.a)
}
var obj1 = {
  a:2,
  foo:foo
}

var bar = obj1.foo; //函数别名
var a = "global"; //全局属性a
bar(); //bar引用的是foo本身 因此此时的bar其实是一个不带任何修饰符的函数调用 因此使用了默认绑定
```
**2.当传入回调函数时 使用默认绑定规则**

参数传递就是一种隐式赋值，因此我们传入函数时也会被隐式赋值

如果把函数传入语言内置函数 结果也是一样的

```
function foo(){
  console.log(this.a)
}

function dofoo(fn){
  fn();
}
var obj1 = {
  a:2,
  foo:foo
}

var a = "global mama"; //全局属性a

dofoo(obj1.foo) //global mama

setTimeout(obj1.foo,100) //global mama 将函数传入语言内置的函数
相当于
function setTimeout(fn,delay){
//等待delay之后
fn() 
}
```

#### 显式绑定(使用call apply bind)
通过固定this来修复，可以在某个对象上强制调用函数
`使用 call apply 或者bind`


```
他们的第一个参数式对象，它们会把这个对象绑定到this，接着在调用函数时指定这个this 因为你可以直接指定this的绑定对象 因此我们称之为显式绑定
```

如果 foo.call(1) //一个原始值当作对象 这个原始值会转换成它的对象形式(装箱)

**可惜，显示绑定无法解决丢失绑定的问题**

但是显示绑定的一个变种“硬绑定”可以解决这个问题

**硬绑定**：无论何时调用函数bar它总会手动在obj调用foo 这种绑定是一种显式的强制绑定

```
function foo(){
  console.log(this.a)
}

var obj1 = {
  a:2
}
var obj2 = {
  a:4
}
var bar = function(){
  foo.call(obj1)
}
bar(); //2
setTimeout(bar,100) //2
bar.call(obj2) // 2 硬绑定的bar不可能再修改它的this


```

由于硬绑定是一种常用的模式，所以ES5提供了内置的方法
`Function.prototype.bind`


了解了用法 我们来看一个demo

```
let a = {};
let fn = function(){
  console.log(this)
}
fn.bind().bind(a)() // ?
```
答案应该是`window` 你回答对了吗

事实上 我么可以改写一下

它类似于

```
var a = {};
let fn = function(){console.log(this)}
let fn2 = function fn1(){
  return function(){
    return fn.apply();
  }.apply(a)
}

fn2() //window
```

这样儿结果能猜到了吗

fn中的this永远由`第一个bind`决定 所以 结果永远是window


#### new绑定

```
js中的构造函数只是一些使用new操作符时被调用的函数 他们不并不属于某个类也不会去实例化一个类 他们只是被new操作符调用的普通函数

插播new调用的时候会自动执行下面的操作
1. 创建（构造一个全新的对象）
2. 这个对象会被执行原型链接
3. 这个信贷想回绑定到函数调用的this
4. 如果函数没有返回其他对象那么new表达式中的函数调用会自动返回这个新对象
```
由此可以知道 new操作符调用时，this指向生成的新对象
⚠️new 调用时的返回值，如果没有显式返回对象或者函数，才会返回新对象
关于这一点

[这应该是一篇关于模拟实现JS的new操作符的文章]()

#### 绑定优先级
new调用 >显示绑定(apply,call,bind) > 隐式绑定（对象上的函数调用） > 默认绑定

#### 绑定例外
规则总有例外，这里也一样

当你把`null`或者`undefined`作为`this`的绑定对象传入`call apply`或者`bind` 这些在调用时候会被忽略应用默认规则
应用场景：当你要传入参数时候 如果函数不关心this的话 你可以传入null当作一个占位值 但是这种方式不可取 绑定this可能会引起副作用

```
function foo(){
  console.log(this.a)
}

var obj = {
  a:2
}
var a = 3
foo.call(null)
```

一种更安全的this

`Object.create(null) `创建的空对象不会创建`object.prototype` 比`{}`更空 比`null`的语义更清楚

代码如下

```
function foo(a,b){
  console.log(a,b)
}
var w = Object.create(null);

foo.apply(w,[2,3])

//使用bind()进行柯里化
var bar = foo.bind(w,2)
bar(3)
```
**间接引用**
你会有意无意的创建一个函数的间接引用，在这种情况下，调用这个函数会应用默认规则
**它最容易在赋值时产生**
类似这样儿的

`(p.foo = o.foo)();` 它的返回值是目标函数的引用，因此调用的位置是`foo()`而不是`p.foo()`或者`o.foo()`所以使用默认绑定

#### 箭头函数调用模式
先看箭头函数和普通函数的重要区别：
1. 没有自己的this、super、arguments和new.target绑定。
2. 不能使用new来调用。
3. 没有原型对象。
4. 不可以改变this的绑定。
5. 形参名称不能重复。

-------

箭头函数中没有`this`绑定，必须通过查找作用域链来决定其值。
如果箭头函数被非箭头函数包含，则`this`绑定的是最近一层非箭头函数的`this`，否则`this`的值则被设置为全局对象。

```
var name = 'window';
var student = {
    name: 'ma',
    doSth: function(){
        // var self = this;
        var arrowDoSth = () => {
            // console.log(self.name);
            console.log(this.name);
        }
        arrowDoSth();
    },
    arrowDoSth2: () => {
        console.log(this.name);
    }
}
student.doSth(); // 'ma'
student.arrowDoSth2(); // 'window'
```



### 结论

*  this不指向函数自身
*  this不指向函数的词法作用域，当你想要把this和词法作用域查找混合使用时，一定要提醒自己 这是无法实现的
*  this实际上是在函数被调用时发生绑定的，指向什么完全取决于函数在哪里被调用
*  如果要判断一个运行中函数的this绑定 就需要找到这个函数的直接调用位置 找到之后按照下面这四条规则判断this的绑定对象

```
1.由new调用？绑定到新创建的对象
2.由call或apply或bind调用(显示绑定)？绑定到指定的对象
3.由上下文对象调用（隐式绑定）？绑定到上下文对象
4.如果都不是的话 使用默认绑定，如果在严格模式下，就绑定到undefined 否则绑定到全局对象 
```
* ES6中的箭头函数不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this,它会继承外层函数调用的this绑定(无论this绑定到什么),这跟ES6之前代码中的`self=this`机制一样（实际上箭头函数将程序猿们常犯的一个错误：<font color="red">混淆this绑定规则和词法作用域规则 </font>给标准化了,这点容易造成误解）
* DOM事件函数：一般指向绑定事件的DOM元素，但有些情况绑定到全局对象（比如IE6~IE8的attachEvent）

检验一下学习成果

[小小沧海：一道常被人轻视的前端JS面试题](https://www.cnblogs.com/xxcanghai/p/5189353.html)
[从这两套题，重新认识JS的this、作用域、闭包、对象](https://segmentfault.com/a/1190000010981003)
