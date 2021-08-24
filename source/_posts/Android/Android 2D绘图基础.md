---
layout: post
title: Android 2D绘图基础
date: '2016-09-13 17:02'
categories: Android
tags: [Android]
description: Android 2D绘图基础
comments: true
---

## 一.Canvas简介
Canvas我们可以称之为画布，能够在上面绘制各种东西，是安卓平台2D图形绘制的基础，非常强大。

## 二.Canvas的常用操作

![image](https://note.youdao.com/yws/api/personal/file/WEBbe27657053189b5a6d70041bfcc54f8b?method=getImage&version=1311&cstk=aNW9Q6WS)



操作类型  | 相关API | 备注 | 
---|---|---
绘制颜色 | drawColor, drawRGB, drawARGB	 | 使用单一颜色填充整个画布
绘制基本形状| 	drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc	| 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片 | 	drawBitmap, drawPicture	| 绘制位图和图片
绘制文本 | 	drawText, drawPosText, drawTextOnPath | 	依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径| 	drawPath| 	绘制路径，绘制贝塞尔曲线时也需要用到该函数
顶点操作| 	drawVertices, drawBitmapMesh| 	通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁| 	clipPath, clipRect| 	设置画布的显示区域
画布快照| 	save, restore, saveLayerXxx, restoreToCount, getSaveCount| 	依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数
画布变换| 	translate, scale, rotate, skew| 	依次为 位移、缩放、 旋转、错切
Matrix(矩阵)| 	getMatrix, setMatrix, concat	|  实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

### Canvas的类图
![类图](https://github.com/davyjoneswang/static-res/blob/master/canvas-diagram.png?raw=true)
http://git.sankuai.com/users/wangyonghua/repos/erp-pos-cahsier-surfaceview/browse/1%E6%9C%88-05-2018%2015-23-52.gif?at=b297faa53f72b0b7b9632b9635f2d03d5049cba8&raw

## 三.Canvas详解

### 1. 绘制颜色

1.   drawColor, drawRGB, drawARGB 绘制颜色

```
canvas.drawColor(Color.RED); //绘制红色
```

2. drawColor(int color, PorterDuff.Mode mode) 带混合模式的绘制颜色, 后面讲混合模式

```
canvas.drawColor(Color.BLUE, PorterDuff.Mode.SRC);
```

3. drawARGB 绘制颜色



3.  drawPoint 绘制点

``` 
//在坐标（100, 100处画一个点、  需要设置画笔的属性，否则看不到点的大小
canvas.drawPoint(100, 100, mPaint);
```

4.   drawPoints 绘制多个点色
    
```
//画多个点
canvas.drawPoints(new float[]{400,400,400,500,400,600,400,700,400,800,400,900},mPaint);
```

5.   drawLine 绘制直线


```
//在坐标（100,100）和（600,100）之间画一条线
canvas.drawLine(100,100,600,100,mPaint);
```

6.   drawLines 绘制多条直线

```
 //画多条线
 canvas.drawLines(new float[]{100,150,600,150,100,200,600,200,100,250,600,250},mPaint);
```

7.  drawPicture 绘制picture 
这里的Picture，个人理解：Picture记录了画笔的操作，可以在将来再次使用。


8. drawRect 绘制矩形

```
// 第一种
canvas.drawRect(100,100,800,400,mPaint);
```

9. drawRoundRect 绘制圆角矩形

```
RectF rectF = new RectF(100,100,600,300);
canvas.drawRoundRect(rectF,20,20,mPaint);
```

10. drawOval 绘制椭圆

```
//绘制椭圆
RectF rectF = new RectF(100,100,600,400);
canvas.drawOval(rectF,mPaint);
```

11. drawCircle 绘制圆

```
// 绘制一个圆心坐标在(360,400)，半径为300 的圆。
canvas.drawCircle(360,400,300,mPaint);
```

11.  drawArc 画弧

```
public void drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint){}
```

```
//弧在的矩形
oval 
//开始角度
startAngle 
//扫过角度
sweepAngle  
//是否使用中心
useCenter
```
 **角度是按照按顺时针方向扫描的**

**useCenter true的话，会将矩形中心连接，否则不连接**
 
**Paint的setStyle印象弧是否填充还是描边。**

12.   drawBitmap 绘制位图，


```
drawBitmap
```


13.   drawBitmapMesh 绘制位图网格
14.   drawPatch 绘制N-patch图 (most often, a 9-patches.)
15.   drawPath 绘制路径
16.   drawPosText 
17.   drawText
18.   drawTextOnPath
19.   drawTextRun
20.   drawVertices
21.   

## Paint 详解

### 画笔样式
设置画笔样式
mPaint.setStyle(Paint.Style.FILL);  


Paint.Style	 | 描述
---|---
STROKE	|  描边
FILL	|  填充
FILL_AND_STROKE	| 描边加填充



### 线宽

mPaint.setStrokeWidth（）


### 颜色

mPaint.setColor（） 


### 