
---
title: Javascript 设计模式系列
date: 2018/01/16
tags: [js,design_mode]
toc: false
comments: true
categories: web
---


## 面向对象的 Javascript

不同于传统面向对象语言中的类式继承 js通过 **原型委托** 的方式实现对象与对象之间的继承

编程语言分为 ：  
**静态类型语言**：编译时已确定变量的类型
  优点：在编译时就能发现类型不匹配的错误，编译器可以针对不同的数据类型对程序做一些优化工作 提高程序之心速度
  缺点：程序员依照契约来编写程序
**动态类型语言**：要到程序运行的时候 待变量被赋予某个值之后 才会具有某种类型
  优点：代码数量少
  缺点：程序在运行期间有可能发生跟类型相关的错误

  Javascript 是一门典型的动态类型语言

  多态：给不同的对象发送同一条消息的时候 这些对象会根据这个消息分别给出不同的反馈


 多态：
 多种形态 在面向对象语言中，接口的多种不同的实现方式即为多态
 多态指同一个实体同时具有多种形式
 思想：把做什么 和 谁去做 分开
 同一个函数 传入不同的参数 可以实现不同的结果

 js的多态是与生俱来的
 它作为一门动态类型语言 他在编译时没有类型检查的类型


 多态的好处：你不必再向对象询问‘你是什么类型’而后根据得到的答案  调用对象的某个行为 你只管调用该行为就是了

 最根本的作用就是通过把过程话的条件分支语句转化为对象的多态性  从而消除这些条件分支语句

 面向对象编程的优点
 将行为分布在各个对象中，并且让这些对象各自负责自己的行为

 当我们对一些函数发出 调用的指令时 这些函数会返回不同的结果
 这也是多态的一些体现


 封装：封装的目的是将信息隐藏  

 封装数据
 封装实现
 封装类型
 封装变化
         把系统中不变的和变的分离开 只替换变化的 如果变化的也是封装好的 就好替换多了 保证程序的稳定性和可扩展性

 隐藏数据 隐藏实现细节 设计细节以及隐藏对象的类型
 其他对象或者用户不关心他的具体实现 封装使对象之前的耦合变得松散 对象之间只暴露APi接口来通信



##  **原型模式和基于原型继承的Javascript 对象系统**

##### 使用克隆的原型模式



```
	     var Plane = function(){
      	this.blood=100;
      	this.ss = 11;
      	this.tt =2;
      }

      var plane = new Plane();  

      	Object.create = Object.create ||function(value){

             var F = function(){};
      		 F.prototype= value;
      		 return new F();
      	}

      	var clonePlane = Object.create(plane)
      	console.log('clonePlane',clonePlane)
  	      console.log('clonePlane',clonePlane.blood)  100
  	 	   console.log('clonePlane',clonePlane.ss)  11
  	      console.log('clonePlane',clonePlane.tt) 2


```
<!-- more -->

##### javascript的原型继承

所有的js对象都是从某个对象上克隆而来的

 原型编程范程的规则

1.  所有的数据都是对象

    javascript根对象是 `Object.prototype`空对象


    ```
     var obj1 = new Object();
     var obj2 = {};

    ```

2. 要得到一个对象 不是通过实例化类 而是找到一个对象作为一个原型并克隆他

当我们调用 `var obj1 = new Object();` 引擎内部会从`Object.prototype`克隆一个对象出来


```
    function Person(name){
    	this.name =name
    }

Person.prototype.getName = function(){
	return this.name;
};

	var p = new Person('maying');

	console.log(p.name)
	console.log(p.getName())

```
js没有类的概念 但是为什么还调用了 `new Person('maying');`


```
在这里Person 并不是类 而是函数构造器
Javascript的函数既可以作为普通函数被调用 也可以作为函数构造器被调用
当使用 new 运算符来调用函数时 此时的函数就是一个构造器

用new运算符来创建对象的过程 实际上也是先克隆 Object.prototype 对象
```

**抛出问题：js通过 Object.prototype得到一个新的对象 但实际上并不是每次都真正的克隆了一个对象**

3. 对象会记住他的原型


```
就javascript 真正实现来说并不能说对象有原型 而只能说是对象的构造器有原型 对于对象把请求委托给它自己的原型 更好的说法是对象把请求委托给它的构造器的原型。
 那么对象是如何委托给它的构造器的原型呢

 对象通过 _proto_的隐藏属性指向 {Constructor}.prototype

 _proto_就是跟构造器的原型联系起来的纽带

 js的对象最初都是由object.prototype创建的 但是对象构造器的原型并不局限于object.prototype上 二是可以动态指定其他对象

 应用
 当对象a需要借用对象b的能力时 可以有选择性的把对象a的构造器的原型指向对象b 从而达到继承的效果

  var A = function(){
	 	this.name="buyiyang"
	 }

	 A.prototype = {
	 	constructor: A, //手动指定 不指定的话   console.log(A.prototype.constructor)=object()
	 	name:'maying'
	 }

 var B = function(){}
 B.prototype = new A()

  var a = new A();
  console.log('a',a.name)
  console.log(A.prototype.constructor)  

  //  function(){
	 // 	this.name="buyiyang"
	 // }
  console.log(a.__proto__)    // {name: "maying",  constructor: ƒ}
  console.log(A.prototype)  // {name: "maying", constructor: ƒ}

原型链并不是无限长的

当请求达到 A.prototype，并且在 A.prototype 中也没有找到 address 属性的时候， 请求会被传递给 A.prototype 的构造器原型 Object.prototype，显然 Object.prototype 中也没有 address 属性，但 Object.prototype 的原型是 null，说明这时候原型链的后面已经没有别的节点了。 所以该次请求就到此打住，a.address 返回 undefined。

通过 Object.create(null) 可以创建出没有原型的对象
```
```
通过ES6 class 创建对象



	 class Animal {
	 	constructor(name){
	 		this.name = name
	 	}
	 	getName(){
		 		return this.name
		 	}
	 }
	 class Dog extends Animal{
	 	constructor(name){
	 		// console.log('this',this)
	 		console.log('super',super()) //不调用 super() 就没有绑定this super()只能调用一次
	 		// Dog {name: undefined}
	 		console.log('this',this) // Dog {name: undefined}
	 		console.log('name',name)
	 		// super(name)
	 	}
	 	speak(){
	 		return 'woof'
	 	}
	 }
	 var dog = new Dog('scamp')

     console.log(dog.getName()+'----say-----'+dog.speak()) //undefined----say-----woof

```
4. 如果对象无法响应某个请求 它会把这个请求委托给自己的原型



##  **this call apply**

**this**
js的this总是指向一个对象 在运行时基于函数的执行环境动态绑定的 而非函数被声明时的环境

this的指向可以分为下面四种

作为对象的方法调用

```
this指向当前对象
```
作为普通函数调用

```
this指向window
```

```
window.name='maying';

	var getName = function(){
		return  this.name
	}
	console.log(getName()) //maying

	或者

		window.name='maying';

	var myObject = {
		name:'stven',
		getName :function(){
			console.log('this',this)
			return this.name;
		}
	}

    var a = myObject.getName;
	  console.log(a()) //maying 此时用另外一个变量a来引用myObject.getName 并且调用 a() 时 此时是普通函数的调用函数 this是指向全局的

	console.log(myObject.getName()) // stven  getName是作为对象的属性被调用的 此时的this指向 myObject

	在ES5 的 strict模式下 this已经被规定为不会指向全局对象 而是undefined

	function func(){ "use strict"
    alert ( this );
    func();
```
构造器调用
当使用new运算符调用函数时 该函数总会返回一个对象
通常情况下 构造器里的this就指向返回的这个对象
如果 构造器显示返回了一个object类型的对象 那么此次元算结果最终会返回这个对象


```
var Preson = function(){
	 	this.name='maying'
	 	return {
	 		name:'anne'
	 	}

	 }

	   var p =new Preson()
     console.log('p',p.name)  //anne

     如果不显示返回数据会这返回一个非对象的类型 构造器里的this就指向返回的这个对象

     	 var Preson = function(){
	 	this.name='maying'
	 	return 'swe'	 	
	 }
	  var p =new Preson()
     console.log('p',p.name)  //maying
```


`Function.prototype.call` 或者 `Function.prototype.apply`调用

 作用：可以动态地改变传入函数的this
 ES3给Function原型定义的两个方法
 call 和apply区别只是在传递参数的不同


```
var func = function( a, b, c ){
alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};
func.apply( null, [ 1, 2, 3 ] );
第一个参数：函数体内this的指向 传null时 指向 默认的宿主对象 浏览器默认window
第二个参数： 数组或者类数组 apply方法把这个集合中的元素作为参数传递给被调用的函数
```


```
var func = function( a, b, c ){
alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};
func.call( null, 1, 2, 3 );
第一个参数：函数体内this的指向
第二个参数：第二个参数开始往后，每个参数被依次传入函数

```



```
var obj1 = {
		name:'maying',
		getInfo :function(){
			return this.name
		}
	}

	var obj2 = {
		name:'anne'
	}

	  console.log('obj1',obj1.getInfo()) //maying
    console.log('obj2',obj1.getInfo.call(obj2)) //anne 此刻getInfo函数体内的this指向obj2对象
    相当于

    var obj1 = {
		name:'maying',
		getInfo :function(){
			return obj2.name
		}
	}


    Math.max.apply( null, [ 1, 2, 5, 3, 4 ] ) // 输出:5
```

`Function.prototype.bind()`
 用来指定函数内部的this指向


```
       // js原生模拟 Function.prototype.bind 函数实现
		 Function.prototype.bind =  function(context){
		 	  var self = this //this当前指向 fun函数
		 	  console.log('self',self)
		 	  return function(){
		 	 	return self.apply(context,arguments)
		 	 	// 将fun函数内部的this指向obj
		 	 	// 相当于 function(){
						//  console.log('obj1.name',obj1.name)
						// }
		 	  }
		 }


		var obj1 = {
			name:'maxiaoyin'
		}

		var fun = function(){
		 console.log('this.name',this.name)
		}.bind(obj1)

      fun(); //maxiaoyin

```

借用其他对象的方法实现继承



```
  function A (){
       	this.name='maying'
       }
       function B(){
       	A.apply(this,arguments)
       	this.getName = function(){
       		return this.name
       	}
       }

       var b = new B();

       console.log('b',b.getName()) //maying


```

想往 arguments 中添加一个新的元素

```
  (function abc (){
  		// arguments.push('1') 因为arguments不是数组 没办法执行数组有的push方法
  	    // [].push.call(arguments,'1') 或者
  	    Array.prototype.push.call(arguments,'7')
      	console.log('arguments',arguments)
      }


    再操作 arguments 时候 我们通常会借用
 Array.prototype的各种方法


 Array.prototype.push 实际上是属性复制的过程
  可以推断 我们可以把**任意**的对象 传入  Array.prototype.push

  这个任意的限制
  对象本身可以存取书香
  对象的length可读写


  var a= {}
  Array.prototype.push.call(a,'first')
  console.log(a) //{0: "first", length: 1}
  console.log(a instanceof Object) //true
  console.log(a[0]) // 并非数组的用法 first


```

### 闭**包和高阶函数**

js的函数作用域 就像一层半透明的玻璃 从里面可以看到外面 从外面不能看到里面
变量的搜索是从内往外而不是从外往内的

带有var关键字的局部变量 会随着函数调用的结束而销毁


```
一个有趣的问题
	  <div>1</div>
	  <div>2</div>
	  <div>3</div>
	  <div>4</div>
	  <div>5</div>
<script>

	var nodes = document.getElementsByTagName( 'div' );
	for ( var i = 0, len = nodes.length; i < len; i++ ){

			 nodes[ i ].onclick = function(){

				alert ( i );

			}
	};
</script>
结果是每次都是5
原因是： div的click是被异步触发的 当for循环被触发的时候 for循环早已经结束 此时i变量的值已经是5了 这个时候 click事件的函数顺着作用域链向上找变量i的时候 找到的始终是5

解决方式

	for ( var i = 0, len = nodes.length; i < len; i++ ){
            (function(i){
            	       console.log('i',i)
	            	   nodes[ i ].onclick = function(){
					   alert ( i );
			       }
            })(i)
	};
```

**闭包的作用**
1. 封装变量
 闭包可以把一些不需要暴漏在全局的变量封装成私有变量



```

改造之前

var cache = {};

var mult = function(){

         var args = Array.prototype.join.call( arguments, ',' );

		 if ( cache[ args ] ){
		   return cache[ args ];
		  }
       var a = 1;

		for ( var i = 0, l = arguments.length; i < l; i++ ){

		    a = a * arguments[i];
		}

return cache[ args ] = a;

};
alert ( mult( 1,2,3 ) );
alert ( mult( 1,2,3 ) );


改造原理：如果大函数里面又可以复用的小函数 可以提炼出来 但是这些小函数不需要在其他地方调用的话最好用闭包封闭起来
改造之后



 var mult = (function (){
          var cache={}

	 	  var calculate = function(){
			    	var a =1;
			    	for(var i = 0;i<arguments.length; i++){
			    		a = a *arguments[i];
			    	 }   
		            return a;
	 	  }

	 	  return function(){
	 	  	     var args = Array.prototype.join.call(arguments,',')
		         // 如果存在的话
			 	 if(cache[args]){
			 		 	return  cache[args]
			 	  }
			 	  return cache[args] = 	calculate.apply(window,arguments)    
 }})();
 console.log(mult(1,2,3))

```
2.延续局部变量的寿命


**闭包和面向对象的设计**

```
  // 用闭包实现一个面向对象系统
  var extent = function(){
  	  var value = 0;
         return {
             call: function(){
                    value++;
                    console.log( value );
      }

      // 面向对象的写法

      var extend = {
      	this.value = 0;
      	this.call = function(){
      		this.value ++;
      		console.log('this.value',this.value)
      	}
      }


 // 或者

 var Extend = function(){
 	this.value = 0
 };
 Extend.prototype.call=function(){
 	this.value ++;
      		console.log('this.value',this.value)
 }


```

**高阶函数**
满足下列条件之一
函数可以作为参数被传递
函数可以作为返回值输出


判断数据的类型

Object.prototype.toString.call
对象的原生扩展函数 更精确的区分数据类型

15.2.4.2 Object.prototype.toString()
在toString方法被调用时,会执行下面的操作步骤:

1. 获取this对象的[[Class]]属性的值。

2. 计算出三个字符串"[object ", 第一步的操作结果Result(1), 以及 "]"连接后的新字符串。

3. 返回第二步的操作结果Result(2)。
[[Class]]是一个内部属性,所有的对象(原生对象和宿主对象)都拥有该属性.在规范中,[[Class]]是这么定义的:
内部属性 描述
[[Class]] 一个字符串值,表明了该对象的类型。
其过程简单说来就是：1、获取对象的类名（对象类型）。2、然后将[object、获取的类名、]组合并返回。


```
 Object.prototype.toString.call( [1,2,3] )
"[object Array]"
```

**判断数据的类型**


```
var isType = function( type ){
    return function( obj ){
          return Object.prototype.toString.call( obj ) === '[object '+ type +']';
     }
};

var isString = isType( 'String' );
 var isArray = isType( 'Array' );
 var isNumber = isType( 'Number' );
console.log( isArray( [ 1, 2, 3 ] ) );

```

高阶函数实现AOP
（AOP）面向切面编程 主要作用是把一些跟核心业务逻辑的代码抽离出来


函数节流

函数被频繁调用的场景
window.onresize 事件
mousemove 事件
图片或者视频的上传进度

函数节流的原理

 setTimeout 来完成这件事情

 分时函数
 惰性加载函数


## ** 设计模式**


##### 单例模式
定义：保证一个类仅有一个实例，并且提供一个访问他的全局访问点

###### 实现单例模式
 原理： 用一个对象来表示当前是否已经为这个类创建过对象，如果是 则在下一次获取该类的实例的时 直接返回之前创建的对象


```
  var Singleton = function(name){
  	this.name=name,
  	this.create = null;
  }
Singleton.prototype.sayName = function(){
  console.log(this.name)
}

var sayFn = function(name){
	if(!this.create){
		this.create = new Singleton(name)
	}
    return this.create
}

var a = sayFn('maxiaoying').sayName(); //maxiaoying
var b = sayFn('lujing').sayName() //maxiaoying
console.log(a ===b).  //true



另一种闭包的实现方式


var Sngal = function(name){
	this.name=name;
}


Sngal.prototype.sayname =function(){
     consoel.log('this.name',this.name)
}


// var getName= function(){
// 	var instance = null;
// 	return function(name){
// 		if(!instance){
// 			instance = new Sngal(name)
// 		}
// 		return  instance
// 	}
// }(name);

// 或者

var getName= function(){
	var instance = null;
	return function(name){
		if(!instance){
			instance = new Sngal(name)
		}
		return  instance
	}
}();

先让getName 自执行才能得到  然后接收参数 理解闭包
	-------------------------------------
return function(name){
		if(!instance){
			instance = new Sngal(name)
		}
		return  instance
	}
	-------------------------------------
var a = getName('maxiaogiou');
console.log('a',a)
var b = getName('xiaoxioamao');
console.log('b',b)
console.log('a ===b',a===b)

```
