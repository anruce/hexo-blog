
---
title: js冷知识
date: 2018/01/16
tags: [js,issue]
toc: false
comments: true
categories: js
---

<!-- <blockquote class="blockquote-center">blah blah blah</blockquote> -->
1. **去除input[type='number']时的右侧上下箭头**

      /*在chrome下：*/

        input::-webkit-outer-spin-button,
        input::-webkit-inner-spin-button{
            -webkit-appearance: none !important;
            margin: 0;
            padding-left:5px;
        }

        /*Firefox下：*/
        input[type="number"]{-moz-appearance:textfield;}




2. **判断小数不能大于两位**

		  var hopePriceLength = hopePrice.toString().split(".")[1].length;
			if(hopePriceLength>2){
				notify('请输入正数，最多两位小数','error');
				$(`#${mid}jp-hope-price`).focus();
				return false;
			}

<!-- more -->

3.**去掉ios手机上tap时的黑色背景**


```
a,img,button,input,textarea
{-webkit-tap-highlight-color:rgba(255,255,255,0);}

2.另外，如何去掉textarea,input的默认样式：
input,textarea{-webkit-appearance:none;}
```


4.**判断数据的类型**

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

4.**HTML5去除input [type=search] 的默认边框和删除按钮**


 x-webkit-speech  属性：在GOOGLE浏览器上  还会显示一个小话筒
 autocomplete="off"  属性  关闭浏览器自动记录之前输入的值

webkit内核浏览器里 input 框类型如果是 type="search"
那么将会有边框问题，border:0px 也不能起到作用；

解决方案

```
input[type="search"]{-webkit-appearance:none;}
```
移除 重置默认的Webkit引擎下的Input样式


```
input[type=search] {
-webkit-appearance: textfield;
-webkit-box-sizing: content-box;
font-family: inherit;
font-size: 100%;
}
input::-webkit-search-decoration,
input::-webkit-search-cancel-button {
display: none;
}
```

5.禁止ios和android用户选中文字

```
css{-webkit-touch-callout: none}
```



