
---
title: 深入理解Javascript 系列二：揭秘命名函数表达式
date: 2018/03/20
tags: [js]
toc: false
comments: true
categories: 前端
---

[汤姆大叔的原文链接](http://www.cnblogs.com/TomXu/archive/2011/12/29/2290308.html)
## 函数表达式 和函数声明

#### 函数表达式

```
function 函数名称（可选）（参数：可选）{函数体}
```
#### 函数声明

```
function 函数名称（参数：可选）{函数体}
```

#### 如何区分两者

```
函数表达式：不声明函数名称和声明了函数名称的其他情况
          如果 function foo(){} 是作为赋值表达式的一部分的话
          像这样：
           1.var bar = function foo(){}; // 表达式，因为它是赋值表达式的一部分
           2.new function bar(){}; // 表达式，因为它是new表达式
           3.被括号括住的(function foo(){})他是表达式的原因是因为括号 ()是一个分组操作符，它的内部只能包含表达式
            (function foo(){}); // 函数表达式：包含在分组操作符内


函数声明：声明了函数名称且function foo(){} 被包含在一个函数体内（规则：只能出现在程序或函数体内）
            像这样
            (function(){
                function bar(){} // 声明，因为它是函数体的一部分
            })();
        或者位于程序最顶部的话
        像这样
        function foo(){} // 声明，因为它是程序的一部分

```
#### 一些特性

```
函数声明:会在任何表达式被解析和求值之前先被解析和求值 即使你的声明在最后一行 它也会在同作用域内的第一个表达式之前被解析/求值
```
#### 函数语句

```
函数语句不是在变量初始化期间声明的，而是在运行时声明的——与函数表达式一样。不过，函数语句的标识符一旦声明能在函数的整个作用域生效了。标识符有效性正是导致函数语句与函数表达式不同的关键所在

// 此刻，foo还没用声明
    console.log('typeof foo1',typeof foo)// "undefined"

    if (true) {
        // 进入这里以后，foo就被声明在整个作用域内了
        function foo(){ console.log('11111') }
    }
    else {
        // 从来不会走到这里，所以这里的foo也不会被声明
        function foo(){ console.log('222') }
    }
    typeof foo; // "function"
    console.log('typeof foo2',typeof foo)

```
#### 命名函数表达式

```
命名函数表达式，理所当然，就是它得有名字，前面的例子var bar = function foo(){};就是一个有效的命名函数表达式，但有一点需要记住：这个名字只在新定义的函数作用域内有效， 这就是 与函数语句的区别

 var f = function foo(){
    return typeof foo; // foo是在内部作用域内有效
  };
  // foo在外部用于是不可见的
  typeof foo; // "undefined"
  f(); // "function"

  命名函数表达式出现的意义：便于调试 调试器在调试的时候会将它的名字显示在调用的栈上
```