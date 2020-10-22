---
title: 使用Node.js逐步建立多路复用的RPC通道
date: 2020-03-23 00:22:49
tags:
- Node.js
categories:
- 前端
---

# 前言

依托Nodejs使用 `Buffer`  `net`等模块逐步构建满足应用场景的RPC通道

# 1. RPC调用

RPC

全称 `Remote Procedure Call` 翻译成中文：远程过程调用

emm.. 我只是个小前端..

![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1584877960225&di=1beedaa2bf83dbc8d438dc36164af7d3&imgtype=0&src=http%3A%2F%2Fimg3.cache.netease.com%2Fphoto%2F0005%2F2013-02-20%2F8O5Q4R5K0AI90005.jpg)


## 1.1 如何通俗的解释是RPC？

### 1.1.1 本地过程调用

```
我现在在家里，我需要洗衣服，就把衣服扔到洗衣机洗了
```

### 1.1.2 远程过程调用(RPC)

```
我现在在逛街，我需要洗衣服，于是给在家里的男票打个哥电话，他把衣服扔到洗衣机洗了

那么我就实现了RPC调用！！
```
<!-- more -->
## 1.2 从前端的角度上来理解RPC调用？

从我们熟悉的Ajax入手，它与RPC调用类似，我们来对比一下

### 1.2.1 相同点
#### 1.2.1.1 都是两个计算机之间的网络通信

- Ajax：客户端和服务端的通信
- PRC：服务器和另外一台服务器的通信

看图说话

![image](http://cdn.anruence.com/rpc.png)

#### 1.2.1.2 需要双方约定一个数据格式


### 1.2.2 不同点
#### 1.2.2.1 不一定使用DNS作为寻址服务

- Ajax 是发一个HTTP请求，使用DNS进行寻址服务

请求过程
![image](http://cdn.anruence.com/dns.png)

- RPC通信一般是在内网进行请求，使用特有的服务（比如id）
请求过程
![image](http://cdn.anruence.com/rpcxunzhi.png)

#### 1.2.2.2 应用层协议一般不使用HTTP
Ajax：使用HTTP文本协议（html,json）
RPC:服务端之间的通信，对效率要求更高所以使用一些二进制协议取代HTTP，二进制协议性能上存在优势

- 更小的数据包
- 更快的编码速率

#### 1.2.2.3 基于TCP/UDP协议
- 浏览器调用（Ajax）使用 TCP是遵循HTTP的规范
- RPC调用使用了TCP多种通信方式
    1.   单工通信（独木桥）
    
类比独木桥，两岸同一时间内只能有一方通过
![image](http://cdn.anruence.com/dangong.png)
    1.   半双工通信（轮番单工通信，独木桥）
    
![image](http://cdn.anruence.com/banshuanggongtongxin.png)
    1.   全双工通信
![image](http://cdn.anruence.com/quanshuanggongtongxiin.png)

## 2. 使用Buffter编解码二进制数据包

用来处理TCP链接中的流以及文件系统中的数据

[Buffer官方介绍](http://nodejs.cn/api/buffer.html)


### 2.1 buffter创建

```
const buffter1 = Buffer.from('yishu')
const buffter2 = Buffer.alloc(20)
console.log(buffter1)
console.log(buffter2)

<Buffer 79 69 73 68 75>
<Buffer 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00>
```
### 2.2 buffter读写

二进制协议：不同字段塞在二进制流中的不同位置

基本操作

```
buffter2.writeInt8(12,1)
```

图示编码二进制包
![image](http://cdn.anruence.com/lll.png)

**图解：**
前三位代表一个字段，中间代表一个字段，后面又代表一个字段
所以，编码二进制包的时候，我们需要执行三次write写操作

看起来还是稍许麻烦嗷

![image](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=908304341,3029384854&fm=26&gp=0.jpg)

有木有像Json格式化方式如此简单的编码方式


答案：有！

[protocol-buffers-npm包](https://www.npmjs.com/package/protocol-buffers)

使用示例

```
test.proto

message Test {
  required int32 id  = 1;
  required string payload = 2;
}
```
index.js
```
const fs = require('fs');
var protobuf = require('protocol-buffers')

// pass a proto file as a buffer/string or pass a parsed protobuf-schema object
var messages = protobuf(fs.readFileSync(__dirname + '/test.proto','utf-8'))
var buf = messages.Test.encode({
  id: 42,
  payload: 'hello world'
})

console.log(buf) // should print a buffer
{/* <Buffer 08 2a 12 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64> */}


console.log(messages.Test.decode(buf))
// { id: 42, payload: 'hello world' }
```

明显发现

- 更直观
- 更好维护
- 更便于合作


正是所期盼的这样鸭～
![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1584891026316&di=474986ffbe473f99d8664a54b1f89076&imgtype=0&src=http%3A%2F%2Fp5.pccoo.cn%2Fwinccoo%2F20170317%2F2017031716023399665328.gif)

## 3. 建立多路复用的RPC通道

### 3.1 需求1 实现单工通信通道

client.js

```
const net = require('net');
const socket  = new net.Socket({});

socket.connect({
    host:'127.0.0.1',
    port:6002
})

socket.write('good!maying')  //单工通信
```
server.js

```
const net = require('net');

net.createServer((socket)=>{
    socket.on('data',function(buffer){
        console.log('buffer',buffer,buffer.toString())
    })
})
.listen(6002)
```
得到结果

![image](http://cdn.anruence.com/goodmoring.png)

这里实现了TCP通信方式之一 <font color=red>单工通信</font>

### 3.1 需求2 实现半双工通信通道

#### 3.1.1 客户端和服务器有来有回

- 客户端请求一个正常数据
- 服务端返回一个相应的数据

#### 3.1.2 重点逻辑
在单工通信模式下

- client端：发请求数据，等到服务器端返回结果之后，再次请求
- server端：接收到请求后，匹配返回

#### 3.1.3 代码

client.js

```
const net = require('net');

// 创建socket
const socket = new net.Socket({});

// 连接服务器
socket.connect({
    host: '127.0.0.1',
    port: 6002
});


const lessonids = [
    "136797",
    "136798",
    "136799",
    "136800",
    "136801",
    "136803",
    "136804",
    "136806",
    "136807",
    "136808",
    "136809",
    "141994",
    "143517",
    "143557",
    "143564",
    "143644",
    "146470",
    "146569",
    "146582"
]

let id = Math.floor(Math.random() * lessonids.length);

// 往服务器传数据
socket.write(encode(id));

socket.on('data', (buffer) => {
    console.log(buffer.toString())

    // 接收到数据之后，按照半双工通信的逻辑，马上开始下一次请求
    id = Math.floor(Math.random() * lessonids.length);
    socket.write(encode(id));
})

// 把编码请求包的逻辑封装为一个函数
function encode(index) {
    buffer = Buffer.alloc(4);
    buffer.writeInt32BE(
        lessonids[index]
    );
    return buffer;
}

```

server.js


```
const net = require('net');

// 创建tcp服务器
const server = net.createServer((socket) => {

    socket.on('data', function(buffer) {
        // 从传来的buffer里读出一个int32
        const lessonid = buffer.readInt32BE();

        // 50毫秒后回写数据
        setTimeout(()=> {
            socket.write(
                Buffer.from(data[lessonid])
            );
        }, 50)
    })

});

// 监听端口启动服务
server.listen(6002);

const data = {
    136797: "01 | 课程介绍",
    136798: "02 | 内容综述",
    136799: "03 | Node.js是什么？",
    136800: "04 | Node.js可以用来做什么？",
    136801: "05 | 课程实战项目介绍",
    136803: "06 | 什么是技术预研？",
    136804: "07 | Node.js开发环境安装",
    136806: "08 | 第一个Node.js程序：石头剪刀布游戏",
    136807: "09 | 模块：CommonJS规范",
    136808: "10 | 模块：使用模块规范改造石头剪刀布游戏",
    136809: "11 | 模块：npm",
    141994: "12 | 模块：Node.js内置模块",
    143517: "13 | 异步：非阻塞I/O",
    143557: "14 | 异步：异步编程之callback",
    143564: "15 | 异步：事件循环",
    143644: "16 | 异步：异步编程之Promise",
    146470: "17 | 异步：异步编程之async/await",
    146569: "18 | HTTP：什么是HTTP服务器？",
    146582: "19 | HTTP：简单实现一个HTTP服务器"
}

```
![image](http://cdn.anruence.com/half.gif)

### 3.2 需求1 实现全双工通信通道

client端自由发送数据包，无需等待server端返回

#### 3.2.1 解决半双工通信的问题

- 半双工通信进行并发容易导致请求包和响应包时序错乱

看图解释一下


![image](http://cdn.anruence.com/qwwqw.png)

1. client同时发送id1，id2的请求
1. server端处理...
1. server返回id2的处理结果
1. server返回id1的处理结果

client端如何将两个请求和返回数据对应呢？

如果根据返回的时间来进行匹配，就会造成错乱

![image](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=2814248882,3142720346&fm=26&gp=0.jpg)

<font color=red>如何解决?</font>


这正是全双工通信模式要解决的问题

将请求包和返回包都加上一个序号

就像下图这样


![image](http://cdn.anruence.com/sasasas.png)


#### 3.2.2 重点逻辑

在半双工通信模式下 
- client端：增加seq，为数据包绑定特有的id buffer
- server端：在返回的数据包里绑定id buffer

#### 3.2.3 代码

client.js

```

const net = require('net');
const socket  = new net.Socket({});
let seq =  0;
socket.connect({
    host:'127.0.0.1',
    port:6002
})

const LESSON_IDS = [
    "136797",
    "136798",
    "136799",
    "136800",
    "136801",
    "136803",
    "136804",
    "136806",
    "136807",
    "136808",
    "136809",
    "141994",
    "143517",
    "143557",
    "143564",
    "143644",
    "146470",
    "146569",
    "146582"
]


let index = Math.floor(Math.random() * LESSON_IDS.length);


socket.on('data',(buffer)=>{
    const seqBuffer = buffer.slice(0,2);
    const titleBuffer = buffer.slice(2);

    console.log(seqBuffer.readInt16BE(),titleBuffer.toString())
    // 请求回来之后再次发送
    index = Math.floor(Math.random() * LESSON_IDS.length);
    socket.write(encode(index));
})

function encode(index){
    buffer = Buffer.alloc(6);
    buffer.writeInt16BE(seq)
    buffer.writeInt32BE(
        LESSON_IDS[index],2
    )
    console.log('发包',seq,LESSON_IDS[index])
    seq++;
    return buffer;
}


setInterval(() => {
    index = Math.floor(Math.random() * LESSON_IDS.length);
    socket.write(encode(index));
}, 50);


// for(let k = 0;k< 100; k++){
//     index = Math.floor(Math.random() * LESSON_IDS.length);
//     socket.write(encode(index));
// }


```

server.js

```
const net = require('net');

net.createServer((socket)=>{
    // socket.write
    socket.on('data',function(buffer){
        // console.log('buffer',buffer)
        const seqBuffer = buffer.slice(0,2);
        const lessonId = buffer.readInt32BE(2);
        setTimeout(function(){
        let buffer = Buffer.concat([
            seqBuffer,
            Buffer.from(LESSON_DATA[lessonId])
        ])
        socket.write(buffer)
    },10+Math.random() * 1000)
    })
})
.listen(6002)


// 假数据
const LESSON_DATA = {
    136797: "01 | 课程介绍",
    136798: "02 | 内容综述",
    136799: "03 | Node.js是什么？",
    136800: "04 | Node.js可以用来做什么？",
    136801: "05 | 课程实战项目介绍",
    136803: "06 | 什么是技术预研？",
    136804: "07 | Node.js开发环境安装",
    136806: "08 | 第一个Node.js程序：石头剪刀布游戏",
    136807: "09 | 模块： CommonJS规范",
    136808: "10 | 模块：使用模块规范改造石头剪刀布游戏",
    136809: "11 | 模块：npm",
    141994: "12 | 模块：Node.js内置模块",
    143517: "13 | 异步：非阻塞I/O",
    143557: "14 | 异步：异步编程之callback",
    143564: "15 | 异步：事件循环",
    143644: "16 | 异步：异步编程之Promise",
    146470: "17 | 异步：异步编程之async/await",
    146569: "18 | HTTP：什么是HTTP服务器？",
    146582: "19 | HTTP：简单实现一个HTTP服务器"
}

```

得到结果

![image](http://cdn.anruence.com/gif5%E6%96%B0%E6%96%87%E4%BB%B6.gif)


# 总结
我们在大量前置知识的基础上，一步步推导出了全双工通道的搭建，当然了，这是不完整的，还有一些情况需要处理
回顾一下全双工通道搭建过程

- 关键在于应用层协议需要有标记包号的字段✅
- 处理以下情况，需要有标记包长的字段
    - 出现原因：TCP底层优化机制，把同时发的一些包拼起来
    - 粘包❎
    - 不完整包❎
- 错误处理
    - 网络等 


希望读完本文，你会对RPC通道有些粗浅的认识

未完待续..




