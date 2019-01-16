---
title: Node.js系列(一)之安装
date: 2017/04/13
tags: [node]
toc: false
---

## node简介
node是javascript语言的服务器运行环境
所谓的运行环境有两层意思：
1. javascript语言通过node在服务器运行，在这个意义上，node是**javascriprt的虚拟机**
2.node提供大量的工具库，使得javascript语言与操作系统互动（比如读写文件，新建子进程），在这个意义上，node又是**javascrip的工具库**
Node内部采用[Google公司的V8引擎](http://baike.baidu.com/link?url=1kFBYYp0gB7_P6YD0d1s3sqF64zK41APPhCywsEG9qTKgguFTwwYPZTohqZzd82j)，作为JavaScript语言解释器；通过自行开发的libuv库，调用操作系统资源。
## 什么是Google V8 JavaScript引擎
V8是一个由丹麦Google开发的开源JavaScript引擎，V8就是chrome浏览器用的js解释引擎，主要是C编写的
V8在执行之前将JavaScript编译成了机器码，而非位元组码或是直译它，以此提升效能。更进一步，有了这些功能，JavaScript程序与V8引擎的速度媲美二进制编译。[4]
## 安装相关
访问官方网站nodejs.org
安装完成查看node版本

```
$ node --version
 或者
$ node -v
```
![wunai](http://oucjferwh.bkt.clouddn.com/node1-1.png)

更新node版本，可以通过node.js的n模块完成，
更新为最新发布的稳定版。

```
$ sudo npm install n -g
$ sudo n stable
```
![node-1-2](http://oucjferwh.bkt.clouddn.com/node1-2.png)

```
$ sudo n 0.10.21
```
### 安装版本管理工具nvm
如果想在同一台机器同时安装多个版本的node，就需要用到嗯本管理工具nvm，nvm全称**Node Version Manager**，它与n的实现方式不同，其是通过shell脚本实现的。

```
$ git clone https://github.com/creationix/nvm.git ~/.nvm
$ source ~/.nvm/nvm.sh
```
![node-1-3](http://oucjferwh.bkt.clouddn.com/node1-3.png)
###### 安装最新版本

```
$ nvm install node
```
![node-1-4](http://oucjferwh.bkt.clouddn.com/node1-4.png)

###### 安装指定版本

```
$ nvm install 0.12.1
```

######使用已安装的最新版本

```
$ nvm use node
```

###### 使用指定版本的node

```
$ nvm use 0.12
```
###### 查看本地安装的所有版本

```
$ nvm ls
```
![node-1-5](http://oucjferwh.bkt.clouddn.com/node1-5.png)
###### 退出已经激活的nvm，使用deactivate命令。

```
$ nvm deactivate
```
######卸载nvm

```
rm -rf ~/.nvm
```
######查看nvm帮助

```
nvm -h
```
详细文档请参考官方文档
https://github.com/creationix/nvm
