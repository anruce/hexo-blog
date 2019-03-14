---
title: 模拟实现JS的bind方法
date: 2019-03-13 13:36:45
tags:
- bind
categories:
- js
---


## 为什么要实现一个bind函数？
`bind()`函数在 `ECMA-262 第五版`才被加入
它可能无法在所有浏览器上运行，为了世界和平,必要的时候我们要手动实现它

<!-- more -->
## 现有bind函数的功能？
改造之前要清楚现有`bind()`函数做了哪些事儿

从[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)上找到一些关于它的定义

bind()方法创建一个新的函数，在调用时设置this关键字为提供的值。并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项

**语法** `function.bind(thisArg[, arg1[, arg2[, ...]]])`

函数会创建一个新绑定函数,它包装了原函数对象
`ceshiFn.bind(myObject)`

绑定函数也可以使用`new`运算符构造，此时提供的`this`值会被忽略，但前置参数（arg1,arg2）仍会提供给模拟函数

```
var Fn = ceshiFn.bind(myObject,1,2)
new Fn() 
```
此时 `myObject` 被忽略 但是 参数依然会传递给`ceshiFn`令其初始化

**参数：**

* <font color=red>thisArg</font>：当被绑定的函数被调用时，将它的`this`关键字设置为`thisArg`
* <font color=red>arg1，arg2</font>:被调用时，这些参数将传递给被绑定的方法

**返回值：**
指定的`this`值和初始化参数改造过原函数拷贝

### 继续探索bind函数的功能

```
var obj = {};
console.log('obj',obj)
console.log(typeof Function.prototype.bind)  // bind
console.log(typeof Function.prototype.bind()) //bind
console.log(Function.prototype.bind.name) //bind
console.log(Function.prototype.bind().name) // bound
```
由此我们可以得到得出以下结论
1. `bind`是 `Function`原型链中 `Function.prototype`的一个属性，每个函数都可以调用它
2. `bind`本身是一个函数名为`bind`的函数，返回值是一个名为`bound` 的函数

下面这个例子

```
var obj = {
    name: 'yishu',
};
function original(a, b){
    console.log(this.name);
    console.log([a, b]);
    return false;
}
var bound = original.bind(obj, 1);
var boundResult = bound(2); // 'yishu', [1, 2]

console.log(boundResult); // false
console.log(original.bind.name); // 'bind'
console.log(original.bind.length); // 1
console.log(original.bind().length); // 2 返回original函数的形参个数
console.log(bound.name); // 'bound original'
console.log((function(){}).bind().name); // 'bound '
console.log((function(){}).bind().length); // 0
```
1. `bind`是函数可被传参数，返回值`bound`也是函数，也可以传参数
2. 被`bind()`绑定的函数的this关键字是`bind()`的第一个参数
3. 传递`bind`的其他参数被接收处理了，`bind()`之后返回的函数`bound`函数的参数也被接收处理了，也就是说被合并处理了
4. 并且`bind()`后的`name`为`bound + 空格 + 调用bind的函数名`。如果是匿名函数则是`bound + 空格`
5. bind后的返回值函数`bound`，执行后返回值是原函数（original）的返回值
6. bind函数形参（即函数的length）是1。bind后返回的bound函数形参根据绑定的函数原函数（original）形参个数确定

到这里 我们根据得出的结论 就可以模拟一个简单版本的bind函数了

### 核心功能的bindFn函数

```
Function.prototype.bindFn = function(thisArg){

  //保证是一个函数调用了bindFn函数
  if (typeof this != 'function'){
    throw new TypeError(this + 'must be a function');
  }

  //保存除了thisArg之外的其他形参 转成数组
  console.log('bindFn',arguments)
    let arg = [].slice.call(arguments ,1);
   var self = this;

   var bound = function(){
        // bind返回的函数 的参数转成数组
        var boundArgs = [].slice.call(arguments);
        // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
        return self.apply(thisArg, arg.concat(boundArgs));
    }
    return bound;
}

// 测试
var obj = {
    name: 'yishu',
    age:18
};
function original(){
  console.log([].slice.call(arguments))
}
var bound = original.bindFn(obj,3,4,5)
bound(7); // [3, 4, 5, 7]

```
到这里基本上把bind的核心功能写完了，也能够适用大部分场景了`bindFn`只能能做到的只是永久地绑定指定的`this` ，但是我们发现`MDN`上关于`bind函数`描述 还有一种情况，那就是<font color=red>当你使用`new操作符`调用绑定函数时</font>
是这么说的

> thisArg：当使用new 操作符调用绑定函数时，该参数无效。
> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的`this`值被忽略，同时调用时的参数被提供给模拟函数


我们可以通过一个实例来看原生的bind对于使用new的情况是怎么样的

```
var obj = {
    name: 'yishu',
};
function original(){
    console.log('this', this.name,[].slice.call(arguments));
}
var bound = original.bind(obj,1);
bound(2,3) //this yishu (3) [1, 2, 3]
new bound(2,3) //this undefined (3) [1, 2, 3]s
```
此时 `this`指向了`new bound()`生成的新对象，所以找不到`name`为`yishu`的值了，但是参数依然传递的

**结论**

* `bind`原先指向`obj`的失效了，其他参数有效。
* `new bound`的返回值是以`original`原函数构造器生成的新对象。`original`原函数的`this`指向的就是这个新对象

我们看到 又涉及到`new`操作了，[写过关于模拟new的文章](http://maying.ink/2019/03/13/new/)
简单摘要 new 做了什么

```
1.创建了一个全新的对象。
2.这个对象会被执行[[Prototype]]（也就是__proto__）链接。
3.生成的新对象会绑定到函数调用的this。
4.通过new创建的每个对象将最终被[[Prototype]]链接到这个函数的prototype对象上。
5.如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象。
```
所以 ，当使用`new`调用的时候，`bind`的返回值函数`bound`内部要模拟实现`new`实现的操作,似曾相识了

###bindFn函数的升级
区分是否是new调用 new调用，需要在bind函数返回值函数里实现模拟`new`的操作

```
Function.prototype.bindFn = function bind(thisArg){
    if(typeof this !== 'function'){
        throw new TypeError(this + ' must be a function');
    }
    // 存储调用bind的函数本身
    var self = this;
    // 去除thisArg的其他参数 转成数组
    var args = [].slice.call(arguments, 1);
    var bound = function(){
        // bind返回的函数 的参数转成数组
        var boundArgs = [].slice.call(arguments);
        var finalArgs = args.concat(boundArgs);
        // new 调用时，其实this instanceof bound判断也不是很准确。es6 new.target就是解决这一问题的。
        //new.target属性允许你检测函数或构造方法是否是通过new运算符被调用的
        if(new.target){ //检测函数或构造方法是否是通过new运算符被调用的
            // 这里是实现上文描述的 new 的第 1, 2, 4 步
            // 1.创建一个全新的对象
            // 2.并且执行[[Prototype]]链接
            // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
            // self可能是ES6的箭头函数，没有prototype，所以就没必要再指向做prototype操作。
            if(self.prototype){
                // ES5 提供的方案 Object.create()
                // bound.prototype = Object.create(self.prototype);
                // 但既然是模拟ES5的bind，那浏览器也基本没有实现Object.create()
                // 所以采用 MDN ployfill方案 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create
                function Empty(){}
                Empty.prototype = self.prototype;
                bound.prototype = new Empty();
            }
            // 这里是实现上文描述的 new 的第 3 步
            // 3.生成的新对象会绑定到函数调用的`this`。
            var result = self.apply(this, finalArgs);
            // 这里是实现上文描述的 new 的第 5 步
            // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，
            // 那么`new`表达式中的函数调用会自动返回这个新的对象。
            var isObject = typeof result === 'object' && result !== null;
            var isFunction = typeof result === 'function';
            if(isObject || isFunction){
                return result;
            }
            return this;
        }
        else{
           //不使用new操作符时
            // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
            return self.apply(thisArg, finalArgs);
        }
    };
    return bound;
}
```

## 总结
* bind是Function原型链中的Function.prototype的一个属性，它是一个函数，修改this指向，合并参数传递给原函数，返回值是一个新的函数。
* bind返回的函数可以通过new调用，这时提供的this的参数被忽略，指向了new生成的全新对象。内部模拟实现了new操作符。
* bindFn

```
// 最终版 删除注释 详细注释版请看上文
Function.prototype.bind = Function.prototype.bind || function bind(thisArg){
    if(typeof this !== 'function'){
        throw new TypeError(this + ' must be a function');
    }
    var self = this;
    var args = [].slice.call(arguments, 1);
    var bound = function(){
        var boundArgs = [].slice.call(arguments);
        var finalArgs = args.concat(boundArgs);
        if(new.target){
            if(self.prototype){
                function Empty(){}
                Empty.prototype = self.prototype;
                bound.prototype = new Empty();
            }
            var result = self.apply(this, finalArgs);
            var isObject = typeof result === 'object' && result !== null;
            var isFunction = typeof result === 'function';
            if(isObject || isFunction){
                return result;
            }
            return this;
        }
        else{
            return self.apply(thisArg, finalArgs);
        }
    };
    return bound;
}

```


