---
layout: post
title: Android 全屏与沉浸式
date: '2016-07-14 17:02'
categories: Android
tags: [Android]
description: Android 全屏与沉浸式
comments: true
---

# Android 全屏与沉浸式

## Android实现全屏

### 通过主题属性来实现

```
<style name="FullScreenTheme">
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowFullscreen">true</item>
</style>
```

### 使用全屏的主题

```
<activity
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen">
</activity>

或者

<activity 
    android:theme="@android:style/Theme.Material.NoActionBar.Fullscreen">
</activity>
```

### 通过代码实现

```
requestWindowFeature(Window.FEATURE_NO_TITLE);
setContentView(R.layout.activity_test);
Window window = getWindow();
window.addFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

Android 全屏后，状态栏被隐藏，通过手势下滑可以让状态栏展示，之后，状态栏会自定隐藏。


## Android实现沉浸式

### 半沉浸式

```
<style name="ImmersionTheme">
    <item name="android:windowTranslucentNavigation">true</item>
    <item name="android:windowTranslucentStatus">true</item>
    <item name="android:windowNoTitle">true</item>
</style>
```

使用：

```
<activity android:theme="@style/ImmersionTheme">
</activity>
```

### 全沉浸式6.0

```
Window window = activity.getWindow();
window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS
        | WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
        | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
window.setStatusBarColor(Color.TRANSPARENT);
window.setNavigationBarColor(Color.TRANSPARENT);

```

### App内容放置在状态栏后面

#### 4.4之上
```
getWindow().addFlags(LayoutParams.FLAG_TRANSLUCENT_STATUS);
```
LayoutParams.FLAG_TRANSLUCENT_STATUS 相当于设置了 View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
| View.SYSTEM_UI_FLAG_LAYOUT_STABLE;


