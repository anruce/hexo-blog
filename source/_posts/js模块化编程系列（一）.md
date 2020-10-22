
---
title: js模块化编程系列（一）
date: 2017/04/23
tags: [js]
toc: false
---

1. 原始写法

```
 function m1(){}
 function m2(){}
```
 缺点：污染了全局变量，容易与其它模块发生命名冲突，而且模块之间看不出直接关系

2. 对象写法

```
var moudle = new Object({
 _count = 0;
 m1:function(){},
 m2:function(){}
})
```
缺点：会暴露所有模块成员，内部状态可以被外部改写

3. 立即执行函数的写法(达到不暴露私有成员的目的)

```
 var moudle1 = (function(){
 var count = 0;
 var m1 = function(){
 //...
 };
 var m2 = function(){
 //...
 };

 return {
          m1:m1,
          m2:m2
        }
 })();
```

 moudle1就是javascript模块的基本写法

4. 放大模式
  背景：如果一个模块很大必须分为几个部分，或者一个模块需要继承另外一个模块时

```
var moudle1 = (function(mod){
   mod.m3 = function(){
   //...
   }
   return mod;
  })(moudle1);
```

  上面的代码为 moudle1 添加了一个新方法m3，然后返回新的moudle1模块

5. 宽放大模式
  背景：在浏览器环境中，模块的各个部分都是从网上获取的，有时候无法知道哪个部分会先加载，如果单纯采用放大模式，第一个执行的 部分有可能加载一个不存在的空对象

```  
 var moudle1 = (function(mod){
   mod.m3 = function(){
   //...
   }
   return mod;
  })(window.moudle1 || {});
```

6. 输入全局变量
 背景：保持模块独立性，内部最好不要与程序的其他部分直接交互

```  
var moudle1 = (function($,YAHOO){
      //...
  })(jQuery, YAHOO);
```
  保持独立的同时，模块的依赖关系变的更明显

7. 模块的规范
  CommonJS和AMD
  CommonJS：nodejs的模块系统，是参照 CommonJS 规范实现的，在CommonJS中，有一个全局方法 require()，用于加载模块

```  
eg:
var math = require('math');
  调用math模块提供的方法：
  math.add(2,3); // 5
```

8. 浏览器环境
  局限使CommonJS规范不适用于浏览器环境

```
 var math = require('math');
 math.add(2,3); // 5
```
  在浏览器中运行，第二行在第一行之后运行，也就是说必须得等到math模块加载完成，如果加载时间很长，整个应用都会停在那里等，对于服务器端来说，所有模块都放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间，但是对于浏览器，这就是致命的问题，取决于网速，
  所以，浏览器端的模块不能采用同步加载，要采用**异步加载**

9. AMD
 ‘异步模块定义’，采用异步方式加载模块，所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，回调函数才执行
 AMD也采用require()语句加载模块，不同于CommonJS，它要求两个参数
　

```
 require([module], callback)
 require(['math'], function (math) {
　　　　math.add(2, 3);
　　});
```
