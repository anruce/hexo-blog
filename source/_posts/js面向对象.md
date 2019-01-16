
---
title: js面向对象
date: 2017/04/21
tags: [js]
toc: false
---

**工厂模式**
解决了重复实例化的问题


```
function createObject(name,age){
var obj = new Object();
obj.name= name;
obj.age = age;
obj.run = function(){
 return this.name + this.age + '运行中'
}
return obj;
}

var box1 = createObject（‘lee’,100）
var box2 = createObject（‘jack’,200）
```

工厂模式的缺点：
无法区分实例是哪个对象的实例


**构造函数模式**


```
function Box(name,age){
 this.name= name;
 this.age = age;
 this.run = function(){
  return this.name + this.age + '运行中'
 }
}
var box3 = new Box（‘lee’,100）
var box4 = new Box（‘jack’,200）
```
如何识别了对象？
构造函数没有new Object，但是它后台回自动var obj = new Object();
this指的就是obj
没有返回值

console.log(box4 instanceof Box)


**对象冒充**
把o冒充成box对象

```
var  o = new Object();
Box.call(o,'Lee',100)
```


**原型**

prototype原型属性是一个对象


```
 function Box(){}
```
 这里如果有属性或者方法 叫做实例属性和实例方法

```
Box.prototypr.name 原型属性
Box.prototypr.run=function(){} 原型方法
```

**_proto_：** 实际上是一个指向原型对象的一个指针，它的作用就是指向构造函数的原型属性constructor


```
var box1 = new Box()
box1.constructor 指向构造函数
box1._proto_指向原型对象
```

判断一个对象实例是不是指向了对象的原型对象，实例化之后是自动指向的


```
Box.prototype.isPrototypeOf(box1)
```

**什么叫闭包？ 有什么用**

闭包是指有权访问另一个作用域中的变量和函数，常见的形式是在<font face="STCAIYUN" color=red size=4>某个作用域中定义的函数</font>

闭包的作用域链包括三部分：

1. 函数本身作用域
2. 闭包定义的作用域
3. 全局作用域

<font face="STCAIYUN" color=green size=4>闭包的常见用途？</font>
1. 读取函数内部的变量
2. 将变量始终保持在内存中
3. 模拟面向对象的代码风格

**匿名执行函数**

不加**var**关键字 默认回呗添加到全局对象的属性中去，类似的临时变量的属性加入全局对象有很多坏处
比如：
别的函数可能误用这些变量 造成全局变量过于庞大，影响访问速度（因为变量的取值是需要从圆形链上遍历的）


```
var datamodel = {    
    table : [],    
    tree : {}    
};    

(function(dm){    
    for(var i = 0; i < dm.table.rows; i++){    
       var row = dm.table.rows[i];    
       for(var j = 0; j < row.cells; i++){    
           drawCell(i, j);    
       }    
    }    

    //build dm.tree      
})(datamodel);  

创建了一个匿名的函数并立即执行它，由于外部无法引用它内部的变量，因此在执行之后很快就会被释放 不会污染全局对象

```


**缓存**

设想我们有一个处理过程很耗时的函数对象，每次调用都会花费很长时间，
那么我们就需要将计算出来的值存储起来，当调用这个函数的时候，首先在缓存中查找，如果找不到，则进行计算，然后更新缓存并返回值，如果找到了，直接返回查找到的值即可。闭包正是可以做到这一点，因为它不会释放外部的引用，从而函数内部的值可以得以保留。


**实现封装**


```
var person = function(){    
    //变量作用域为函数内部，外部无法访问    
    var name = "default";       

    return {    
       getName : function(){    
           return name;    
       },    
       setName : function(newName){    
           name = newName;    
       }    
    }    
}();    

print(person.name);//直接访问，结果为undefined    
print(person.getName());    
person.setName("abruzzi");    
print(person.getName());   
```
