---
title: 你不知道的JS系列-理解JS中 赋值，浅拷贝，深拷贝
date: 2019-03-25 18:28:10
tags:
- clone
categories:
- js
---

## 开篇
又回到了这个老生常谈，新生绝望的问题上，通常遇到这种大家都比较熟悉的问题，反而不知道怎么列大纲，怕不够深入也怕脱离主题～ 
emm..
此文系 不要再问我XX系列之 <font color=red>不要再问我JS Clone的问题了</font>

<!--more-->

## 为什么会存在这三种情况？三者有何差异

`clone`本来很简单，只是因为JS中不同的数据类型存储方式(**堆和栈**)的差异，我们才会觉得它貌似有点‘复杂’

![clone1](/images/clone/clone1.png)

基本类型和引用类型的差异如上图所示了
它们共同的目标就是<font color=red>以一个对象为原型clone出另外一个新对象，因为自身的问题产生一些副作用，三者的差异其实就体现在副作用的差异上</font>

### 差异（堆和栈）

* 栈（stack）为自动分配的内存空间，它由系统自动释放
* 而堆（heap）则是动态分配的内存，大小不定也不会自动释放

-------

<font color=red>基础类型：</font> 值存放在栈中，**比较是值的比较**
<font color=red>引用类型：</font> 值存放在堆中，变量实际上是一个存放在栈内存的指针，这个指针指向堆内存中的地址。每个空间大小不一样，要根据情况开进行特定的分配，**引用类型的比较是引用的比较**

```
var person1 = [1,2,3];
var person2 = [1,2,3];
console.log(a === b); // false
```


### 赋值

赋值的概念 即使刚入行也不陌生，每天都在用的`'='`

#### 原理 

* 基本类型：在内存中新开辟一段栈内存，然后再把再将值赋值到新的栈中,是两个独立相互不影响的变量
* 引用类型：赋值是传址，是对象保存在栈中的地址的赋值，这样的话两个变量就指向堆内存的同一个对象，因此两者之间操作互相有影响

#### Demo

```
var obj1 = {
  name:'maying',
  age:22,
  sex:'女',
  language : [1,[2,3],[4,5],[9,0]]
}
var sringD = 'pre';
var obj3 = sringD;
sringD = 'post';

var obj2 = obj1;
obj1.name = 'gaile',
obj1.language[0] = 'jjj'
console.log('obj1',obj1)
       /*
        {
            age: 22
            language: (4) ["jjj", Array(2), Array(2), Array(2)]
            name: "gaile"
            sex: "女"
        }
       */
console.log('obj2',obj2)
        /*
            age: 22
            language: (4) ["jjj", Array(2), Array(2), Array(2)]
            name: "gaile"
            sex: "女"
        */
console.log('sringD',sringD) //post
console.log('obj3',obj3) //pre
```

### 理解浅拷贝

之前的很多年，我认为**赋值差不多等于浅拷贝**
写个小demo 发现它们之间的差异

```
var obj2 = obj1;
var obj3 = {...obj1};
obj1.name = 'gaile',
obj1.language[0] = 'jjj'
console.log('obj1',obj1)
console.log('obj2',obj2)
console.log('obj3',obj3)
```
![qiankaobei2](/images/clone/qiankaobei2.png)
赋值对象，是将对象指针直接赋值给另一个变量
浅拷贝，是重新创建了新对象，所以你更改`obj1.name`的时候不会影响到它,但是改变引用类型时就不能幸免了

**所谓的浅拷贝就是：**

* 当对简单的数据类型进行赋值的时候，其实就是直接在栈中新开辟一个地方专门来存储一样的值
* 当对引用类型进行浅拷贝，后面的对象和前面的对象在第一层数据结构中指向同一个堆地址，但是如果前面的数据不止有一层（<font color=red>属性值是一个指向对象的引用只拷贝那个引用值</font>），类似

```
 language : [1,[2,3],[4,5],[9,0]]
```
**内部的子对象的指针还是同一个地址**

如果要实现一直往下复制 就引出了接下来要说的<font color=red>深拷贝</font>

**结论：浅复制要比复制来的深刻一点，至少它开辟了一个新对象，一块儿新的堆内存**

#### 目前可行的实现方式

站在巨人的肩膀上，我们可以轻松实现浅拷贝

* 数组的浅拷贝

```
1. b = [...a]
2. b = a.slice(0) / [].slice.call(a,0)
3. b = a.concat() / [].concat.call(a)
```
* 对象的浅拷贝

```
1. b = Object.assign({},a)
2. b = {...a}
```

#### 如果要你自己实现呢

原理：遍历对象的每个属性进行逐个拷贝

```
function copy(obj) {
  if (!obj || typeof obj !== 'object') {
    return
  }

  var newObj = obj.constructor === Array ? [] : {}
  for (var key in obj) {
       if(obj.hasOwnProperty(key)){
          newObj[key] = obj[key]
        }
  }
  return newObj
}
```
### 理解深拷贝

深拷贝的意义，就是完全复制，如果你读了上文，应该就没有什么疑问了

将a对象复制一份给对象b，不管a中的数据结构嵌套有多深，当改变a对象中的任意深度的某个值后，b中的该值不会受任何影响

#### 目前可行的实现方式

* `JSON.stringify()``和JSON.parse()`的混合配对使用 

```
var obj4 = JSON.parse(JSON.stringify(obj1)) 

obj1.name='yishu',

obj1.language[1] = ["二","三"];
obj4.language[2] = ["四","五"];


console.log(obj1);   
console.log(obj4); 
```
![deepclone](/images/clone/deepclone.png)
`obj1`,`obj4` 是两个独立的对象，更改数据互不影响，达到了我们要的目的

**它粗暴，有用，但是也有缺点**

1. `在JSON.stringify()`做序列化时，`undefined`、`function`以及`symbol`值，会被忽略

例如

```
var obj = {
  a: {b: 'old'}, 
  c:undefined, 
  d: function () {},
  e:  Symbol('')
 }
var newObj = JSON.parse(JSON.stringify(obj))
newObj.a.b = 'new'
console.log(obj)
console.log(newObj)
```
结果
![jsonquedian](/images/clone/jsonquedian.png)

#### 如果要你自己实现呢

原理：使用递归，遍历每一个对象属性进行拷贝

```
var obj = {
  a: {b: 'old'}, 
  c:undefined, 
  d: function () {},
  e:  Symbol('')
 }


function copy(obj) {
  if (!obj || typeof obj !== 'object') {
    return
  }
  var newObj = obj.constructor === Array ? [] : {}
  for (var key in obj) {
    if (obj.hasOwnProperty(key)) {
      if (typeof obj[key] === 'object' && obj[key]) {
        newObj[key] = copy(obj[key])
      } else {
        newObj[key] = obj[key]
      }
    }
  }
  return newObj
}

var newObj = copy(obj)
newObj.a.b = 'new'
console.log(obj)
console.log(newObj)
```
![jsonquedian](/images/clone/jsonquedian.png)

## 总结

* 赋值：引用复制 执向同一个对象
* 浅拷贝 ：生成一个新对象，只能拷贝一层，当属性值是一个指向对象的引用只拷贝那个引用值
* 深拷贝：完全拷贝，前后对象没有任何关系

## 参考链接
 
https://www.zhihu.com/question/23031215
https://segmentfault.com/a/1190000018204798
