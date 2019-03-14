---
title: 模拟实现JS的new操作符
date: 2019-03-13 00:15:39
tags:
categories:
---


`JS`中 `new`关键字用来实例化构造函数，那么它背后到底做了什么,能否被模拟实现
答案是肯定的
<!--more-->

## new关键字做了什么
你一定从别的文章或者在实际开发中感受到`new`的妙处，不错,总结下来它主要支持了四个功能

1. 创建了一个全新的对象
2. 这个对象会被执行[[prototype]]（也就是__proto__）链接
3. 生成的新对象会绑定到函数调用的this
4. 通过`new`创建的每个对象将最终被来接到这个函数的prototype对象上
5. 如果函数没有返回对象类型Object(包含Functoin, Array, Date, RegExg, Error)，那么new表达式中的函数调用会自动返回这个新的对象

## 模拟实现new

```
function newOperator(ctor){
    if(typeof ctor !== 'function'){
      throw 'newOperator function the first param must be a function';
    }
    // ES6 new.target 是指向构造函数
    newOperator.target = ctor;
    // 1.创建一个全新的对象，
    // 2.并且执行[[Prototype]]链接
    // 4.通过`new`创建的每个对象将最终被`[[Prototype]]`链接到这个函数的`prototype`对象上。
    var newObj = Object.create(ctor.prototype);
    // ES5 arguments转成数组 当然也可以用ES6 [...arguments], Aarry.from(arguments);
    // 除去ctor构造函数的其余参数
    var argsArr = [].slice.call(arguments, 1);
    // 3.生成的新对象会绑定到函数调用的`this`。
    // 获取到ctor函数返回结果
    var ctorReturnResult = ctor.apply(newObj, argsArr);
    // 小结4 中这些类型中合并起来只有Object和Function两种类型 typeof null 也是'object'所以要不等于null，排除null
    var isObject = typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
    var isFunction = typeof ctorReturnResult === 'function';
    if(isObject || isFunction){
        return ctorReturnResult;
    }
    // 5.如果函数没有返回对象类型`Object`(包含`Functoin`, `Array`, `Date`, `RegExg`, `Error`)，那么`new`表达式中的函数调用会自动返回这个新的对象。
    return newObj;
}
```
## 实例验证

```
function Student(name, age){
    this.name = name;
    this.age = age;
    // this.doSth();
    // return Error();
}
Student.prototype.doSth = function() {
    console.log(this.name);
};
var student1 = newOperator(Student, '轩辕', 18);
var student2 = newOperator(Student, 'Rowboat', 18);
// var student1 = new Student('轩辕');
// var student2 = new Student('Rowboat');
console.log(student1, student1.doSth()); // {name: '轩辕'} '轩辕'
console.log(student2, student2.doSth()); // {name: 'Rowboat'} 'Rowboat'

student1.__proto__ === Student.prototype; // true
student2.__proto__ === Student.prototype; // true
// __proto__ 是浏览器实现的查看原型方案。
// 用ES5 则是：
Object.getPrototypeOf(student1) === Student.prototype; // true
Object.getPrototypeOf(student2) === Student.prototype; // true

```

模拟`new`最大的功臣当属于`Object.create()`这个ES5提供的API

`Object.create(proto, [propertiesObject]) `方法创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__` 它接收两个参数，不过第二个可选参数是属性描述符（不常用，默认是`undefined`）

对于不支持ES5的浏览器，MDN上提供了ployfill方案。

```
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        if (typeof proto !== 'object' && typeof proto !== 'function') {
            throw new TypeError('Object prototype may only be an Object: ' + proto);
        } else if (proto === null) {
            throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
        }

        if (typeof propertiesObject != 'undefined') throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");

        function F() {}
        F.prototype = proto;
        return new F();
    };
}
```

##总结

模拟new洁净版

```
// 去除了注释
function newOperator(ctor){
    if(typeof ctor !== 'function'){
      throw 'newOperator function the first param must be a function';
    }
    newOperator.target = ctor;
    var newObj = Object.create(ctor.prototype);
    var argsArr = [].slice.call(arguments, 1);
    var ctorReturnResult = ctor.apply(newObj, argsArr);

    var isObject = typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
    var isFunction = typeof ctorReturnResult === 'function';
    if(isObject || isFunction){
        return ctorReturnResult;
    }

    return newObj;
}


```





