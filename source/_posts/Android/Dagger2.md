---
layout: post
title: Dagger2 使用
date: '2018-07-14 17:02'
categories: Android
tags: [Android, Dagger2]
description: Dagger2 使用
comments: true
---

# Dagger2 使用

# 包含可变参数的构造方法的情况
这里的"可变参数"指的是注入类构造方法传递的参数可能每个都不同，比如对于BeanNeedParam的注入类的构造方法：

我们引入了Dagger中的又一个新的注释标记：@Named，他的作用相当于给方法添加了一个新的属性，来区分当前注入类的不同注入方式。

更通俗的理解是，告诉Dagger，在BeanModule中存在两种方法来获得BeanNeedParam对象，其中一个叫做TypeA，另一个叫做TypeB，具体使用哪个请按照需求来选择。


# 注入类拥有多个构造方法的情况

更通俗的理解是，告诉Dagger，在BeanModule中存在两种方法来获得BeanNeedParam对象，其中一个叫做TypeA，另一个叫做TypeB，具体使用哪个请按照需求来选择。


# Singleton注释的使用

- @Singleton的单例作用，只对同一次的inject()有效。
- @Singleton在同一个Component对象内，单例是有效的。

**让@Singleton成为全局单例**

当我们需要全局的单例时，就可以在Application中创建Component对象，然后将其提供给需要的类，从而实现App范围内的单例模式。

# @Scope注释

简单来说就是没有Scope时，每次注入都创建新的对象(而**使用了Scope的注释以后，创建的对象会被复用**，从而实现单例模式)。
  
与@Singleton的作用非常相似，而实际上，**@Singleton注释就是@Scope的一个默认实现。**

# 总结注入
![image](http://img.blog.csdn.net/20170522142336895?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDk2MTYzMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

# Component的组织方法

所谓Component组织方法，也就是我们工程中的Component该如何分布和结合。

对于一款APP来说，一些基础的服务类比如全局Log、图片加载器、网络请求器、缓存器等应该做到全局单例，而对某个Activity或者Fragment来说又有自己的单例或者非单例的对象，那么这种情况下该如何组织我们的注入结构呢？

我们现在知道Component是连接注入类和目标类的桥梁，那么最简单的结构应该是这样的：

1、Application负责创建全局的单例或者非单例注入类的Component对象

2、Activity或Fragment在继承Application提供的Component基础上扩展自己的Component接口那么具体该如何操作呢？

Dagger2给我们提供两种方法来实现注入继承。

## 使用dependencies属性实现继承注入
