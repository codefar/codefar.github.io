---
layout: post
title: Android selector StateListDrawable的匹配原理
date: '2016-04-14 17:02'
categories: Android
tags: [Android]
description: Android selector StateListDrawable的匹配原理
comments: true
---


# Android selector StateListDrawable的匹配原理


## 

常见的selector  xml

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/button_bg_pressed" android:state_pressed="true"></item>
    <item android:drawable="@drawable/button_bg_normal"></item>
</selector>
```

而对应的Java代码实现为StateListDrawable


```
StateListDrawable drawable = new StateListDrawable();
drawable.addState(new int[]{android.R.attr.state_pressed}, mPressedDrawable);
drawable.addState(new int[0], mDefaultDrawable);
```

StateListDrawable的父类为DrawableContainer。就是可以包含很多Drawable的容器。

StateListDrawable是有很多的状态的，每种状态又对应不同的Drawable，那么怎么取出当前的状态呢。


```
 backgroundDrawable.setState(new int[]{android.R.attr.state_enabled, android.R.attr.state_pressed});
 backgroundDrawable.getCurrent()
```

这样我们就取出来了 enable 状态下按下时的Drawable。