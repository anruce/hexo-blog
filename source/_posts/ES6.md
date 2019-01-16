
---
title: ES6系列(一)
date: 2017/04/20
tags: [js,es6]
toc: false
---

**数组的扩展**
Array.form() 将两类对象转换为数组
1.类似数组的对象
2.可遍历的对象


```
let arrayLike = {
'0':'a',
'1':'b',
'2':'c',
'3':'d'
}
```
ES5写法

```
var arr1 = [].slice.call(arrayLike)
```
ES6 写法

```
let arr2 = Array.form(arrayLike)
```

任何有length属性的对象，都可以通过Array.from方法转为数组，而此时扩展运算符就无法转换。

值得提醒的是，扩展运算符（...）也可以将某些数据结构转为数组。


```
// arguments对象
function foo() {
  var args = [...arguments];
}

// NodeList对象
[...document.querySelectorAll('div')]

Array.from还可以接受第二个参数，作用类似于数组的map方法

Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.of()
Array.of方法用于将一组值，转换为数组。

Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1

[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10

```

fill方法使用给定值，填充一个数组。


```
['a', 'b', 'c'].fill(7)
```

ES6提供三个新的方法——**entries()**，**keys()**和**values()**——用于遍历数组


```
[1, 2, 3].includes(2);     // true
```
**for of 循环**

ES6 引入 rest 参数（形式为“...变量名”），用于获取函数的多余参数

扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
console.log(...[1, 2, 3])
// 1 2 3

函数的name属性，返回该函数的函数名。


```
function foo() {}
foo.name // "foo"
```



<font face="STCAIYUN" color=red size=4>（1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。</font>

对象的扩展
var bax= {bac} = var baz = {bac:bac}


```
var o = {
method:function(){}
} ==
var o = {
 method(){}
}
```

let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};

const person = {
  sayName() {
    console.log('hello!');
  },
};

person.sayName.name   // "sayName"

Object.is（）用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。


```
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false

不同之处只有两个：一是+0不等于-0，二是NaN等于自身
```


<font face="STCAIYUN" color=green size=4>Object.is(+0, -0) // false
</font>

<font face="STCAIYUN" color=blue size=4>Object.is(NaN, NaN) // true
</font>

Object.assign() 实行的是<font face="STCAIYUN" color=red size=4>浅拷贝不是深拷贝
</font>
**如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。**

例子

```
var obj1 = {a: {b: 1}};
var obj2 = Object.assign({}, obj1);

obj1.a.b = 2; 如果值更改 则拷贝之后的值也是变化的
obj2.a.b // 2

如果该参数不是对象，则会先转成对象，然后返回。

typeof Object.assign(2) // "object"

Object.assign可以用来处理数组，但是会把数组视为对象。

Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]

Object.assign(someClass.prototype,{
someMethod(){
...
},
antherMethod(){
...
}
})

===
// 等同于下面的写法
SomeClass.prototype.someMethod = function (arg1, arg2) {
  ···
};
SomeClass.prototype.anotherMethod = function () {
  ···
};

```
**克隆对象**
只能可通他自身的值而不能可通它继承的值


```
function clone(origin){
 return Object.assign({},origin)
}
```


```
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```


```
Object.getOwnPropertyDescriptor 获取该属性的描述对象
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```

Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名。

```
var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]
```
![屏幕快照 2017-02-25 下午12.58.09](media/14879652049673/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-25%20%E4%B8%8B%E5%8D%8812.58.09.png)


```
var obj = { foo: 'bar', baz: 42 };
Object.values(obj)
// ["bar", 42]
```


```
var obj = { foo: 'bar', baz: 42 };
Object.entries(obj)
// [ ["foo", "bar"], ["baz", 42] ]

```

**ES6 中遍历对象的属性**

<font face="STCAIYUN" color=red size=4>for...in </font>循环遍历对象自身的和继承的可枚举属性（不含Symbol属性）
 <font face="STCAIYUN" color=red size=4>Object.keys</font>返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含Symbol属性）。
 <font face="STCAIYUN" color=red size=4>Object.getOwnPropertyNames</font>返回一个数组，包含对象自身的所有属性（不含Symbol属性，但是包括不可枚举属性）
 <font face="STCAIYUN" color=red size=4>Object.getOwnPropertySymbols</font>返回一个数组，包含对象自身的所有Symbol属性。

<font face="STCAIYUN" color=red size=4>__proto__属性（前后各两个下划线）</font>，用来读取或设置当前对象的prototype对象。目前，所有浏览器（包括 IE11）都部署了这个属性。



```
// es6的写法
var obj = {
  method: function() { ... }
};
obj.__proto__ = someOtherObj;
```



```
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40
上面代码将proto对象设为obj对象的原型，所以从obj对象可以读取proto对象的属性。
```


**ES6引入的原始类型“**
<font face="STCAIYUN" color=red size=4>Symbol</font>
凡是属性名属于Symbol类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。
Symbol函数前<font face="STCAIYUN" color=red size=4>不能使用new命令</font>，否则会报错。这是因为生成的Symbol是一个原始类型的值，不是对象
由于Symbol值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。
Symbol值不能与其他类型的值进行运算，会报错。
但是，Symbol值可以显式转为字符串。
另外，Symbol值也可以转为布尔值，但是不能转为数值。

let s = Symbol();

typeof s

**数组去重**

新增的数据结构 <font face="STCAIYUN" color=red size=4>Set Map </font>


```
const s = new Set();
[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

```



```
var set = new Set([1, 2, 3, 4, 4]);
[...set]
```

// 去除数组的重复成员
在Set内部，两个NaN是相等。


```
[...new Set(array)]
var set = new Set([1,1,2,2,3,4,5,5,6])
[...set]
```



```
let map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);
```


```
for (let key of map.keys()) {
  console.log(key);
}
```

<font face="STCAIYUN" color=red size=4>Proxy（代理器） 元编程（对编程语言进行编程）</font>


**在目标对象之前设置一层拦截 外界对它的访问必须先通过这一层拦截**


```
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },

  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
```
重写了get和set方法


```
var proxy = new Proxy(target, handler);


var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35

```

<font face="STCAIYUN" color=red size=4>Promise(承诺)</font>

所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。


```
var promise = new Promise(function(resolve,reject){

if('异步操作成功'){
resolve(value)
}else{
reject(error)
}
})


promise.then(function(value){
成功
},function(error){
失败
})
```

遍历器对象本质上，就是一个指针对象。

回调函数
事件监听
发布/订阅
Promise 对象

Generator函数
yield表示执行到此处执行权将交给其它协程也就是说yield 命令部两个阶段的分界线

Generator 函数不同于普通函数的另一个地方，即执行它不会返回结果，返回的是指针对象


```
function* gen(x){
 try{
 var y = yield x +2;
 }catch(e){
 console.log(e)
 }
 return y
 }
```




```
var fetch = require('node-fetch');

 function* gen(){
 var url = 'http://api.github.com/users/github';
 var result = yield fetch (url);
 console.log('ds')
 }
```

 Thunk 函数是自动执行generator函数的一种方法

 传名调用


```
 f(x + 5)
// 传名调用时，等同于
f(x + 5)
```

 传值调用


```
 f(x + 5)
// 传值调用时，等同于
f(6)
```

<font face="STCAIYUN" color=red size=4>Thunk是传名调用的实现</font>，将参数放到一个临时的函数中，再将这个临时函数传入函数体

这个临时函数叫做**Thunk** 函数


```
var x = 1;
function f(m){
 return m*2;
}

f(x+5)

等同于

var thunk = function(){
  return x+5
}

function (thunk){
return thunk()*2
}
```



<font face="STCAIYUN" color=red size=4>async ：Generator 函数的语法糖</font>



```
var asyncReadFile = async function () {
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

**async 🌧️generator比较**

1. async有内置执行器 不需要调用next方法
拥有更好的语义（比起星号和yield 语义更清楚了）
2. async函数返回的是promise对象比 Generator函数返回值是Iterator对象方便多了 可以用then方法指定下一步操作

**async和await**

*   async表示函数中有异步操作
*   await表示紧跟在后面的表达式需要等待结果
*   async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。
*  async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。


错误处理

```
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

async function main() {
  try {
    var val1 = await firstStep();
    var val2 = await secondStep(val1);
    var val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}

async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}


```
### class

**基本形式**

```
class Point{

constructor(x,y){

this.x = x;
this.y = y

}


toString(){
 return  '(' + this.x + ', ' + this.y + ')';
}
}
```

Class不存在变量提升（hoist），这一点与ES5完全不同。



```
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```
类名是MyClass 而不是me Me只在Class的内部代码可用，指代当前类。

const MyClass = class { /* ... */ };


**模块加载方案**

*  CommonJs应用于服务器
*  AMD应用于浏览器

 ES6 提供模块功能 尽量的静态化)

```
import {stat，exut，readFile} from 'fs'
```
 Es6模块是编译时加载
 ES6的模块自动采用严格模式


```
function  f(){}
 export {f}
```


```
 import {lastName as surname} from './profile'
 import命令具有提升效果
```


```
 导出
 export default

 引入
 import * as circle from './circle'; 模块的整体加载

```

使用import命令的时候用户不需要知道所加载的变量名或者函数名 用这个语法可以为模块指定默认输出


```
export default function (){
 console.log('foo')
 }
```
 引入的时候import可以为该匿名函数指定任意的名字，这个时候import 的后面不使用大括号


```
 import customName from  ..

  export default function ee (){
  console.log('foo')
 }
 ==

 function foo (){
 console.log('ee')
 }

 export default foo;
```

 foo的函数名foo 在模块外部时无效的 视同匿名函数加载

 使用export时，对应的import语句需要使用大括号。

 使用 export default  对应的import语句不需要大括号


```
 // 第一组
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

import { default as xxx } from 'modules';

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入

```
本质上，export default就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

import { default as xxx } from 'modules'


```
export {add as default};
```
// 等同于

```
// export default add;
```

ES6模块

```
<script type="module" src="foo.js"></script>

```
都是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了<script>标签的defer属性。

拷贝数组


```
const itemCopy = [];
const item = [1,2,3]
itemCopy = [...item]
console.log(itemCopy)
```
