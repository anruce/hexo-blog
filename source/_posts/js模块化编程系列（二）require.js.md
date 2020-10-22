
---
title: js模块化编程系列（二）
date: 2017/04/24
tags: [js]
toc: false
---

**require.js**

出现背景：所有的javascript代码都在一个文件中，代码越来越多时必须分成多个文件，依次加载，问题：加载js的时候浏览器停止渲染，加载文件越多，网页的响应时间就越长，由于js之间有依赖关系，因此必须严格保证加载顺序，当依赖关系变的复杂时，代码的编写和维护都会变的异常困难

1.  加载require.js

```
<script src="js/require.js" defer async='true'></script>
async   表明这个文件需要异步加载
在require.js的基础上加载自己的 main.js
<script src="require.js" data-main="js/main"></script>
ata-main:指定程序的主模块，这个人间会第一个被require.js加载，由于require.js默认的文件后缀名是js，所以可以把main.js 简写成main

//main.js

require(['moduleA', 'moduleB', 'moduleC'],   function (moduleA, moduleB, moduleC){
　　　　// some code here
　　})``
```

2.  模块的加载
  当加载不同路径下的模块可以使用 require.config()可以对模块的加载进行自定义， require.config()就写在主模块(main.js)的头部，参数就是一个对象，这个对象的path属性指定各个模块的加载路径

```
　　require.config({
　　　　paths: {
　　　　　　"jquery": "lib/jquery.min",
　　　　　　"underscore": "lib/underscore.min",
　　　　　　"backbone": "lib/backbone.min"
　　　　}
　　});
　　
　　另一种形式
　　
　　　require.config({
　　　　baseUrl: "js/lib",
　　　　paths: {
　　　　　　"jquery": "jquery.min",
　　　　　　"underscore": "underscore.min",
　　　　　　"backbone": "backbone.min"
　　　　}
　　});
　　
　　再或者
　　　require.config({
　　　　paths: {
　　　　　　"jquery": "https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min"
　　　　}
　　});
```

3.  AMD模块的写法
 require.js 加载的模块采用AMD规范
 具体来说，就是模块必须采用特定的define()函数来定义，如果一个模块不依赖其他模块，那么可以直接定义在define函数中

```
//math.js 定义了一个math模块
define(function(){
var add = function(){
   return x+y;
};

return {
 add:add
}
})

加载方法：

// main.js
　　require(['math'], function (math){
　　　　alert(math.add(1,1));
　　});
　　
　　如果这个模块还依赖其他模块，那么那么define()函数的第一个参数，必须是一个数组，指明该模块的依赖性。
　　　define(['myLib'], function(myLib){
　　　　function foo(){
　　　　　　myLib.doSomething();
　　　　}
　　　　return {
　　　　　　foo : foo
　　　　};
　　});
　　当require()函数加载上面这个模块的时候，就会先加载myLib.js文件。

```

3.  AMD模块的写法
 加载非规范的模块
 理论上 require.js加载的模块，必须是按照AMD规范、用define()函数定义的模块
 加载非规范模块，必须先用require.config()方法定义它们的一些特征


```
eg：加载非AMD规范模块 underscore，backbone
　　require.config({
　　　　shim: {

　　　　　　'underscore':{
　　　　　　　　exports: '_'
　　　　　　},
　　　　　　'backbone': {
　　　　　　　　deps: ['underscore', 'jquery'],
　　　　　　　　exports: 'Backbone'
　　　　　　}
　　　　}
　　});
　　
　　shim属性：专门用来配置不兼容的模块。具体来说，每个模块要定义
　　（1）exports值（输出的变量名），表明这个模块外部调用时的名称；
　　（2）deps数组，表明该模块的依赖性。
　　
　　eg：
　　   jQuery的插件可以这样定义：
　　　　shim: {
　　　　'jquery.scroll': {
　　　　　　deps: ['jquery'],
　　　　　　exports: 'jQuery.fn.scroll'
　　　　}
　　}
```

4. require插件
 domready插件，可以让回调函数在页面DOM结构加载完成后再运行。

 　
```
require(['domready!'], function (doc){
　　　　// called once the DOM is ready
　　});
　　
```


text和image插件，则是允许require.js加载文本和图片文件。

```　　
define([
　　　　'text!review.txt',
　　　　'image!cat.jpg'
　　　　],

　　　　function(review,cat){
　　　　　　console.log(review);
　　　　　　document.body.appendChild(cat);
　　　　}
　　);
```


类似的插件还有json和mdown，用于加载json文件和markdown文件
