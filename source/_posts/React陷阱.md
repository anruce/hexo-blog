---
title: React陷阱
date: 2017/07/2
tags: [react,js]
toc: false
---

* 不要改变props

错误例子：

```
var component = <Component />;
component.props.foo = x; // bad
component.props.bar = y; // also bad
```

这样写是错误的，因为我们手动直接添加的属性React后续没办法检查到属性类型错误，也就是说，当我们手动添加的属性发生类型错误时，在控制台是看不到错误信息的

在React的设定中，初始化完props后，props是不可变的。改变props会引起无法想象的后果

正确写法：

```
var props = {};
props.foo = x;
props.bar = y;
var component = <Component {...props} />;
```

当需要拓展我们的属性的时候，定义个一个属性对象，并通过{…props}的方式引入，React会帮我们拷贝到组件的props属性中。
重要的是—这个过程是由React操控的，不是手动添赋值的属性

需要覆盖的时候可以这么写


```
var props = { foo: 'default' };
var component = <Component {...props} foo={'override'} />;

```


* React默认会进行HTML的转义如下


输入：


```
var content='<strong>content</strong>';

React.render(
    <div>{content}</div>,
    document.body
);
```

输出：


```
<stonrg>content</strong>
```

避免转义：


```
var content='<strong>content</strong>';    

React.render(
    <div dangerouslySetInnerHTML={{__html: content}}></div>,
    document.body
);
```

* 如果在编写时使用了react自定义属性  react是不会渲染的

错误做法：


```
React.render(
    <div dd='xxx'>content</div>,
    document.body
);
```

正确做法：


```
React.render(
    <div data-dd='xxx' aria-dd='xxx'>content</div>,
    document.body
);

```
