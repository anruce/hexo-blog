---
title: 一个问题引出的pushState用法
date: 2018-11-27 18:40:40
categories:
  - react
---

### 引出问题
为什么有这篇文章.
最近的开发中遇到这么一个问题
如下图
![gifqian](/images/pushState/pushState1.gif)
扫码进入网页 点击弹出覆盖整个手机屏幕的层 此时点击浏览器的返回 会直接回退到之前扫码页面
其实 这个逻辑很合理 因为它没有历史记录 没有所谓的上一个页面 程序上是合理的
但用户体验无疑是差到极致
对于用户来讲 可能我只是想把当前的弹层关掉而不是退出网页

那么如何解决呢？
### 解决方案
没有历史记录 那我们就手动造出来一条“历史记录”，让程序的返回时 能够有迹可循
最终效果
![gifqian](/images/pushState/pushState2.gif)

相关代码

```
 componentDidMount(){
        //监听popstate事件
        window.addEventListener('popstate',() => {
            this.navLeftClick();
        })    
    }
   //弹层的返回按钮
   navLeftClick = () => {
       this.setState({
           showBrandContainer: false
       })
}

componentWillUnmount() {
    // 离开页面的时候取消监听popstate
   window.removeEventListener('popstate',(state) => {
       this.back();
   }) 
}
    
    selectBrand = () => {
        window.history.pushState({page: 1}, "title 1", "?page=1"); //向history对象push一条state

          <!-- 实现参数透传----
        let search = window.location.search;
        window.history.pushState({ page: 1 }, "", search);
        实现参数透传---- -->
        
        this.setState({
            showBrandContainer: true //开启弹层
        })
    }

```

### History
DOM中的window对象通过window.history方法提供了对浏览器历史记录的读取，让你可以在用户的访问记录中前进和后退
从HTML5开始，我们可以开始操作这个历史记录堆栈
**前进**  `window.history.forward();`
**后退** `window.history.back();`
**向前移动N页** `window.history.go(-N);`
**向后移动N页** `window.history.go(N);`
你甚至可以通过检查浏览器历史记录的length属性来找到历史记录堆栈中的页面总数
`window.history.length`
####  HTML5 history新特性pushState、replaceState
HTML5引入了histtory.pushState()和history.replaceState()这两个方法，他们允许添加和修改history实体。同时，这些方法会和window.onpostate事件一起工作，关于window.popstate 可参考 [window.popstate](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onpopstate)
pushState：向 history 添加当前页面的记录 使用history.pushState()方法来修改referrer
replaceState：和 pushState 的用法完全一样，区别就是它用于修改当前页面在 history 中的记录 

一个🌰

```
假设http://10.70.134.53:3000/opt/financial 控制台执行了JS
var stateObj = { foo: "test" }; history.pushState(stateObj, "page 2","test.html");
url地址栏变为 http://10.70.134.53:3000/opt/test.html，但浏览器不会加载bar.html页面，即使这个页面存在也不会加载。
此时 如果你点击浏览器的返回 浏览器就貌似有了前一页
如下图：

```
![gifqian](/images/pushState/pushState3.gif)

总结：

关于 popstate 事件 需要注意的几点
* 调用history.pushState()或者history.replaceState()不会触发popstate事件.
* popstate事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮(或者在JavaScript中调用history.back()、history.forward()、history.go()方法).

也就是说 要触发该事件 你需要两步
1. 添加并激活一个历史记录条目(history.pushState)
2. .改变历史记录条目(用户行为,比如后退,前进)

