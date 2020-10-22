
---
title: 深入理解Javascript 系列一：编写高质量代码的基本要点
date: 2018/03/20
tags: [js]
toc: false
comments: true
categories: js
---


[汤姆大叔的原文链接](http://www.cnblogs.com/TomXu/archive/2011/12/28/2286877.html)

<!-- more -->
## 书写可维护的代码

```
目标：可读的 一致的 可预测的 看上去就像是同一个人写的
```
## js的变量问题

```
js通过函数管理作用域
在任何函数外面上声明的变量 或者 不声明就直接使用的（隐含的全局概念） 均为全局变量

var uu; // 显式全局变量
function sum(x, y) {
   // 不推荐写法: 隐式全局变量
   result = x + y;
   return result;
}
// 创建隐式全局变量 反例，勿使用
function foo() {
   var a = b = 0;发生的原因是 这个从右往左赋值 b = 0  此时 b是未声明的
   相当于
   var a = (b = 0);
   你可以这个定义局部变量（正确写法）
   var a, b;
}
```
#### <font color=green>隐式全局变量 与显式全局变量的差异</font>
* 通过var创建的全局变量是不能被删除的
* 无var创建的隐式全局变量 是能被删除的

###### 关于隐式全局变量的解释

```
不管一个变量有没有用过，JavaScript解释器反向遍历作用域链来查找整个变量的var声明，如果没有找到var，解释器则假定该变量是全局变量，如果该变量用于了赋值操作的话，之前如果不存在的话，解释器则会自动创建它，这就是说在匿名闭包里使用或创建全局变量非常容易 避免创建式全局变量 方式

(function ($, YAHOO) {
    // 这里，我们的代码就可以使用全局的jQuery对象了，YAHOO也是一样
} (jQuery, YAHOO))
将全局变量当做一个参数传入匿名函数内使用 jquery源码就用这种方式
```


```
在技术上 隐式全局变量 并不是真正的全局变量 但他们是全局对象的属性
属性是可以通过delete 操作符删除的
```
#### <font color=green>如何访问全局变量</font>

```
一般情况下 全局对象可以通过window属性在任何位置访问
也可以

var gobal =function(){
return this;
}
这种方法可以随时获取全局变量 this总是指向全局对象
```
#### <font color=green>变量与函数的预解析（变量提升）</font>

```
function func() {
   var a = 1,
       b = 2,
       sum = a + b,
       myobject = {},
       i, // undefined
       j; //undefined
   // function body...
}
变量初始化可以防止逻辑错误 也解决了var散布的问题

// 反例
myname = "global"; // 全局变量
function func() {
    alert(myname); // "undefined"
    var myname = "local"; // 声明myname的变量被当做局部变量预解析了
    alert(myname); // "local"
}

相当于
myname = "global"; // 全局变量
function func() {
     var myname； // === var myname == undefined
     alert(myname); // "undefined"
     myname = "local";
     alert(myname); // "local"
}

```

#### <font color=green>【延伸】js 是如何执行的</font>

```
代码处理 分为两个阶段
1.变量 函数声明 以及 正常格式的参数创建 这是一个解析和进入上下文的阶段
2.代码执行 函数表达式和 声明的变量被创建
```
## js的循环问题
#### for循环

```
缓存 数组 以及 HTMLCollection 的长度
当要循环的是一个集合对象时候 如果不缓存长度 你要实时查询Dom 而Dom操作一般来讲比较昂贵

像这样

for (var i = 0, max = myarray.length; i < max; i++) {
   // 使用myarray[i]做点什么
}

尽量不要使用i++
像这样儿

i = i + 1
i += 1

还有两种改进形式（参考）
少了一个变量(无max)
向下数到0，通常更快，因为和0做比较要比和数组长度或是其他不是0的东西作比较更有效率

像这样

//第一种变化的形式：

var i, myarray = [];
for (i = myarray.length; i–-;) {
  //....
}

//第二种使用while循环：

var myarray = [],
    i = myarray.length;
while (i–-) {
   // ....
}


```


#### for-in循环

```
也称 ‘枚举’ 应该用在非数组对象的遍历上 不推荐用来循环数组（数组也是对象 但是 for-in中 属性列表的顺序是不能保证的）

值得说明的是 我们平常老是忽略一个很重要的方法
hasOwnProperty() 遍历对象属性的时候可以过滤掉从原型链上下来的属性


	var man = {
	    hands:2,
		legs:2,
		heads:1
	}

	if (typeof Object.prototype.clone === 'undefined'){
	    Object.prototype.clone = function(){
	        //。。。。
		}
	}

//	为了避免枚举 出现clone方法
	for (var i in man) {
//	    方式一 过滤原型属性
	    if (man.hasOwnProperty(i)) {
//            console.log(i, "1:", man[i]);
		}
//		方式二 取消Object。prototype上的方法
		if (Object.prototype.hasOwnProperty.call(man,i)){
//            console.log(i, "1:", man[i]);
		}
		//或者 避免长属性查找
		var hasOwn = Object.prototype.hasOwnProperty;
        if (hasOwn.call(man,i)){
//            console.log(i, "1:", man[i]);
        }
	}

```
#### 给原型自定义添加方法

```
if (typeof Object.prototype.myMethod ！=="function"){
    Object.prototype.myMethod = function(){
    //....
    }
}
```
## 避免隐式类型转换

```
js变量在比较时会隐式类型转换
为了避免引起混乱的隐式类型转换 比较值和表达式类型的时候始终使用
=== 和 !== 操作符
```
## 避免 eval() 不给 给setInterval(), setTimeout()和Function()构造函数传递字符串

```
// 反面示例
setTimeout("myFunc()", 1000);
setTimeout("myFunc(1, 2, 3)", 1000);

// 更好的
setTimeout(myFunc, 1000);
setTimeout(function () {
   myFunc(1, 2, 3);
}, 1000);

```

## parseInt()数值转换不应该忽略第二个参数

```
在ECMAScript 3中，开头为”0″的字符串被当做8进制处理了，但这已在ECMAScript 5中改变了。为了避免矛盾和意外的结果，总是指定基数参数

var month = "06",
    year = "09";
month = parseInt(month, 10);
year = parseInt(year, 10);

```
## 编码规范
* 编码规范
* 代码缩进
* 技术上可以省略的花括号( 只有一条语句的for循环{}) 也不要省略
* 不要省略分号;
* 左花括号的位置
* 适当的地方使用空格 -- 列表模样表达式（相当于逗号）和结束语句（相对于完成了“想法”）后面添加间隔
* 命名规范
* 注释


```


if (true) {
   alert("It's TRUE!");
}
或者
if (true)
{
   alert("It's TRUE!");
}
但是如果要返回一个对象自变量的话
function func() {
   return  // === return undefined;
  // 下面代码不执行
   {
      name : "Batman"
   }
}

结论
总之，总是使用花括号，并始终把在与之前的语句放在同一行：


使用空格

for循环分号分开后的的部分：如for (var i = 0; i < 10; i += 1) {...}
for循环中初始化的多变量(i和max)：for (var i = 0, max = 10; i < max; i += 1) {...}
分隔数组项的逗号的后面：var a = [1, 2, 3];
对象属性逗号的后面以及分隔属性名和属性值的冒号的后面：var o = {a: 1, b: 2};
限定函数参数：myFunc(a, b, c)
函数声明的花括号的前面：function myFunc() {}
匿名函数表达式function的后面：var myFunc = function () {};
花括号的间距  最好使用空格
有一个经常被忽略的代码可读性方面是垂直空格的使用。你可以使用空行来分隔代码单元，就像是文学作品中使用段落分隔一样。


命名规范

以大写字母写构造函数(Capitalizing Constructors)

getName() 公共方法
_getFirst() 私有方法
```


