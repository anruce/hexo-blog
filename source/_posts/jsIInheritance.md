---
title: 细说JS继承的几种方式
date: 2019-02-27 16:16:28
tags:
categories:
- js
---


面向对象语言支持两种继承方式：接口继承（只继承方法签名）和实现继承（继承实际的方法）由于函数没有签名，ECMAScript只支持实现继承，而实现继承主要是依靠原型链实现的

下面就当下几种继承方式做一个对比参考
<!--more-->

## 原型链继承

<font color="red">核心思想：</font>利用原型让一个引用类型继承另一个引用类型的属性和方法（将父类的实例作为子类的原型）

-------

### 举个🌰

```
function SuperType(){
  this.name = 'yishu';
}
SuperType.prototype.sayName = function(){
  return this.name
}

function SubType (){
  this.age = 25;
} 
 //原型链继承
 SubType.prototype = new SuperType();

 SubType.prototype.sayAge = function(){
  return this.age
 }
 
  var instance = new SubType(); //原型链继承
  console.log('age',instance.age) // 25
  console.log('name',instance.name) //yishu
  console.log('sayName',instance.sayName()) //yishu
  console.log('sayAge',instance.sayAge()) //25
  console.log(' instance.toString()', instance.toString()) //[object Object]
```

-------

#### 优点
* 纯粹的继承关系，实例是子类的实例，也是父类的实例
* 父类新增原型方法/原型属性，子类都能访问到
* 简单 易于实现

#### 缺点
* 无法实现多继承
* 创建子类型的实例时，不能向超类型的构造函数中传递参数
* **包含引用类型值的原型会被所有的实例共享**，通过原型来实现继承的时候，原型实际上会变成另一个类型的实例，于是原来的实例属性也就变成了现在的原型属性了(<font color="red">来自原型对象的所有属性被所有实例共享</font>)
* 想要为子类添加属性或方法 只能在`new SubType()`之后

-------

推荐指数：❤️ （3，4问题比较致命）



## 借用构造函数

<font color="red">核心思想：</font>使用父类的构造函数来增强子类实例，相当于复制父类的实例属性给子类（没用到原型） （**不涉及到原型**）

<font color="red">技术原理：</font>
在子类型构造函数的内部调用超类型构造函数

**插播:**函数只不过是在特定环境中执行代码的对象，因此 你可以通过使用`apply`或者`call`方法也可以在将来新创建的对象上执行构造函数

举个🌰

```
function SuperType(){
this.colors = ['red','blue'];
}

function SubType(){
SuperType.call(this) //继承了SuperType
}

var instance1 = new SubType();
instance1.colors.push('black')
console.log(instance1.colors) //["red", "blue", "black"]

var instance2 = new SubType();
console.log(instance2.colors)  //["red", "blue"]

```
### 优点

* 解决实例共享问题

```
这样儿会在`SubType`对象上执行`SuperType`函数中定义的所有对象初始化代码，`SubType`的每个实例就都会具有自己`colors`属性的副本了
```

* 解决传递参数的问题

```
function SuperType(name){
this.name= name
}

function SubType(name,age){
SuperType.call(this,name) //继承了SuperType 同时传递了参数
this.age = age;
}

var instance = new SubType('yishu',18);
```
* 可以实现多继承（call多个父类对象）

### 缺点

* 只能继承父类的实例属性和方法，不能继承原型属性和方法
* 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

-------

推荐指数：❤️❤️（缺点2比较致命）


## 组合继承（最常用的继承模式）
<font color="red">核心思想：</font>将原型链和构造函数的技术组合到一起 从而发挥二者之长

<font color="red">技术原理：</font>
使用原型链实现对原型属性和方法的继承 
使用构造函数来实现对实例属性的继承
这样能实现**在原型上定义方法实现了函数的服用又能保证每个实例有它自己的属性**

### 举个🌰

```
function SuperType(name){
  this.name = name;
  this.colors = ['red','blue','green']
}

SuperType.prototype.sayName = function(){
  return this.name
}

function SubType (name,age){
  SuperType.call(this,name); //继承实例属性 （第二次调用SuperType()）
  this.age = age;
} 

 SubType.prototype = new SuperType(); //继承原型属性和方法（第一次调用SuperType()）
 SubType.prototype.sayAge = function(){
  return this.age;
 }

var instance1 = new SubType('yishu',25);
instance1.colors.push('yellow');
console.log('instance1.colors',instance1.colors); // ["red", "blue", "green", "yellow"]
console.log('instance1.name',instance1.sayName()); //yishu
console.log('instance1.age',instance1.sayAge());//25

var instance2 = new SubType('Grei',29);
 console.log('instance2.colors',instance2.colors); //["red", "blue", "green"]
console.log('instance2.name',instance2.sayName());//Grei
console.log('instance2.age',instance2.sayAge());//29
```

-------
### 优点

* 可以继承实例属性/方法，也可以继承原型属性/方法
* 不存在引用属性共享问题
* 可传参
* 函数可复用

### 缺点
调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

```
具体的过程

第一次调用的时候 SubType.prototype 会得到两个属性  name和colors  他们都来自于 SuperType 但是现在位于 SubType的原型中 当调用SubType构造函数时 又会调用一次
SuperType的构造函数 这一次又在新对象SubType上创建了实例属性 name和colors  于是这两个属性屏蔽了原型中同名属性
```
-------

推荐指数：❤️❤️❤️❤️（仅仅多消耗了一点内存）

## 原型式继承

<font color="red">核心原理：</font>借助原型可以基于已有的对象创建新的对象 同时还不必因此创建自定义类型


**ES5以前**

```
function ObjectCreate(o){
  function F(){} //创建了一个临时性的构造函数
  F.prototype = o;//将传入的对象当作这个构造函数的原型
  return new F(); //返回这个临时类型的新实例
}

ObjectCreate 方法本质上对传入其中的对象执行了一次浅复制


var person = {
  name:'yishu',
  friends:['xiaohong','xiaoming']
}

var instance1 = ObjectCreate(person);
instance1.age = 45;
instance1.friends.push('xiaolan');
console.log('instance1',instance1.age); //45
console.log('instance1',instance1.friends);// ["xiaohong", "xiaoming", "xiaolan"]
console.log('person.friends',person.friends)//["xiaohong", "xiaoming", "xiaolan"]
console.log('person.age',person.age)//undefined

var instance2 = ObjectCreate(person);
console.log('instance2',instance2.age); //undefined
console.log('instance2',instance2.friends);//["xiaohong", "xiaoming", "xiaolan"]
```



**ES5以后**

通过新增 `Object.create(obj1，obj2)`规范化了原型式继承

* obj1：用做新对象原型的对象
* obj2（可选）为新对象定义额外属性的对象 在传入一个参数的情况下与`ObjectCreate`函数功能相同

```
var person = {
  name:'yishu',
  friends:['xiaohong','xiaoming']
}

var otherPerson = Object.create(person);

otherPerson.name='maying';
otherPerson.friends.push('wqs');
console.log('otherPerson',otherPerson.name) // maying
console.log('otherPerson',otherPerson.friends) // ["xiaohong", "xiaoming", "wqs"]

var otherPerson1 = Object.create(person,{name:{value:'dsdd'}});
console.log('otherPerson1',otherPerson1.name) //dsdd
console.log('otherPerson1',otherPerson1.friends)// ["xiaohong", "xiaoming", "wqs"]

```

-------

### 缺点
* 不是类式继承，而是原型式基础，缺少了类的概念
* 对于引用类型值的属性依然是 共享状态的，这相当于创建了两个person的副本


## 寄生式组合继承（最理想的继承范式）
 
<font color="red">核心原理：</font>
借用构造函数来继承属性
通过原型链的**混成形式**来继承方法

<font color="red">技术原理：</font> 不必为了指定子类型的原型而调用超类型的构造函数 我们所需要的无非就是超类型原型的一个副本而已（使用寄生式来继承超类型的原型 然后再将结果指定给子类型的原型 ）

举个🌰

```
function inhertPrototype(SubType, SuperType){

  var prototype = Object.create(SuperType.prototype); //创建对象
  prototype.constructor = SubType; //如果你创建了一个新对象并替换了函数默认的.prototype对象引用,那么新对象不会自动获得.constructor属性
  SubType.prototype = prototype;//指定对象
}

function SuperType(name){
  this.name = name;
  this.colors = ['red','blue','green'];
}

SuperType.prototype.sayName = function(){
  alert(this.name);
}


function SubType(name,age){
  SuperType.call(this, name);//第二次调用SuperType()
  this.age = age;
}

inhertPrototype(SubType, SuperType);


SubType.prototype.sayAge = function(){
  alert(this.age);         
}

var dd = new SubType('yishu',22);
dd.colors.push('gold');
console.log('dd',dd.colors);  //["red", "blue", "green", "gold"]
dd.sayAge(); //22
dd.sayName(); //yishu

var cc = new SubType('xiaogou',10);

console.log('cc',cc.colors); ["red", "blue", "green"]
cc.sayAge(); //10
cc.sayName();//xiaogou

```

### 优点
完美
* 它只调用了一次构造函数 避免了在SubType.prototype上创建不必要的属性 与此同时 原型链还能保持不变
### 缺点
* 实现不如组合式继承简单

-------

推荐指数：❤️❤️❤️❤️（复杂度扣掉一颗心）




## 总结

ECMAscript支持面向对象编程 但是不使用类或者接口 对象可以在代码执行过程中创建或增强 因此具有动态性而非严格定义的实体 在没有类的情况下 可以采用下列模式创建对象

* 工厂模式

```
使用简单的函数创建对象 为对象天假属性和方法 然后返回对象被构造函数模式取代
```
* 构造函数模式

```
可以创建自定义引用类型 
可以像创建内置对象实例一样使用new

缺点：成员无法复用 包括函数
```
* 原型模式

```
使用构造函数的prototype属性来指定那些应该共享的属性和方法

组合使用 构造函数模式和原型模式 分别定义属性和方法
```

js主要通过原型链实现继承 原型链的构建是通过**将一个类型的实例复制给另一个构造函数的原型**实现的，这样子类型就能访问到超类型所有的属性和方法 这一点与基于类的继承很相似。

原型链的问题是**对象实例共享所有的属性和方法** 因此不适合单独使用
解决这个问题的技术是借助构造函数 （在子类型构造函数中的内部调用超类型的构造函数） 这样就能做到每个实例具有自己的属性 同时还能保证只使用构造函数模式来定义类型

使用最多的继承模式是**组合继承**
通过原型链继承共享的属性和方法
而通过借用构造函数继承实例属性

还有其他继承模式

* 原型式继承

```
可以在不必预先定义构造函数的情况下实现继承 本质是执行对给定对象的浅复制 而复制的副本还可以进行进一步的加强 改造
```
* 寄生式继承

```
与 原型式继承相似
```
* 寄生组合式继承

```
集寄生式继承和组合继承的优点与一身
是实现基于类型继承的最有效的方式
```

## 参考
JavaScript 高级程序设计（第三版）

