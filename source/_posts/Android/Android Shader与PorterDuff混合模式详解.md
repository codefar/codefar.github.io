---
layout: post
title: Android PorterDuff和Shader混合模式详解
date: '2016-06-14 14:01'
categories: Android
comments: true
tags: [Android]
description: Android PorterDuff和Shader混合模式详解
---

# Android PorterDuff和Shader混合模式详解


## 什么是PorterDuffXfermode、PorterDuff？

首先，PorterDuff是Tomas Porter和Tom Duff两个人的名字的拼写。PorterDuff的命名对Tomas Porter和Tom Duff两个人在“Compositing digital images（合成数字图像）”方面的工作的致敬。

Android中的PorterDuff.Mode描述了12种图片合成时的模式。这些模式控制了将原（将要渲染的图像对象）和目标的（已经渲染的对象的内容）合成后结果的值。

首先要搞清楚这两个概念。

-  **原图片**：指的是我们将要后绘制的东西。
-  **目标图片**：指的是我们已经渲染上的东西

Android 官网描述内容：
https://developer.android.com/reference/android/graphics/PorterDuff.Mode.html


## 什么是Shader？

Shader是OpenGL中的着色器的概念。
