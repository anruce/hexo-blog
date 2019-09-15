---
title: 看完这篇，我奶奶都会设计组件了
date: 2019-07-29 01:40:30
tags: 组件设计
categories: 前端
---

# 聊聊组件设计

# 前言

[掘金链接](https://juejin.im/post/5d566e82f265da03f77e653c)

组件化思想并不是前端独有的，但却是前端技术的延伸

随着三大框架崛起，前端组件化逐渐成为前端开发的迫切需求，一种主流，一种共识，它不仅提高开发效率，同时也降低了维组件内聚原则护成本
开发者们不需要再面对一堆晦涩难懂的代码，转而只需要关注以组件⽅式存在的代码⽚段

这是一场新的挑战！

## 文章开始之前，明确本文的边界
- 从前端工程谈到组件化开发
- 组件的设计原则
- 组件的职能划分及利弊
- 组件设计的边界
- 落实到具体业务中如何做
- 一些感悟
- 总结

# 一个面试题引发的思考

```
面试官通常会问 写过前端通用组件吗？
```

<!-- more -->

你可能会自信的表示： sure！

emm..是的吗？

# 从前端工程谈到组件化开发


前端工程经历的三个阶段

## 1. 库/框架选型

![image](http://pw1zvypwt.bkt.clouddn.com/ku.png)
确定技术选型，为项目节省许多工程量
后来三大框架的横空出世，解放了不少生产力

## 2. 简单构建优化

![image](http://pw1zvypwt.bkt.clouddn.com/tools.png)

解决完开发效率，还需要兼顾运行性能，
故而选择某种构建工具，对代码进行压缩，校验，之后再以页面为单位进行简单的资源合并

## 3. JS/CSS模块化开发

![image](http://pw1zvypwt.bkt.clouddn.com/jsmokuaihua.png)

解决了基本开发效率和运行效率之后，开始考虑维护效率了

分而治之（以分解降低复杂度）是软件工程中的重要思想，是复杂系统开发和维护的基石,<font color="red">模块化就是前端的分治手段</font>

因此,模块化强调的是**拆分**，最大的价值就是分治，意味着不管你将来是否要复用这块儿代码，都有将他们拆成一个模块的理由

> 将一个大问题，不断的拆解为各个小问题进行分析研究，然后再组合到一起(分而治之原则)


### 模块化的方案

- JS模块化

```
无模块化->函数写法->对象写法->自执行函数->CommonJS/AMD/CMD->ES6 Module
```

- CSS模块化

```
css模块化是在less，sass等预处理器的支持下实现的
```

### 做到这些就够了吗？
当然是不够的

模块化强调的是**拆分**，无论是从业务角度还是从架构、技术角度，模块化首先意味着将代码、数据等内容按照其职责不同分离

单纯的横向拆分业务功能模块有一些问题
![](http://pw1zvypwt.bkt.clouddn.com/2.png)

- 面向过程的代码 随着业务的发展不利于维护

```
随着业务发展，”过程线“也会越来越长，其他项目成员根据各自需要，在”过程线“ 加插各自逻辑，最终这个页面的逻辑变得难以维护
我们需要摆脱【一泻而下】式的代码编写
```
- 仅仅有JS/CSS模块化是不够的，UI（页面）的分治也比较迫切

```
除了JS和CSS，界面也需要拆分，如何让模块化思想融入HTML语言

```


## <font color=red>4. 组件化开发（本文重点）</font>

### 组件化开发的演变
在大肆宣扬组件化开发概念之前，也经历了寻求组件化最佳实践的阶段

#### 页面结构模块化

![image](http://pw1zvypwt.bkt.clouddn.com/jimu)
简单来说就是把页面想象成乐高机器人，需要不同零件组装，然后将各个部分拼到一起

落实到实际开发中像这样
![](http://pw1zvypwt.bkt.clouddn.com/jiegouhua.png)

我们可以发现

- 页面pageModel包含了 `tabContainer`，`listContainer` 和 `imgsContainer` 三个模块
- 我们根据不同的业务逻辑封装了不同类型的model
- 每个model有自己的数据，模板，逻辑，已经算是一个完整的功能单元

<font color=red>咦？嗅到一丝组件化的味道</font>

#### N年前微软的组件化的解决方案 HTML Component
历史总有遗🐖

早在N年前微软提出过一套解决方案，名为`HTML Component`

![jsworke](http://pw1zvypwt.bkt.clouddn.com/4.jpg)

事实上已经是一个比较完整的组件化方案了，但最后却没能进入标准，从今天的角度看，它可以说是生不逢时

#### WebComponents 标准

当时”所谓的组件“

- 此时的组件基本上只能达到某个功能单元上的集合，资源都是资源都是松散地分散在三种资源文件中
- 而且组件作用域暴露在全局作用域下，缺乏内聚性很容易就会跟其他组件产生冲突（如最简单的 css 命名冲突）

于是 W3C 按耐不住了，制定一个 WebComponents 标准，为组件化的未来指引了明路

大致四部分功能

- `<template>` 定义组件的 HTML模板能力
- Shadow Dom 封装组件的内部结构，并且保持其独立性
- Custom Element 对外提供组件的标签，实现自定义标签
- import 解决组件结合和依赖加载

我们思考一下，可行的实践化方案需要具备哪些能力

- 资源高内聚（组件资源内部高内聚，组件资源由自身加载控制）
- 作用域独立（内部结构密封，不与全局或其他组件产生影响）
- 自定义标签（定义组件的使用方式）
- 可相互组合（组件间组装整合）
- 接口规范化（组件接口有统一规范，或者是生命周期的管理）

#### 三大框架出现
今天的前端生态里面 React，Angular和Vue三分天下，即使它们定位不同，但核心的共同点就是提供了组件化的能力，算是目前是比较好的组件化实践

##### 1. Vue.js采用了JSON的方法描述一个组件

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

##### 2. React.js发明了JSX，把CSS和HTML都塞进JS文件里

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

##### Angular.js选择在原本的HTML上扩展


```
<input type="text" ng-model="firstname">

var app = angular.module('myApp', []);
app.controller('formCtrl', function($scope) {
    $scope.firstname = "John";
});
```

### 标准下的资源整合

![image](http://pw1zvypwt.bkt.clouddn.com/sss.png)

#### 具有以下特点
- 每个组件对应一个目录，组件所需的各种资源都在这个目录下就近维护；（最具软件工程价值）
- 页面上的每个独立的可视/可交互区域视为一个组件；
- 由于组件具有独立性，可以自由组合；
- 页面是组件的容器，负责组合组件形成功能完整的界面；
- 当不需要某个组件，或者想要替换组件时，可以整个目录删除/替换

### 应用结构图

![image](http://pw1zvypwt.bkt.clouddn.com/niubi.png)


- 分子是由原子组成的，分子分成原子，原子也可以重新组合成新的分子
- 一个界面是由独立的分子组件搭建而成，分子组件由原子元件构成，这些原子可通过不同的组合方式，组成新分子组件，继而重组构成新的界面


### 模块化与组件化对比

如果你去网上搜【模块和组件的异同】
可能会得到截然不同的答案，大部分描述的都是片面的

它们之间的关系可以从以下三个方面分析：

#### 从整体概念来讲
- 模块化是一种分治的思想，诉求是解耦，一般指的是js模块，比如用来格式化时间的模块
- 组件化是模块化思想的实现手段，诉求是复用，包含了`template`，`style`，`script`，script又可以由各种模块组成

#### 从复用的角度来讲
- 模块一般是项目范围内按照项目业务内容来划分的，比如一个项目划分为子系统、模块、子模块，**代码分开就是模块**
- 组件是按照一些小功能的通用性和可复用性抽象出来的，可以跨项目，是可复用的模块

#### 从历史发展角度来讲
随着前端开发越来越复杂、对效率要求越来高，由项目级模块化开发，进一步提升到通用功能组件化开发，**模块化是组件化的前提，组件化是模块化的演进**

# 组件的设计原则

组件化方案下，我们需要具有组件化设计思维，它是一种【整理术】帮助我们高效开发整合


1. 标准性

```
任何一个组件都应该遵守一套标准，可以使得不同区域的开发人员据此标准开发出一套标准统一的组件

API尽量和已知概念保持一致
```

2. 独立性

```
遵循单一职责原则，保持组件的纯粹性
属性配置等API对外开放，组件内部状态对外封闭，尽可能的少与业务耦合

避免暴露组件内部实现

入口处检查参数的有效性，出口处检查返回的正确性
```

3. 复用与易用，适用SPOT法则 

```
UI差异，消化在组件内部（注意并不是写一堆if/else）
输入输出友好，易用

Single Point Of Truth，就是尽量不要重复代码，出自《The Art of Unix Programming》
```

4. 避免直接操作DOM，避免使用ref

```
使用父组件的 state 控制子组件的状态而不是直接通过 ref 操作子组件

```

5. **无环依赖原则(ADP)**

设计不当导致环形依赖示意图
![image](http://pw1zvypwt.bkt.clouddn.com/1111111111.png)

**影响**

- 组件间耦合度高，集成测试难
- 一处修改，处处影响，交付周期长
- 因为组件之间存在循环依赖，变成了“先有鸡还是先有蛋”的问题

**那倘若我们真的遇到了这种问题，就要考虑如何处理？**

<font color=red>消除环形依赖</font>

> 我们的追求是沿着逆向的依赖关系即可寻找到所有受影响的组件

创建一个共同依赖的新组件

![image](http://pw1zvypwt.bkt.clouddn.com/xiaochu.png)



6. 稳定抽象原则(SAP)


```
- 组件的抽象程度与其稳定程度成正比，
- 一个稳定的组件应该是抽象的（逻辑无关的）
- 一个不稳定的组件应该是具体的（逻辑相关的）
- 为降低组件之间的耦合度，我们要针对抽象组件编程，而不是针对业务实现编程
```
7. 避免冗余状态

```
如果一个数据可以由另一个 state 变换得到，那么这个数据就不是一个 state，只需要写一个变换的处理函数，在 Vue 中可以使用计算属性

如果一个数据是固定的，不会变化的常量，那么这个数据就如同 HTML 固定的站点标题一样，写死或作为全局配置属性等，不属于 state

如果兄弟组件拥有相同的 state，那么这个state 应该放到更高的层级，使用 props 传递到两个组件中

```

8. 合理的依赖关系

```
父组件不依赖子组件，删除某个子组件不会造成功能异常

```

9. 扁平化参数

```
除了数据，避免复杂的对象，尽量只接收原始类型的值
```

10. 良好的接口设计
```
把组件内部可以完成的工作做到极致，虽然提倡拥抱变化，但接口不是越多越好

如果常量变为 props 能应对更多的场景，那么就可以作为 props，原有的常量可作为默认值。

如果需要为了某一调用者编写大量特定需求的代码，那么可以考虑通过扩展等方式构建一个新的组件。

保证组件的属性和事件足够的给大多数的组件使用。

```

# 组件的职能划分

那有了组件设计的“API”，就一定能开发出高质量的组件吗？

> 组件最大的不稳定性来自于展现层，一个组件只做一件事，基于功能做好职责划分

根据以往经验，我将组件分为以下几类

- 基础组件（通常在组件库里就解决了）
- 容器型组件（Container）
- 展示型组件（stateless）
- 业务组件
- 通用组件
  - UI组件
  - 逻辑组件
- 高阶组件（HOC）

<font color=red>除容器组件外，尽量保证组件都是stateless的，这并不冲突！</font>

## 基础组件

为了让开发者更关注业务逻辑，涌现出了很多优秀的UI组件库
比如`antd`，`element-ui`，我们只需要调用API便能满足大部分的业务场景，前端角色后置了，开发变得更简单了

## 容器型组件
一个容器性质的组件，一般当作一个业务子模块的入口，比如一个路由指向的组件

![image](http://pw1zvypwt.bkt.clouddn.com/RONGQIZUJIAN.png)


### 特点
- 容器组件内的子组件通常具有业务或数据依赖关系
- 集中/统一的状态管理，向其他展示型/容器型组件提供数据（充当数据源）和行为逻辑处理（接收回调）
- 如果使用了全局状态管理，那么容器内部的业务组件可以自行调用全局状态处理业务
- 业务模块内子组件的通信等统筹处理，充当子级组件通信的状态中转站
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

主要表现为组件是怎样渲染的，就像一个简单的模版渲染过程

![image](http://pw1zvypwt.bkt.clouddn.com/zhanshixing.png)

### 特点

- 只通过props接受数据和回调函数，不充当数据源
- 可能包含展示和容器组件 并且一般会有Dom标签和css样式
- 通常用props.children(react) 或者slot(vue)来包含其他组件
- 对第三方没有依赖（对于一个应用级的组件来说可以有）
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

通常是根据最小业务状态抽象而出，有些业务组件也具有一定的复用性，但大多数是一次性组件

![image](http://pw1zvypwt.bkt.clouddn.com/yewu.png)

## 通用组件
可以在一个或多个APP内通用的组件
### UI组件
- 界面扩展类组件，比如弹窗

![image](http://pw1zvypwt.bkt.clouddn.com/UI.png)

特点：复用性强，只通过 props、events 和 slots 等组件接口与外部通信


#### 表现形式🌰（vue）


```
<template>
  <div class="empty">
    <img src="/images/empty.png" alt>
    <p>暂无数据</p>
  </div>
</template>
```
### 逻辑组件
- 不包含UI层的某个功能的逻辑集合

## 高阶组件（HOC）
高阶组件可以看做是函数式编程中的组合
可以把高阶组件看做是一个函数，他接收一个组件作为参数，并返回一个功能增强的组件

高阶组件可以抽象组件公共功能的方法而不污染你本身的组件
比如 `debounce` 与 `throttle`

用一张图来表示

![jsworke](http://pw1zvypwt.bkt.clouddn.com/hoc.png)

React中高阶组件是比较常用的组件封装形式，Vue官方内置了一个高阶组件[keep-alive](https://github.com/vuejs/vue/blob/dev/src/core/components/keep-alive.js)，但并未推荐使用HOC :(

**猜想原因**

- React：写组件就是在写函数，函数拥有的功能组件都有
- Vue：更像是高度封装的函数，能够让你轻松的完成一些事情的同时损失一定的灵活性，你需要按照一定规则才能使系统更好的运行


### 表现形式🌰（react）

品牌车系滑动的动画

![](http://pw1zvypwt.bkt.clouddn.com/reacthoc.png)


## 各类组件协同组成业务模块

![image](http://pw1zvypwt.bkt.clouddn.com/MOKUAITU.png)

## 容器/展示组件

对比图

![image](http://pw1zvypwt.bkt.clouddn.com/rongqitu.png)

### 引入容器组件的概念只是一种更好的组织方式

- 各司其职，不易出错，即使出错，也能快速定位问题
- 容器组件，一个载体的存在
- 展示型组件不与store耦合，通过props接口来定义所需的数据和方法，**复用性与正确性更能保证**

```
展示型组件直接和store通信的话，那么它就会收到限制，因为你在store里面的字段已经限制他的使用次数和使用的位置
```

<font color="red">既然如此，那我什么时候引入容器组件，什么时候引入展示组件</font>

### 引入容器组件的时机

优先考虑展示组件，当你意识到有一些中间组件不使用它继承的props而是转而传递给他们的子级，每次子级组件需要更多数据时，都需要“路过”这些中间组件时就要考虑引入容器组件！

两者的区别并没有被严格定义，<font color="red">事实上不在技术上而是目的性上</font>

#### 这里有几个供参考的点

- 容器组件倾向于有状态，展示组件倾向于无状态，这不是硬性规定，它们都是可以有状态的
- 不要把分离容器组件和展示组件当做教条，如果你不确定该组件是容器组件还是展示组件，就暂时不要分离，写成展示组件，也许是为时尚早，别着急！
- 这是一个持续的重构过程，不用试图一次就把它做好，习惯这种模式就会培养起一种直觉，知道何时引入容器 就像你知道何时封装一个函数那样！

## 进行组件职能划分的利弊

### 优点

- 更好的关注分离

```
用这种方式写组件，你可以更好的理解你的app和你的ui，甚至会逐渐形成你自己的开发套路
```

- 复用性高

```
一个组件只做一件事，解除了组件的耦合带来更高复用性
```

- 它是app的调色版，设计师可以随意调整它的ui而不用改变app的逻辑
- 这会强制你提取“布局组件”，达到更高的易用性
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

### 所谓的缺点

- 设计组件初期会增加一些学习成本
- 由于需要封装一个容器，包装一些数据和接口给展示组件，**会增加一些工作量**
- 在展示组件内对props的声明会带来少量的工作

<font color=red>长远来看，利大于弊，特别是项目初期，一定要有一个好的设计习惯</font>

# 组件设计的边界

物极必反，跃跃欲试前，常常思考以下几个问题以引导完善组件的设计


## 页面层级不宜嵌套超过三层，切勿过度设计

```
原则上组件嵌套超过三层，数据传递的过程就会变得相对复杂
```
## 这个组件可否（有必要）再分？

```
划分粒度的根据实际情况权衡，太小会提升维护成本，太大又不够灵活和高复用性

是否打破了一个逻辑上有意义的实体，倘若抽离的话，这个代码被复用的概率有多大？

如果它只是几行代码，那么最终可能会创建更多的代码来分离它，有必要吗？我这么做的好处是否超过了成本？

如果你当前的逻辑不太可能出现在其他地方，那么将它嵌入其中更好，如果需要，你可以随时抽离，毕竟组件化没有终点

每一个组件都应该有其独特的划分目的的，有的是为了复用实现，有的是为了封装复杂度清晰业务实现
组件划分的依据通常是业务逻辑、功能，要考虑各组件之间的关系是否明确，及可复用度

```

## 性能会受到影响吗？

```
如果状态频繁更改，并且当前在一个较大且关系比较紧密的组件里，为了避免性能受到影响最好抽离出来 与diff策略相关
```

## 这个组件的依赖是否可再缩减？

缩减组件依赖可以提高组件的可复用度

## 这个组件是否对其它组件造成侵入？

- 封装性不足或自身越界操作，就可能对自身之外造成了侵入
- 一个组件不应对其它兄弟组件造成直接影响

```
常见的一种情况是：组件运行时对window对象添加resize监听事件以实现组件响应视窗尺寸变化事件

最优的方案：组件提供刷新方法，由父组件实现调用
次优的方案：组件destroy前清理恢复
```

## 接口设计是否兼容大部分场景？


```
需要考虑需要适用的不同场景，在组件接口设计时进行必要的兼容
```

## 当别人使用这个组件时，会怎么想？

```
接口设计符合规范和大众习惯，尽量让别人用起来简单易上手，易上手是指更符合直觉
```

## 假如业务需要不需要这个功能，是否方便清除？

```
各组件之前以组合的关系互相配合，也是对功能需求的模块化抽象，当需求变化时可以将实现以模块粒度进行调整
```


上文提到的各种准则仅仅描述了一种开发理念，也可以认为是一种开发规范，倘若你认可这规范，对它的分治策略产生了共鸣，那我们就可以继续聊聊它的具体实现了

问自己一个问题

<font color=red>你心中的相对完美的组件是什么样子的？</font>


# 落实到具体业务中如何做


## 划分依据

明确你的组件划分依据，目前是两种

- 根据业务划分
- 根据技术划分

1. 我更多的是根据业务去设计我应用中的组件树，可能会画个草图或xmind，它可以帮我统观全局
2. 明确各个组件的边界，内部state的设计，props的设计以及与其他组件的关系（需要回调出去的事件）
3. 明确各个组件的定位与职能划分，设计好父子组件、兄弟组件的通信机制
4. 搭架子
5. 架子有了，开始填空

## 切割模版（页面结构模块化）

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

好处：
- 同之前的方式相比，这个微妙的改进是革命性的
- 解决了测试困难，维护困难的问题

问题：
- 没有解决代码重复的问题，这种按模块划分，复用性低

但我看过很多项目的代码，就是这么干的，认为自己做了组件化，抽象的还不错(@_@)

### 组件抽象

它们有相似的外层，part2和part3更有相似的titlebar，除了业务内容，完全就是一模一样

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

- UI差异在哪里定义？

在业务逻辑层处理

```
首先要明确一点，这些差异并不是组件本身造成的，是你自己的业务逻辑造成的，所以容器组件（父组件）应该为此买单
```

- 数据差异在哪里定义？

结合组件本身和业务上下文将差异合理的消除在内部

```
比如part3中，其他的part只有一个类似更多>>的link，但是它却有多个(一居，二居...)
这里我推荐将这种差异体现在组件内部，设计方法也很多：
比如可以将link数组化为links；
比如可以将更多>>看作是一个default的link，而多余的部分则是用户自定义的特殊link，这两者合并组成了links。用户自定义的默认是没有的，需要引用组件时进行传入。


```

- 组件命名规则？

组件设计初期，就应该拥有不耦合业务的名字

```
一个通用的或者说未来可能通用的，要有相对合理的命名，比如 Search，List,尽量不要出现与业务耦合过深的业务名词，通用组件与业务无关，只与自身抽象的组件有关
我们在设计组件初期，就应该有这种思想，等到真正可以抽出公用组件了，再去苦逼的名改名字？
库通常都想让广大开发者用，我们在设计组件时，可以降低标准到先做到你的整个APP中通用
```

## 组件划分细粒度的考量（抽之有度）

组件设计规则明明白白写着我们要遵循单一职责原则，这也带来了上文聊过的<font color="red">过度抽象（组件化）</font>的问题，我们结合具体的业务聊一下

![jsworke](http://pw1zvypwt.bkt.clouddn.com/7.png)


要实现徽章组件，它有两部分组成
- 按钮
- 右上角提示（小红点/icon）

两者都是符合单一职责的，可以将其抽离成一个独立组件，但是通常不要这么做

```
因为同一个app的风格必将是统一的，除此之外没别的应用场景了，就像上文所说的，抽离组件之前，多问自己为什么以及投入/产出比，没有绝对的规则
```


### tips

<font color="red">单一职责组件要建立在可复用的基础上，对于不可复用的单⼀职责组件我们仅仅作为独立组件的内部组件即可</font>

### 某二手车网站体现其细粒度的例子

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

![jsworke](http://pw1zvypwt.bkt.clouddn.com/10.png)

- 卖车平台类似的图文混排多且形态各不相同，应用场景广泛，抽！UI差异消化在组件内部，参考FlexLayout.js，给定default props
- 可提取的组件过多（业务驱动）导致通讯困难如何解决？ 那说明你需要新增可管理状态的容器组件，上例中Brand，Series也是容器组件，负责管理子组件的大小事宜
- 细粒度的考量，考虑付出产出比

```
<p className="brands-title hot-brands-title">热门品牌</p> 只有一行，直接写就完了

```

- 组件抽离的过程就是无限向无状态（展示型）组件无限靠近的过程

## 通用性考量

组件的形态(UI)永远是千变万化的,但是其行为(逻辑)是固定的,因此通用组件的秘诀之⼀就是<font color=red>将DOM 结构的控制权交给开发者,组件只负责⾏为和最基本的DOM结构</font>

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

难用无非是两方面的问题
1. 不肯移交控制权
2. 没有API文档

所有的业务逻辑与场景都包含在组件内部，外界只通过变量来控制，初衷是好的，但是随着业务发展，组件越来越庞大，开发者也越来越力不从心了


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

世界上本没有设计模式，写的人多了，就自成一套脱颖而出进而被历史铭记了！不仅如此,一部分看似复杂的业务如果合理设计配置项，可以会为你省去一大篇js


# 一些感悟

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

## 具体体现（磐石刚刚重构的模块）


采购模块结构图

![image](http://pw1zvypwt.bkt.clouddn.com/2222323232323.png)

Form

![image](http://pw1zvypwt.bkt.clouddn.com/edit.png)

Edit

![image](http://pw1zvypwt.bkt.clouddn.com/form.png)

无论有多少种状态，只在edit这层容器维护


## 要这么做的原因

- components中的组件只是暂存，都有可能被升级成通用组件，所以命名要注意，一类的保持了统一，防止业务耦合
- bug有迹可循，数据的问题我一定从外向里排查，样式问题从里向外排查，定位问题快
- 与重复代码做斗争，时刻保持一种强迫症的心态去整理各个模块，形成自己的编码风格，进而团队风格才有可能统一

# 总结

- 对于组件设计，充分的准备固然，但在现实世界中，切实的结果才是最重要的，组件设计也不要过度设计更不要停滞不前，该做的时候就去做，发现不好就去改
- 有空闲时间就去思考早期不够理想的代码，它可以作为我们向前发展的基础
- 技术在变迁，但组件化的核心并没有改变，目标仍然是在API设计尽可能接近原生的情况下完成复用、解耦、封装、抽象的目标，最终服务于开发，提高效率降低错误率
* 组件化是对实现的分层，是更有效地代码组合方式
* 组件化是对资源的重组和优化，从而使项目资源管理更合理，方便拔插、方便集成、方便删除、方便删除后重新加入
* 这种化繁为简的思想在后端开发中的体现是微服务，而在前端开发中的体现就是组件化
- 组件化有利于单元测试与自测效率对重构较友好
- 新人加入可以直接分配组件进行开发、测试，而非需要熟悉整个项目，可以从一个组件的开发使新进人员比较快速熟悉项目、了解到开发规范
- 你的直接责任可能是编写代码，但你的终极目标是在创建产品

最后说一句

<font color="red">组件化没有终点，day day up</font>

# 参考链接

- https://engineering.carsguide.com.au/front-end-component-design-principles-55c5963998c9?gi=b5b86599de92
- https://segmentfault.com/a/1190000009952681
- https://juejin.im/post/5a73d6435188257a6a789d0d
- https://medium.com/merrickchristensen/function-as-child-components-5f3920a9ace9
- http://www.alloyteam.com/2015/11/we-will-be-componentized-web-long-text/