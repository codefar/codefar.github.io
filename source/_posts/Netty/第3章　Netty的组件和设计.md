---
layout: post
title:Netty学习
date: '2020-6-14 16:02'
categories: Netty
tags: [Netty]
description:Netty学习
comments: true
---

## 3.1 Channel、EventLoop

### 3.1.1　Channel接口
****
基本的I/O操作（bind()、connect()、read()和write()）依赖于底层网络传输所提供的原语。在基于Java的网络编程中，其基本的构造是class Socket。Netty的Channel接口所提供的API，大大地降低了直接使用Socket类的复杂性。此外，Channel也是拥有许多预定义的、专门化实现的广泛类层次结构的根，下面是一个简短的部分清单：

- EmbeddedChannel；
- LocalServerChannel；
- NioDatagramChannel；
- NioSctpChannel；
- NioSocketChannel。

### 3.1.2　EventLoop接口


EventLoop定义了Netty的核心抽象，用于处理连接的生命周期中发生的事件。

- 一个EventLoopGroup包含一个或者多个EventLoop；
- 一个EventLoop在它的生命周期内只和一个Thread绑定；
- 所有由EventLoop处理的I/O事件都将在它专有的Thread上被处理；
- 一个Channel在它的生命周期内只注册于一个EventLoop；
- 一个EventLoop可能会被分配给一个或多个Channel。


## 3.2　ChannelHandler和ChannelPipeline

### 3.2.1  ChannelHandler接口

### 3.2.2  ChannelPipeliner接口

### 3.2.3  更加深入地理解ChannelHandler

### 3.2.4  编码器和解码器

### 3.2.5　抽象类SimpleChannelInboundHandler
SimpleChannelInboundHandler 自动释放内存。

### 3.3 引导

# 第4章　传输

本章主要内容

- OIO——阻塞传输
- NIO——异步传输
- Local——JVM内部的异步通信
- Embedded——测试你的ChannelHandler

流经网络的数据总是具有相同的类型：字节。这些字节是如何流动的主要取决于我们所说的网络传输——一个帮助我们抽象底层数据传输机制的概念。用户并不关心这些细节；他们只想确保他们的字节被可靠地发送和接收”


每个Channel都将会被分配一个ChannelPipeline和ChannelConfig。ChannelConfig包含了该Channel的所有配置设置，并且支持热更新.由于特定的传输可能具有独特的设置，所以它可能会实现一个ChannelConfig的子类型。

第5章 ByteBuf

网络数据的基本单位总是字节。Java NIO提供了ByteBuffer作为它的字节容器，但是这个类使用起来过于复杂，而且也有些繁琐。

Netty的ByteBuffer替代品是ByteBuf，一个强大的实现，既解决了JDK API的局限性，又为网络应用程序的开发者提供了更好的API。

5.1 ByteBuf的API

Netty的数据处理API通过两个组件暴露——abstract class ByteBuf和interface ByteBufHolder。

- 它可以被用户自定义的缓冲区类型扩展；
-  通过内置的复合缓冲区类型实现了透明的零拷贝；
-  容量可以按需增长（类似于JDK的StringBuilder）；
-  在读和写这两种模式之间切换不需要调用ByteBuffer的flip()方法；
- 读和写使用了不同的索引；
- 支持方法的链式调用；
- 支持引用计数；
- 支持池化。

5.2 ByteBuf类——Netty的数据容器

5.2.1 它是如何工作的

5.2.2 ByteBuf的使用模式


5.3 字节级操作

5.3.1 随机访问索引

5.3.2 顺序访问索引

5.3.3 可丢弃字节

5.3.4 可读字节

5.3.5 可写字节

5.3.6 索引管理

5.3.7 查找操作

5.3.8 派生缓冲区

5.3.9 读/写操作

5.3.10 更多的操作



5.4 ByteBufHolder接口

5.5 ByteBuf分配 5.5.1 按需分配：ByteBufAllocator接口

5.5.2 Unpooled缓冲区

5.5.3 ByteBufUtil类



5.6 引用计数

5.7 小结

