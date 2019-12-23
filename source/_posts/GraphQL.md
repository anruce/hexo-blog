---
title: 面向未来的API-GraphQL
date: 2019-12-23 16:59:30
categories: 前端
---

# 开篇

你有没有遇到以下问题

- 字段冗余

![image](http://cdn.anruence.com/%E7%A3%90%E7%9F%B3%E6%95%B0%E6%8D%AE.png)

- 若干个不得不发的HTTP请求

![image](http://cdn.anruence.com/graphql1.png)


发生这些，并不是前端er本意，但是又要承担诸如页面渲染慢等副作用而被用户诟病
究其原因，是前端在数据层面没有主动权

针对但不限于以上种种
我们需要以前端的设计者和开发者的角度出发 设计新的数据查询方式

Facebook工程师率先提出RESTful架构体系的替代方案

并且应用在了其应用中

[Facebook 使用graphql重构他们的pc站](https://www.youtube.com/watch?v=WxPtYJRjLL0&t=2s)

接下来 让我们站在巨人的肩膀上，由浅至深聊聊被称之为面向未来的API的-GraphQL

<!-- more -->
# 明确本文的边界
本文主要介绍接触GraphQL这段时间，觉得必须要掌握的一些核心 适合以下人群

- 完全没听说过GraphQL
- 听说过GraphQL的读者，想深入了解一下
- 想系统地学习GraphQL的读者
- 正在调研GraphQL技术的读者


帮助你对GraphQL建立一个统观全局的认知及原理性的解读

你可能会得到以下解答
- 重新思考RESTful
- what GraphQL
- RESTful & GraphQL
- how GraphQL
- GraphQL组成链路
- 阻碍你使用GraphQL的N个问题
- 现有应用的接入方式
- GraphQL不足
- 基于GraphQL的社区解决方案
- 小结

# 重新思考RESTful

- 接口数膨胀，需自行组合多个接口才能获取到完整的数据结构
- API文档更新不及时，联调基本靠猜
- 客户端对接口数据类型校验

```
- 除了服务端要校验客户端传来的参数，客户端自己也需要去校验服务端返回的参数
- 比如客户端要的是数组，你有没有返回数组
- 需要依赖类似出var x = data?(data.obj?data.obj.name:null):null兼容
```

- 接口字段冗余，移动/PC需求无法满足

```
- 冗余数据的返回浪费了流量
- 服务端决定有哪些数据获取方式，客户端只能挑选使用，如果数据过于冗余也只能默默接收再对数据进行处理
- 而数据不能满足需求则需要请求更多的接口

```

- 前后端字段命名规范不一致，

```
依赖数据层逐量转换
```

- 维护多版本接口

```
任何的变动都会被视为一种破坏性的改变，而破坏性改变就需要更新API的版本

```

## 我的诉求

1. 可不可以客户端要什么字段，服务端就给什么字段的值？
1. 可不可以定义一个返回数据格式与请求的数据格式的一个强类型的约束？
1. 能不能客户端可以问服务端要1、2、3这些数据，服务端一次给我返回就行？

GraphQL的出现就是为了解决RESTful的痛点


# what GraphQL

[GraphQl官网](http://graphql.org)

[GraphQL中文网](https://graphql.cn/)

> 它既是一种用于 API 的查询语言(规范) 也是一个满足你数据查询的运行时

强类型可以在查询执行之前进行验证

用于组织应用程序中数据的创建，读取，更新和删除（是的，CRUD）


脑袋里巨大的问号❓ API怎么就可以查询呢？

这正是其强大之处

> ask exactly what you want.

- 用已有的代码和技术来进行数据源管理
- 对API数据提供了一套易于理解的完整描述
- 非数据库查询语言，不是一门语言/框架 
- 不绑定任何的数据库或者存储引擎
- 使得客户端能按需获取数据，无冗余
- 让API更容易随着时间推移而演进
- GraphQL = Graph (图表化/可视化)+ QL (查询语言)
- 是一种描述客户端如何向服务端请求数据的API语法


# RESTful & GraphQL

## 资源获取

![image](http://cdn.anruence.com/rest.png)

- RESTful 用不同URL来区分资源，GraphQL 用特有的类型区分资源
- 获取相同资源REST API需要聚合多个接口
- 获取相同资源GraphQL只需一次请求获取多组数据
- GraphQL更有效率更强大更灵活，对前端更友好

## 数据获取

![image](http://cdn.anruence.com/graphql%20%E6%95%B0%E6%8D%AE%E8%8E%B7%E5%8F%96.png)

- 获取数据的方式由==这里有什么==向 ==你需要什么==转变 
- GraphQL可以简化理解成一个灵活的ajax接口
- 客户端完全自主决定获取信息的内容，服务端负责精确的返回目标数据

## GraphQL优点

- 请求你所要的数据，不多不少

```
- 向你的 API 发出一个 GraphQL 请求就能准确获得你想要的数据，不多不少。
- GraphQL 查询总是返回可预测的结果。使用 GraphQL 的应用可以工作得又快又稳，因为控制数据的是应用，而不是服务器。
```
![image](http://cdn.anruence.com/graphql-fetchdata.png)

- 获取多个资源，只需要一个请求

```
- GraphQL 查询不仅能够获得资源的属性，还能沿着资源间引用进一步查询
- 典型的 REST API 请求多个资源时得载入多个 URL
- GraphQL 可以通过一次请求就获取你应用所需的所有数据
- 即使是比较慢的移动网络连接下，使用 GraphQL 的应用也能表现得足够迅速。
```
![image](http://cdn.anruence.com/graphQL-huoqu.png)

- 描述所有可能的类型系统（强类型自身）

强类型可以在查询执行之前进行验证

```
- GraphQL API 基于类型和字段的方式进行组织，而非入口端点
- 你可以通过一个单一入口端点得到你所有的数据能力
- GraphQL 使用类型来保证应用只请求可能的数据
- 还提供了清晰的辅助性错误信息
- 应用可以使用类型，而避免编写手动解析代码。
```

- 强大的开发者工具

```
- 代码即文档
- 不用离开编辑器就能准确知道你可以从 API 中请求的数据
- 发送查询之前就能高亮潜在问题，高亮代码智能提示
- 提供了GraphiQL图形界面编写可测试的查询语句
```

- 无版本约束 平滑演进（<font color="red">GraphQL的设计精髓</font>）

> 由于仅返回明确的请求数据，所以设计良好的「GraphQL API」不存在「接口突变」的情况，这是从「版本化」到「无版本」的一个明确转变！

```
- 给你的 GraphQL API 添加字段和类型而无需影响现有查询
- 老旧的字段可以废弃，从工具中隐藏
- 通过使用单一演进版本，GraphQL API 使得应用始终能够使用新的特性，并鼓励使用更加简洁、更好维护的服务端代码

```
# GraphQL改善RESTful
了解了GraphQL的一大堆特点，我们开篇的诉求解决了吗？

![image](http://cdn.anruence.com/graphql99.png)

到这里 我们看到了GraphQL 原则上的可行性

# How GraphQL
接下来 趁热打铁 来聊聊怎么用GraphQL

![image](http://cdn.anruence.com/graphql-banner.png)

官网上特别醒目的一张图，我们可以得到如下信息

- 服务端定义好强类型的数据入参和返回的数据结构
- 客户端发送一个带有查询语句（GraphQL查询协议）的请求，定义好返回数据的格式及类型
- 返回符合客户端预期的Json字符串结果

<font color="red">再通俗一点</font>

我们拥有UI，并且需要用数据填充它，因此我们向服务器进行查询
使用传统的REST API，我们的查询将以GET请求的形式出现 借助GraphQL，我们引入了一种用于请求数据的新语法

### 一个基础的GraphQL服务

#### GraphQL服务 = 类型（schema） + 解析器 （resolve）

明确以下知识点

- 为了发出GraphQL请求，我们需要有一个GraphQL服务器
- GraphQL服务器是附加了GraphQL模式的常规HTTP服务器
- 类型系统描述了数据的类型与结构，但它只是形状，不包含真正的数据
- 通过编写Resolver函数，从而去获取真正的数据

![image](http://cdn.anruence.com/graphQL%20lizi.png)

- 服务端（或中间层）需要描述所有可能的类型系统（schema）

![image](http://cdn.anruence.com/graphql.png)

- 按你所需请求你所需要的数据，解决了不同客户端不同的渲染需求


是不是贼简单～


不知道你没有注意到 上面我们提到了<font color="red">GraphQL查询协议</font>

## GraphQL查询协议

GraphQL有三种请求方式

- query(请求)   
- mutation(修改)
- subscribe(订阅)

GraphQL的核心依赖于简单的GET或POST请求来将数据往返于客户端，而GraphQL只是一个经过修饰的GET或POST请求，通过```https://myapp.com/graphql```
之类的URL发送到GraphQL服务器


是的，虽然GraphQL确实引入了一些新的概念来组织数据进行交互，但在幕后，但GraphQL仍然依靠良好的HTTP请求来实现其神奇效果


只需要为类型系统的字段编写函数，GraphQL 就能通过优化并发的方式来调用它们

具体参照如上的demo，建议拷贝代码亲自感受一下

# GraphQL组成链路

当然了 对于开发者来说，我们无非关注两点

- 客户端做什么？

```
向服务端发送查询字符串
```

- 服务端做什么？

```
基于GraphQL构建类型系统 
定义与Query下字段对应的resolver
可以在resolver获取真正的数据
```

## 资源路径图

![image](http://cdn.anruence.com/ziyuanlujing.png)

==客户端 Schema 本质上就是一段字符串，服务端如何识别并响应这段字符串？==

## 服务端执行过程
![image](http://cdn.anruence.com/%E6%9C%8D%E5%8A%A1%E7%AB%AF.png)

拿到客户端字符串之后，依赖官方类库```graphql-js```服务端具体执行经历三个阶段

- 解析：逐字符扫描，如果不符合服务端定义的AST规范，解析过程会直接跑出语法异常，当然了，是结构化报错
- 校验：发起了查询，GraphQL会解析我们的查询语句，确保啊我们查询的结构是存在的，参数是足够的，类型是一致的，任何环节出了问题，都将返回错误信息
- 执行：验证通过后，GraphqL会根据query语句包含的字段结构一一触发对应的Resolver函数，获取查询结果，也就是说 如果前端没有查询某个字段，就不会触发该字段对应的Resolver函数，也就不会产生对数据的获取行为

注：如果Reaolver返回的数据结构，大于Schema里描绘的结构，那么多出来的部分会被忽略，这是一个合理的设计，我们可以通过控制Schema 来控制前端的数据访问权限，防止意外的将用户的隐私信息泄漏出去 

# 阻碍你使用GraphQL的N个问题
既然GraphQL那么方便，为啥没有大火呢？
结合了多篇文章，整理了若干了阻止你使用GraphQL的N个问题

一起来看一看


## GraphQL一定要操作数据库？
- 数据提供方编写GraphQL Schema
- 数据消费方编写GraphQL Query
- GraphQL只是关于schema和resolver的一一对应和调用，它并为对数据的获取方式和来源做任何假设

## GraphQL 跟 RESTful api 是对立的？
两者不仅不是对立的，还可以相互结合
事实上可以把Query 下的字段，理解为一个个 RESTful API

```
 type Query {
    hello: String,
    sayhi:String
  }
```

## GraphQL不一定需要Schema（类型系统）？
- GraphQL Type System 是一个静态的类型系统，我们可以称之为静态类型 GraphQL
- 此外，社区还有一种动态类型的 GraphQL 实践，它跟静态类型的 GraphQL 差别在于，没有了基于 Schema 的数据形状验证阶段，而是直接无脑地根据 query 查询语句里的字段，去触发 Resolver 函数，动态类型的 GraphQL 有一定的便利性，不过，它同时丧失了 GraphQL 的部分精髓

## GraphQL 一定是后端服务？
尽管绝大多数 GraphQL，都以 server 的形式存在。 但它并没有限制在后端场景

 [在浏览器中运行](https://codesandbox.io/s/youthful-mestorf-r8s38) 
 
# 现有应用的接入方式

##  暴力改造RESTful-Like模式

RESTful -> GraphQL

- 就是简单粗暴的把 RESTful API 服务，替换成 GraphQL 实现。之前有多少 RESTful 服务，重构后就有多少 GraphQL 服务，
- 默认情况下，面向两个 GraphQL 服务发起的查询是两次请求，而不是一次

```
前端需要产品数据时，从之前调用产品相关的 RESTful API，变成查询产品相关的 GraphQL。不过，需要订单相关的数据时，可能要查询另一个 GraphQL 服务
```
- 它是一个简单的一对一关系

收益甚微 选型失误

## 作为中间层

同样是API Gateway角色的GraphQL服务，在实现方式上有不同的分类 

- 1，传统意义上的后端服务（包含大量真实的数据操作和处理的 GraphQL）
- 2，<font color=red>GraphQL as BFF</font>（转发接口请求，聚合数据结果的GraphQL）

我们今天主要讨论 第二种

![image](http://cdn.anruence.com/%E6%8E%A5%E5%85%A5%E6%96%B9%E5%BC%8F.png)

- 前端不再直接调用具体的 RESTful 等接口，而是通过 GraphQL 去间接获取产品、订单、搜索等数据
- 在 GraphQL 这个中间层里，我们将各个微服务，按照它们的数据关联，整合成一个基于 GraphQL Schema 的数据关系网络。前端可以通过 GraphQL 查询语句，同时发起对多个微服务的数据的获取、筛选、裁剪等行为。
- 作为 API Gateway 的 GraphQL 服务，可以在其 Resolver 内，向前面提到的 RESTful-like 的 GraphQL 发起查询请求
- 既避免了前端需要一对多的问题，也解决了 API Gateway GraphQL 需要请求 RESTful 全量数据接口的内部冗余问题

![image](http://cdn.anruence.com/airbnb.png)

将 GraphQL schemas转化为 Thhrift IDL，再统一操作底层数据

![image](http://cdn.anruence.com/%E5%BE%AE%E6%9C%8D%E5%8A%A1..png)

# GraphQL不足
- 改造成本：后端服务需要按领域进行重构
- 存量大：迁移困难
- 数据库性能

```
- GraphQL 虽然解决将多个 HTTP 请求聚合成了一个请求，但是schema 会逐层解析方式递归获取全部数据
- 前端请求少了但是query很多 数据库设计影响日后性能
- 后端对前端改造无感知：前端修改了GraphQL的请求格式，可能会造成深层嵌套，对后端服务有较大影响

```
- 侵入性：前端利好，却需要服务端鼎力支持
- 复杂性：学习成本高

```
需要了解GraphQL一整套类型系统
```
- 典型的N+1问题

```
使用REST API 是很容易评估 ，识别和解决N+1问题的
使用GraphQL会使这个问题变得相对复杂

```
- 数据缓存问题

```
REST强制使用具有缓存机制的HTTP协议 ，可以通过它 避免活回去多余资源
GraphQL没有缓存机制，它把缓存的重任交给了用户
```
- 可见，整体来看，实际接入GraphQL并非易事，它只是一套规范，各种语言实现不一致，周边生态不够完善，需要后端配合改造，成本大，
- 除此之外，还有各种错误处理，日志上报及缓存机制的处理办法良莠不齐

正因如此，GraphQL很长一段时间还不能发挥其巨大作用

这一切 随着 Apollo 登场 正在逐步改善

# 基于GraphQL的社区解决方案

## Apollo
可以把GraphQL理解成NodeJS的http包，那么Apollo-server就类似于在前面基础上封装出来的框架

### Apollo-Client
web，iOS，Android三端的实现
### Apollo-Server
koa，express等NodeJsWeb的实现


还提供了如下能力

### 错误处理
接口格式五花八门，错误处理也没有统一的方案，Apollo会将所有的错误内容格式化统一的错误信息，从此可以摆脱后端带来的束缚，方便我们在前端去处理。

### 模块化
借助GraphQL的makeExecutableSchema和mergeSchemas方法，能够按模块去编写类型定义及resolve，最后使用mergeSchemas将他们合并到一起
### 错误监控   
Apollo server提供formatError，formatResponse，能够细化到每一次请求，每一次错误的发生，方便我们去上报日志及错误

工作流程

![image](http://cdn.anruence.com/%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88wee.png)

接入架构设想

![image](http://cdn.anruence.com/jiagoushaxiang.png)

- 通过复用现有的REST接口，做到无需后端配合改造
- 在我们开发的项目里，由于会对接不同的后端团队，伴随着一些历史遗留问题，接口格式五花八门，错误处理也没有统一的方案，Apollo会将所有的错误内容格式化统一的错误信息，从此可以摆脱后端带来的束缚，方便我们在前端去处理
- 在开发大型应用中，模块是是必不可少的。借助GraphQL的makeExecutableSchema和mergeSchemas方法，能够按模块去编写类型定义及resolve，最后使用mergeSchemas将他们合并到一起
- 由于接入了node server，那么我们需要监控错误以及请求日志等内容，Apollo server提供formatError，formatResponse，能够细化到每一次请求，每一次错误的发生，方便我们去上报日志及错误

接入成果设想 
![image](http://cdn.anruence.com/chengguoshexiang.png)
# 小结
- <font color=red>更准确的获取你想要的数据</font> - 核心诉求
- 控制数据的是应用而非服务器

```
在应用层对数据模型的抽象
```

- 将你需要的数据汇聚成数据网格

```
前端不能通过一次查询直接得到自己所需要的数据，Graphql的查询不仅指定了要查询的信息同时给出了期望的数据结构

```

- 应对复杂场景的一种新思路 thinking in graph

```
设计 GraphQL接口更像是在建立资源与资源之间的关系，并最终得到一个单一内聚图的过程 GraphQL 给了我们一种基于「图」的设计思路
```

