---
title: 看完这篇，我奶奶都会设计组件了
date: 2019-07-29 01:40:30
tags: 组件设计
categories: 前端
---


# 前言
组件化思想并不是前端独有的，但却是前端技术的延伸
任何软件开发过程，或多或少都有那么一些组件化的需求
随着三大框架崛起，前端组件化逐渐成为前端开发的迫切需求，一种主流，一种共识，它不仅提高开发效率，同时也降低了维护成本
开发者们不需要再面对一堆晦涩难懂的代码，转而只需要关注以组件⽅式存在的代码⽚段
这是一场新的挑战！

## 文章开始之前，明确本文的边界
- 面试官通常会问：写过前端通用组件吗
- 模块化与组件化的前世今生
- 组件的设计原则
- 组件的职能划分
- 进行组件职能划分的利弊
- 你真的需要组件化吗
- 我对组件化的一些感悟
- 组件化开发方案下，为了提升团队协同效率，需要怎么做
- 总结

<!-- more -->

# 一个面试题引发的思考

```
面试官通常会问 写过前端通用组件吗？
```

你可能会自信的表示： sure！

emm..是的吗？

# 模块化与组件化的前世今生

谈到组件化不得不提模块化，从某种意义上来说模块化和组件化并不在一个维度上，但却又有千丝万缕的联系

## 模块化

 模块一般指能够独立拆分且通用的代码单元

### 几个特点

- 让软件按模块单独开发，各模块通常都用一个标准化的接口来进行通信
- 把系统划分多个模块有助于将耦合减至最低，让代码维护更加简单
- 任何一个类库实际上都是一个模块
- 任何语言都有模块化的思想，如Java 的 import、C++的include、Node的require
- js恰好经历了从无到有的过程

### js的模块化发展历程

#### 1. 没有模块化

最开始前端承载的任务并不多，表单验证基本上是它的全部，js基本上都是写到一个文件或者是直接写到jsp，smarty等模版里

写到js文件

```
<script src="1.js"></script>
```

#### MVC时期

由于早期动态页面时期的业务逻辑都写在页面上，随着逻辑的增多，页面越来越复杂，维护起来也越来越难

整个Web应用的开发轻前端重后端，标签也都是其他语言代码编写的，只能使用，难以维护

目录结构是这个样子的
![jsworke](http://pw1zvypwt.bkt.clouddn.com/11.png)


```
<div class="ency-list">
   <a href=" 'id']}/?channel={$smarty.get.channel}&from={$smarty.get.from}" class="txt"  abevent="article_wiki_list#type={$abevent['first_category'][$category['id']]}/rank={$key+1}/articleid={$article['id']}">{$article['title']}</a >
   {if $article['labels']}
        {foreach from=$article['labels'] item=label }
            <a href="/baike/label_articles/{$label['id']}/?channel={$smarty.get.channel}&from={$smarty.get.from}" class="icon">{$label['name']}</a >
        {/foreach}
   {/if}                
</div>
```

#### 2. 前端AJAX时期  JS大行其道

出现了很多JS的框架（JQuery-UI，easy-UI）
前端能做更多事情了，单文件维护代码已然过于沉重，但是由于JS天生没有模块化的概念又不像后端语言自带模块功能，所以一直在寻求突破
大概经历了这么几个阶段

- 将重复性的代码提取出来，封装函数（<font color=red>全部变量冲突风险，人工维护依赖关系复杂</font>）
- 命名空间
- 闭包封装

举个栗子 找到尘封多年的代码

```
$(function(){
    var iframe_id;
    $(".car-search-fix").popup_frame({
        src:$("#js-hidden-value").data("maintenance_entrance_link"),
        shadowId:".mask",
        beforeShow:function(iframeid){
            iframe_id = iframeid;
            show_loading();
        }
})
```

#### 3. Node来了，它真的来了，它带着模块化方案来了

1. node选择commonJS作为它的模块加载方案，给前端的模块化提供了思路 
2. npm生态，让 node 有了自己的模块仓库

#### 4. 模块化方案

1. 模块化规范出现 Commonjs，AMD，CMD
2. ES6的模块化方案 import/export

自此js正式有了模块化官方规范

### 模块化的不足

![jsworke](http://pw1zvypwt.bkt.clouddn.com/2.png)

- 页面逻辑过于过程

随着页面逻辑越来越复杂，这条“过程线”越来越长，并且越来越绕加之缺少规范约束，其他项目成员根据各自需要，在“过程线”加插各自逻辑，最终这个页面的逻辑变得难以维护

- 创建和挂载对象的方式可不止DOM，API，还有HTML语言，模块化思想如何融入进HTML语言

### 尝试解决

#### 页面结构模块化

为了解决面向过程的问题，出现过很多解决方案，在今天看来无非在向页面结构模块化靠近

抽象一层容器，页面需要不同的零件组装，将事件处理，获取数据，样式处理以页面为单位封装成一个model块，每个 Model 都带有各自的数据，模板，逻辑，已经算是一个完整的功能单元

现在基于Model的页面结构开发，已经带有**组件化**的味道了，此时我们的目录结构变为了

![jsworke](http://pw1zvypwt.bkt.clouddn.com/3.png)

已经有很大的变化了

## 组件化的实践

### N年前微软的组件化的解决方案 HTML Component

历史中总有些遗珠事实上微软提供了组件化的解决方案，名为HTML Component
![jsworke](http://pw1zvypwt.bkt.clouddn.com/4.jpg)

是一个比较完整的组件化方案了但是我们可以看到后来的结果，它没有能够进入标准，默默地消失了，用我们今天的角度来看，它可以说是生不逢时

### W3C制订的 [WebComponents 标准](http://w3c.github.io/webcomponents/explainer/)

四部分功能

- `<template>` 定义组件的 HTML模板能力
- Shadow Dom 封装组件的内部结构，并且保持其独立性
- Custom Element 对外提供组件的标签，实现自定义标签
- import 解决组件结合和依赖加载

我们思考一下，可行的实践化方案需要具备哪些能力

- 资源高内聚（组件资源内部高内聚，组件资源由自身加载控制）
- 作用域独立（内部结构密封，不与全局或其他组件产生影响）
- 自定义标签（定义组件的使用方式）
- 可相互组合（组件正在强大的地方，组件间组装整合）
- 接口规范化（组件接口有统一规范，或者是生命周期的管理）

### 三大框架出现

今天的前端生态里面 React，Angular和Vue三分天下，即使它们定位不同，但核心的共同点就是提供了组件化的能力，算是目前是比较好的组件化实践

#### 1. Vue.js采用了JSON的方法描述一个组件

```
import PageContainer from './layout/PageContainer'
import PageFilter from './layout/PageFilter'

export default {
  install(Vue) {
    Vue.component('PageContainer', PageContainer)
    Vue.component('PageFilter', PageFilter)
  }
}

```

还提供了SFC（Single File Component，单文件组件）‘.vue’文件格式

```
<template>
//...
</template>

<script>
  export default {
    data(){}
  }
</script>

<style lang="scss">
//...
</style>
```

#### 2. React.js发明了JSX，把CSS和HTML都塞进JS文件里

```
class Tabs extends React.Component {
    render() {
        if (!this.props.items) {
            console.error('Tabs中需要传入数据');
            return null;
        }
        const propId = this.props.id;
        return (
            <ul className={this.props.className}>
              <li>测试</li>
            </ul>
        );
    }
}
```

#### Angular.js选择在原本的HTML上扩展

### 小结

我们可以看到现代组件化方案虽然稍显不同 但都尽量保留了原有的标记语言部分与样式表部分

技术一直在变迁【甚至是一个环】，但组件化的核心并没有改变，我们的目标仍然是在API设计尽可能接近原生的情况下完成复用、解耦、封装、抽象的目标，最终服务于开发，提高效率，降低错误率

<font color="red">组件必定是模块化的</font>

组件化就是基于可重用的目的 将一个大的软件系统按照关注点的形式 拆分为多个独立的组件 主要的目的就是减少耦合 
这种化繁为简的思想在后端开发中的体现是微服务，而在前端开发中的体现就是组件化

# 组件的设计原则

我们这里讲的组件专门指前端构建页面的基本组成单位
有了一堆这么好的利器，就能写好组件了吗？
那可不一定，随着业务越来越复杂，还要掌握属于它的一套“设计模式”

## 标准性

```
任何一个组件都应该遵守一套标准，可以使得不同区域的开发人员据此标准开发出一套标准统一的组件
```

## 独立性

```
描述了组件的细粒度，遵循单一职责原则，保持组件的纯粹性
属性配置等API对外开放，组件内部状态对外封闭，尽可能的少与业务耦合
```

## 复用性

```
组件会作为一个复用单元被用在多处，倘若发现它不具有通用性，就不如写在业务里，不需要单独抽离不要为了抽离组件而去抽离

当然，在设计组件时不可能要求使用组件的场景要百分百一致，让组件的适应、拓展能力变强，有助于组件的健壮性
```

## 易用

```
尽量能做到使用者只需要考虑输入以及输出
```

# 组件的职能划分

那有了组件设计的“API”，就一定能开发出高质量的组件吗？
我们还需要了解组件的职能划分，毕竟组件最大的不稳定性来自于展现层

根据以往的开发经验，我认为组件应分为以下几类

- 基础组件（通常在组件库里就解决了）
- 容器型组件（Container）
- 展示型组件（stateless）
- 业务组件
- 通用组件
  - UI组件
  - 逻辑组件
- 高阶组件（HOC）

## 基础组件

为了让开发者更关注业务逻辑，涌现出了很多优秀的UI组件库
比如antd，element-ui，我们只需要调用API便能满足大部分的业务场景，前端角色后置了，开发变得更简单了

## 容器型组件
可以理解为当前功能模块最外层的父组件，集中/统一的状态管理，向其他展示型/容器型组件提供数据（充当数据源）和行为逻辑处理（接收回调），还可以包括处理一些该页面中不属于任何一个展示组件的方法

它有以下这些特点：
- 只负责业务逻辑和数据的处理
- 基本上涵盖了本模块的业务逻辑，主要表现为组件怎么工作的 数据怎么更新的不包含任何`Virtual Dom`的修改或组合，也不会包含组件的样式
- 如果使用了状态管理工具（redux/vuex），那它就是与Store通信的唯一入口
- 充当子级组件通信的状态中转站
- 模版基本都是子级组件的集合，很少包含`DOM`标签
- 辅助代码分离

### 表现形式🌰（vue）

```
<template>
<div class="purchase-box">
  <!-- 面包屑导航 -->
  <bread-crumbs />
  <div class="scroll-content">
    <!-- 搜索区域 -->
    <Search v-show="toggleFilter" :form="form"/>
    <!--展开收起区域-->
    <Toggle :toggleFilter="toggleFilter"/>
    <!-- 列表区域-->
    <List :data="listData"/>
  </div>
</template>
```

## 展示型（stateless）组件
字面意思就是主要做展示的组件，主要表现为组件是怎样渲染的
它有以下这些特点：
- 只通过props接受数据和回调函数，不充当数据源
- 可能包含展示和容器组件 并且一般会有Dom标签和css样式
- 通常用props.children(react) 或者slot(vue)来包含其他组件
- 对第三方没有依赖（其实对于一个应用程序级的组件来说可以有）
- 可以有状态，在其生命周期内可以操纵并改变其内部状态，职责单一，将不属于自己的行为通过回调传递出去，让父级去处理（搜索组件的搜索事件/表单的添加事件）
### 表现形式🌰（vue）

```
 <template>
 <div class="purchase-box">
    <el-table
      :data="data"
      :class="{'is-empty': !data ||  data.length ==0 }"
      >
      <el-table-column
        v-for = "(item, index) in listItemConfig"
        :key="item + index" 
        :prop="item.prop" 
        :label="item.label" 
        :width="item.width ? item.width : ''"
        :min-width="item.minWidth ? item.minWidth : ''"
        :max-width="item.maxWidth ? item.maxWidth : ''">
      </el-table-column>
      <!-- 操作 -->
      <el-table-column label="操作" align="right" width="60">
        <template slot-scope="scope">
          <slot :data="scope.row" name="listOption"></slot>
        </template>
      </el-table-column>
      <!-- 列表为空 -->
      <template slot="empty">
        <common-empty />
      </template>
    </el-table>
    
 </div>
  </template>
<script>
  export default {
    props: {
        listItemConfig:{ //列表项配置
        type:Array,
        default: () => {
            return [{
                prop:'sku_name',
                label:'商品名称',
                minWidth:200
            },{
                prop:'sku_code',
                label:'SKU',
                minWidth:120
            },{
                prop:'product_barcode',
                label:'条形码',
                minWidth:120
            }]
      }
    }}
  }
</script>
```
## 业务组件
话不多说，为你业务服务的组件，先有业务组件，再谈通用组件
## 通用组件
可以在一个或多个APP内通用的组件
### UI组件
- 一般是指通用的纯展示型组件，如弹窗，对话框
- 大概率是由展示型组件抽取出来的

#### 表现形式🌰（vue）

```
<template>
  <div class="empty">
    <img src="../../static/images/empty.png" alt>
    <p>暂无数据</p>
  </div>
</template>
```
### 逻辑组件
- 不包含UI层的某个功能的逻辑集合
#### 表现形式🌰（vue）

```
TODO
逻辑组件的例子
```
## 高阶组件（HOC）
React 和 Vue 都证明了一件事儿：一个函数就是一个组件，用一句话来理解：高阶组件就是包裹组件的组件
用一张图来表示
![jsworke](http://pw1zvypwt.bkt.clouddn.com/hoc.png)
不过vue官方并没有推荐使用HOC :(
但是不妨碍我们理解这种思路，进而可以解决项目中遇到的问题

到这里，介绍完了组件的类别，如果你刚好也有以下的疑问，那这可能会帮助到你
### 表现形式🌰（react）

```
TODO
```

## 为什么要使用容器组件
在此之前，我们一般都是在DidMount里请求数据，如果用vuex/rudux的话会跟store耦合，看起来理所当然，但我们只是寻求更好的代码组织方式

### 引入容器组件的概念只是一种更好的组织方式
- 容器组件专门负责和store通信，把数据通过props传递给展示组件，展示组件如果数据需要更新，需要传递回调给容器组件，在容器组件中执行具体操作（业务逻辑）来获取更新结果
- 展示型组件不再直接和store耦合，而是通过props接口来定义所需的数据和方法，复用性与正确性更能保证

#### tips

```
* 展示型组件直接和store通信的话，那么一个展示型组件就会收到限制，因为你在store里面的字段已经限制他的使用次数和使用的位置
* 各司其职，不易出错，即使出错，也能快速定位问题
```

<font color="red">既然这么好，那我什么时候引入容器组件，什么时候引入展示组件</font>

## 什么时候引入容器组件

我的习惯是功能模块文件夹下的入口文件（index.vue）一定是容器组件
可以尝试这样儿
优先考虑展示组件，当你意识到有一些中间组件不使用它继承的props而是转而传递给他们的子级，每次子级组件需要更多数据时，你都需要重新调整这些中间组件，那么，这时候就要考虑引入容器组件

容器组件和展示组件的区别并没有被严格定义，<font color="red">它们的区别不在技术上而是目的性上</font>

这里有几个供参考的点

- 容器组件倾向于有状态，展示组件倾向于无状态，这不是硬性规定，它们都是可以有状态的
- 不要把分离容器组件和展示组件当做教条，如果你不确定该组件是容器组件还是展示组件，就暂时不要分离，写成展示组件，也许是为时尚早。别着急！
- 这是一个持续的重构过程，不用试图一次就把它做好，习惯这种模式就会培养起一种直觉，知道何时引入容器 就像你知道何时封装一个函数那样儿

# 进行组件职能划分的利弊

## 优点

- 更好的关注分离

```
用这种方式写组件，你可以更好的理解你的app和你的ui，甚至会逐渐形成你自己的开发套路，美其名曰设计模式
```

- 复用性高

```
容器组件/展示组件的划分，采用了单一职责原则的设计模式，容器组件专门负责和 store 通信，展示组件只负责展示，解除了组件的耦合，可以带来更好的可复用性
```

- 它是app的调色版，设计师可以随意调整它的ui而不用改变app的逻辑
- 这会强制您提取“布局组件”，比如ContextMenu，达到更高的易用性
- 提高健壮性

```
由于展示组件和容器组件是通过prop接口来连接，可以利用props的校验机制来增强代码的可靠性，混合的组件就没有这种好处

举个🌰(Vue)
  props: {
    editData: Object,
    statusConfig: {
      type: Object,
      default() {
        return {
          isShowOption: true, //是否有操作栏
          isShowSaveBtn: false
        };
      }
    }
  }
```

- 可测试性

```
组件做的事情更少了，测试也会变得容易
容器组件不用关心UI的展示，只关心数据和更新
展示组件只是呈现传入的props，写单元测试的时候也非常容易mock数据层
```

## 缺点

- 因为容器组件/展示组件的拆分，初期会增加一些学习成本
- 在开发的时候，由于需要封装一个容器，包装一些数据和接口给展示组件，会增加一些工作量
- 在展示组件内对props的声明也会带来少量的工作

# 你真的需要组件化吗？

物极必反
当你决定要组件化的时候，一定要注意以下原则

- 页面层级不宜嵌套超过三层

```
超过三层之后可见组件的数据传递的过程就会变得越复杂
```

- 决定代码是否拆开时，先问自己为什么要这么做！

```
1. 如果它只是几行代码，那么最终可能会创建更多的代码来分离它，有必要吗？我这么做的好处是否超过了成本？
2. 如果你当前的逻辑不太可能出现在其他地方，那么将它嵌入其中更好，如果需要，你可以随时抽离，毕竟组件化没有终点
3. 性能会受到影响吗？
如果状态频繁更改，并且当前在一个较大的，关系比较紧密的组件里，为了避免性能受到影响最好抽离出来
4.考虑你是否打破了一个逻辑上有意义的实体，倘若抽离的话，这个代码被复用的概率有多大

```

文章到这里，不好意思还没有结束～

想结合具体的业务聊下在这么多条条框框下，落实到业务上的所谓的好的组件是什么样儿的
或者说 你心中的组件是什么样子的？

# 我是如何思考的

1. 开始coding之前一定会进行<font color="red">需求分解</font>

## 划分依据

明确你的组件划分依据，目前是两种

- 根据业务划分
- 根据技术划分

1. 我更多的是根据业务去设计我应用中的组件树，可能会画个草图或xmind，它可以帮我统观全局
2. 明确各个组件的边界，内部state的设计，props的设计以及与其他组件的关系（需要回调出去的事件）
3. 明确各个组件的定位与职能划分，设计好父子组件、兄弟组件的通信机制
4. 搭架子
5. 架子有了，开始填空


下面从0设计一个组件，从错误中寻找正确方向的蛛丝马迹

## 切割模版（render()）

这是最容易想到的方法，当一个组件渲染了很多元素，就需要尝试分离这些组件的渲染逻辑
我们以掘金页面为例

![jsworke](http://pw1zvypwt.bkt.clouddn.com/5.png)

大体上看，可以分为Part1，Part2，Part3

### 初步开发

```
<template>
  <div id="app">
    <div class="panel">
      <div class="part1 left">
        <!--内容-->
      </div>
      <div class="part1 right">
        <!--内容-->
      </div>
      <div class="part1 right">
        <!--内容-->
      </div>
  </div>
</template>

```

问题：

- 代码量大，难以维护，难以测试
- 有些许重复量

### 化繁为简

```
<template>
  <div id="app">
      <part1 />
      <part2 />
      <part3 /> 
  </div>
</template>

```

同之前的方式相比，这个微妙的改进是革命性的
解决了测试困难，维护困难的问题，但是依然没有解决代码重复的问题
这些类似将一个大函数逐步拆解成几部分的过程，适用性必然很差
但我看过很多项目的代码，就是这么干的，认为自己做了组件化，抽象的还不错(@_@)

### 组件抽象

经过分析，发现几个部分是有共同点的，具有相似的外层，part2和part3更有相似的titlebar，除了业务内容，完全就是一模一样

🌰（vue）

```
<template>
  <div class="part">
    <header>
      <span>{{ title }}</span>
    </header>
    <slot name="content" />
  </div>
</template>


```

我们将part内可以抽象的数据都做成了props，利用slot去做模版
那么我们在开发相应Part1，Part2时

🌰（vue）
```
<template>
  <div id="app">
      <part title="亦舒">
        <div slot="content">----</div>
      </part>
      <part title="兴隆臻园户型">
        <div slot="content">-----</div>
      </part>
  </div>
</template>
```

更具代表性的示例图

![jsworke](http://pw1zvypwt.bkt.clouddn.com/6.png)

- Part中的UI差异在哪里定义？

```
首先要明确一点，这些差异并不是组件本身造成的，是你自己的业务逻辑造成的，所以容器组件（父组件）应该为此买单
```

- 数据差异在哪里定义？

```
比如Part3中，其他的part只有一个类似更多>>的link，但是它却有多个(一居，二居...)。
这里我推荐将这种差异体现在组件内部，设计方法也很多：
比如可以将link数组化为links；
比如可以将更多>>看作是一个default的link，而多余的部分则是用户自定义的特殊link，这两者合并组成了links。用户自定义的默认是没有的，需要引用组件时进行传入。
总之，只要有数据差异化，就应该结合组件本身和业务上下文将差异合理的消除在内部。

```

- 组件命名规则？

```
一个通用的或者说未来可能通用的，要有相对合理的命名，比如 Search，List,尽量不要出现与业务耦合过深的业务名词，通用组件与业务无关，只与自身抽象的组件有关
我们在设计组件初期，就应该有这种思想，等到真正可以抽出公用组件了，再去苦逼的该名字？
库通常都想让广大开发者用，我们在设计组件时，可以降低标准到先做到你的整个APP中通用
```

## 组件划分细粒度的边界

组件设计规则明明白白写着我们要遵循单一职责原则，这也带来了上文聊过的<font color="red">过度抽象（组件化）</font>的问题，我们结合具体的业务聊一下
![jsworke](http://pw1zvypwt.bkt.clouddn.com/7.png)
如果我们要实现徽章组件，它有两部分组成 按钮与右上角的小红点提示
提示可能是数字，icon等，也是符合单一职责的，可以将其抽离成一个独立组件，但是通常不要这么做
因为同一个app的风格必将是统一的，除此之外没别的应用场景了，就像上文所说的，抽离组件之前，多问自己为什么以及投入/产出比，没有绝对的规则

### tips

<font color="red">单一职责组件要建立在可复用的基础上，对于不可复用的单⼀职责组件我们仅仅作为独立组件的内部组件即可</font>

还有一个典型例子，某二手车网站

![jsworke](http://pw1zvypwt.bkt.clouddn.com/1.gif)
思考，如果让你实现你会如何设计...
我当初是这么设计的
![jsworke](http://pw1zvypwt.bkt.clouddn.com/8.png)

index.js(react)

```
<div className="select-brand-box" onTouchStart={touchStartHandler} onTouchMove={touchMoveHandler} onTouchEnd={touchEndHandler.bind(this, touchEndCallback)}>
     <NavBar></NavBar>
     <Brand key="brands-list" {...brandsProps} />
     <Series key="series-list" {...seriesProps} >
 </div>
 
 export default BrandHoc(index);

```

Brand.js(react)

```
<div className="brand-box">
    <div className="brand-wrap" ref="brandWrap">
        <p className="brands-title hot-brands-title">热门品牌</p>
        <FlexLayout onClick={hotBrandClick}>
            <HotBrands HotBrands={hotBrands} />
        </FlexLayout>
        {!isHideStar && <UnlimitType {...unlimitProps} />}
        <AllBrands {...brandsProps} />
    </div>
    <AsideLetter {...asideProps} />
    {showPop ? <PopTips key="pop-tips" tip={currentLetter} /> : null}
    {showBrandLoading ? <Loading /> : null}
</div>
            

```

FlexLayout.js(react)
![jsworke](http://pw1zvypwt.bkt.clouddn.com/9.png)

这个示例几乎涵盖了所有的规则

- 首先组件的设计是根据业务划分的，所以右侧字母导航（AsideLetter）才没有在最外层的容器组件，否则通信问题会占用一部分篇幅，事实上这是有解的
- 入口组件是容器组件，事实上把它当做一个规则就行了，业务逻辑的载体
- 除了容器组件外，其他的组件都被抽成公用的了，二手车平台类似的场景非常多
- ![jsworke](http://pw1zvypwt.bkt.clouddn.com/10.png)
- 卖车平台类似的图文混排非常多，且形态各不相同，应用场景广泛，抽！UI差异消化在组件内部，参考FlexLayout.js，给定default props
- 可提取的组件过多（业务驱动）导致通讯困难如何解决？ 那说明你需要新增可管理状态的容器组件，上例中Brand，Series也是容器组件，负责管理子组件的大小事宜
- 细粒度的考量，不是所有的都要抽的，考虑付出产出比

```
<p className="brands-title hot-brands-title">热门品牌</p> 只有一行，直接写就完了

```

- 组件抽离的过程就是无限向无状态（展示型）组件无限靠近的过程

## 通用性考量

设计组件的目的就是为了将来某一天可以通用，通⽤组件一定是与业务解耦又要服务于业务开发的,那么问题来 了,如何保证组件的通⽤性,通⽤性⾼⼀定是好事吗?

组件的形态(DOM结构)永远是千变万化的,但是其行为(逻辑)是固定的,因此通⽤用组件的秘诀之⼀就是将 DOM 结构的控制权交给开发者,组件只负责⾏为和最基本的DOM结构

这是一个显眼的栗子
某一天，你接到这样儿的需求
![jsworke](http://pw1zvypwt.bkt.clouddn.com/12.png)
开心，简单，三下五除二写完了

突然有一天又有这样儿的需求
![jsworke](http://pw1zvypwt.bkt.clouddn.com/13.png)
emm..可定制？之前的select没法用了，怎么做？要修改上一个或者再写一个吗？
一旦出现了这种情况，证明之前的组件需要重新设计了

实现通用性设计的关键一点是<font color="red">放弃对Dom的掌控</font>

### 那么问题又来了，那么多需要自定义的地方，那组件会不会很难用？

通用性设计在将Dom结构决定权交给开发者的同时指定默认值

这里是一个新鲜出炉(vue)🌰
List组件
![jsworke](http://pw1zvypwt.bkt.clouddn.com/14.png)

父组件🌰(vue)及slot

```
模版（伪代码）
<template>
<List :data="tableData[item.type]" :loading="loading" @loadMore="loadMore" :noMore="noMore">
    <a v-if="item.type == 0" slot="listOption" slot-scope="childScope" class="edit-btn" @click="edit(childScope.data)" v-bind:key="childScope.data.id">{{Status[childScope.data.status]['text']}}</a>
</List>
</template>

config(伪代码)
export const Status = {
  //....
  1: {
    label: '草稿',
    type: '',
    text: '编辑',
    class: 'note'
  }}
  //...
```

又有一个栗子(vue)
![jsworke](http://pw1zvypwt.bkt.clouddn.com/15.png)

- Dialog只负责基础的逻辑，交出控制权给到业务，至于你的业务需要什么，在容器组件（业务逻辑层）去处理

忍不住放上磐石业务的反面例子
![jsworke](http://pw1zvypwt.bkt.clouddn.com/16.png)

它现在之所以难用无非是两方面的问题
1. 不肯移交控制权
2. 没有详细的API文档

所有的业务逻辑与场景都包含在组件内部，外界只通过变量来控制，初衷是好的，但是随着业务发展，组件越来越庞大，开发者也越来越力不从心了
所以现在的状态是，随便改个UI都得考虑半天，生怕影响了别的页面，并且不断的增加判断逻辑
刚好现阶段UI改版，我们的工作量就由只改样式直接转化为推倒重来了，又没有详细的文档，工作量瞬间翻了N倍😭宝宝心里苦宝宝不说

## 善用设计模式

其实一开始，我并没有专门去套用设计模式，完全是业务驱使
你一定见到过这样儿的
![jsworke](http://pw1zvypwt.bkt.clouddn.com/17.png)
一旦这样儿的逻辑多了，那是不是就跟业务耦合了，跟业务耦合多了，那组件自然没有什么通用性了，即使我们不考虑到通用性，那写的累吧？

考虑下这样写会不会好一点

```
config（伪代码）
export const Status = {
  4: {
    label: '部分入库',
    type: '',
    text: '查看'
  }
}
模版(vue)
<a v-if="item.type == 0" slot="listOption" slot-scope="childScope" class="edit-btn" @click="edit(childScope.data)" v-bind:key="childScope.data.id">{{Status[childScope.data.status]['text']}}</a>

```

美其名曰-策略模式
世界上本没有设计模式，写的人多了，就自成一套脱颖而出进而被历史铭记了！不仅如此,一部分看似复杂的业务如果合理设计配置项，可以会为你省去一大篇js逻辑

# 我对组件化的一些感悟

像磐石这种底层的业务支持系统，离不开大量的列表，查询，编辑，详情等，我一般会花30秒搭好架子，像但不限于下面这种

![jsworke](http://pw1zvypwt.bkt.clouddn.com/18.png)

- index:模块入口（承担容器职责）
- api：整块业务的API
- components 业务组件集合

```
1. Form：表单 一般会被add.vue（编辑） 和edit.vue（详情）引用
2. List：列表
3. Search: 搜索组件
4. 其他业务中有但却没看到的基本上都已经抽离到common了 比如面包屑导航，收起展开功能等
```
- libs 页面的各种配置

## 我必须要这么做的原因

- components中的组件只是暂存，都有可能被升级成通用组件，所以命名要注意，一类的保持了统一，防止业务耦合
- bug有迹可循，数据的问题我一定从外向里排查，样式问题从里向外排查，定位问题快
- 与重复代码做斗争，时刻保持一种强迫症的心态去整理各个模块，形成自己的编码风格，进而团队风格才有可能统一

# 组件化开发方案下，为了提升团队协同效率，需要怎么做

- 把业务任务和组件任务拆开，组件的归组件，业务的归业务
- 使⽤Jira等团队管理工具管理好业务任务对组件任务的依赖，让团队可以容易易地了了解到每个业务价值的实现需要的完成的任务。
- leader需要加深对团队每个成员的了解，清楚的知道他们各自的强项，作为安排任务时的参考
- 业务优先原则，一旦业务任务依赖的所有组件任务完成，业务任务⻢上进⼊最高优先级，团队以交付业务价值为最⾼优先级
- 组件任务先于业务任务完成，未纳入业务流程前，团队需要帮助验收组件任务，输出相应的API文档并确保每个成员都是已知的，不进行无意义的劳动
- 团队成员尽量形成一致的编码套路，遇到分歧要讨论并及时调整方向

# 总结

- 对于组件设计，充分的准备很重要，但在现实世界中，切实的结果才是最重要的，组件设计也不要过度设计更不要停滞不前，该做的时候就去做，发现不好就去改
- 有空闲时间就去思考早期不够理想的代码，它可以作为我们向前发展的基础
- 你的直接责任可能是编写代码，但你的终极目标是在创建产品

# 参考链接

- https://engineering.carsguide.com.au/front-end-component-design-principles-55c5963998c9?gi=b5b86599de92
- https://segmentfault.com/a/1190000009952681
- https://juejin.im/post/5a73d6435188257a6a789d0d
- https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9
- http://www.alloyteam.com/2015/11/we-will-be-componentized-web-long-text/







