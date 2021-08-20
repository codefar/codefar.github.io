---
layout: post
title: Andrid HotFix 总结
date: '2016-07-14 17:02'
---

安卓自定义 View 基础
Android View坐标系
1. 屏幕坐标系以屏幕左上角为坐标原点，向右为x轴增大方向，向下为y轴增大方向
2. View的坐标系 注意：View的坐标系统是相对于父控件而言的.
    getTop();       //获取子View左上角距父View顶部的距离
    getLeft();      //获取子View左上角距父View左侧的距离
    getBottom();    //获取子View右下角距父View顶部的距离
    getRight();     //获取子View右下角距父View左侧的距离
3. MotionEvent中 get 和 getRaw 的区别
    //触摸点相对于其所在组件坐标系的坐标
    event.getX();
    event.getY();
    //触摸点相对于屏幕默认坐标系的坐标
    event.getRawX();
    event.getRawY();
角度弧度
    角度和弧度一样都是描述角的一种度量单位。
    角度	两条射线从圆心向圆周射出，形成一个夹角和夹角正对的一段弧。当这段弧长正好等于圆周长的360分之一时，两条射线的夹角的大小为1度.
    弧度	两条射线从圆心向圆周射出，形成一个夹角和夹角正对的一段弧。当这段弧长正好等于圆的半径时，两条射线的夹角大小为1弧度.

    先设圆的周长为C. 半径为r
    C = 2πr;
    一周对应的角度为360度(角度)，对应的弧度为2π弧度。
    故: 180度 = π弧度. 弧度 = 角度xπ/180 角度 = 弧度x180/π
    在常见的数学坐标系中角度增大方向为逆时针，
    在默认的屏幕坐标系中角度增大方向为顺时针。
绘制基本形状
   简要介绍Paint
   <pre>
   //绘制颜色是填充整个画布，常用于绘制底色
canvas.drawColor(Color.BLUE); //绘制蓝色
//可以绘制一个点，也可以绘制一组点，如下：
canvas.drawPoint(200, 200, mPaint);     //在坐标(200,200)位置绘制一个点
canvas.drawPoints(new float[]{          //绘制一组点，坐标位置由float数组指定
        500, 500,
        500, 600,
        500, 700
}, mPaint);
//绘制直线：
canvas.drawLine(300, 300, 500, 600, mPaint);    // 在坐标(300,300)(500,600)之间绘制一条直线
canvas.drawLines(new float[]{               // 绘制一组线 每四数字(两个点的坐标)确定一条线
        100, 200, 200, 200,
        100, 300, 200, 300
}, mPaint);
//绘制矩形：
// 第一种
canvas.drawRect(100, 100, 800, 200, mPaint);

// 第二种
//Rect rect = new Rect(100, 100, 800, 400);
//canvas.drawRect(rect, mPaint);

// 第三种
//RectF rectF = new RectF(100, 100, 800, 400);
//canvas.drawRect(rectF, mPaint);
// 绘制圆角矩形：两个参数rx 和 ry，这里圆角矩形的角实际上不是一个正圆的圆弧，
// 而是椭圆的圆弧，这里的两个参数实际上是椭圆的两个半径
mPaint.setColor(Color.CYAN);
//RectF rectF = new RectF(100,310,800,410);
//canvas.drawRoundRect(rectF,30,30,mPaint);

//绘制椭圆：
// 第一种
//RectF rectF1 = new RectF(100,510,800,610);
//canvas.drawOval(rectF1, mPaint);

//mPaint.setStyle(Paint.Style.STROKE);
//canvas.drawCircle(500,500,400, mPaint);

/** 绘制圆弧 通过字面意思我们基本能猜测出来前两个参数(startAngle，
 * sweepAngel)的作用，就是确定角度的起始位置和扫过角度， 不过第三个参数是干嘛的？试一下就知道了,上代码：
 * startAngle  // 开始角度
 sweepAngle  // 扫过角度
 useCenter   // 是否使用中心
 */

RectF rectF = new RectF(100, 100, 800, 400);
// 绘制背景矩形
mPaint.setColor(Color.GRAY);
canvas.drawRect(rectF, mPaint);

// 绘制圆弧
mPaint.setColor(Color.BLUE);
canvas.drawArc(rectF, 0, 90, false, mPaint);

//-------------------------------------

RectF rectF2 = new RectF(100, 600, 800, 900);
// 绘制背景矩形
mPaint.setColor(Color.GRAY);
canvas.drawRect(rectF2, mPaint);

/**
 * 可以发现使用了中心点之后绘制出来类似于一个扇形，而不使用中心点则是圆弧起始点和结束点之间的连线加上圆弧围成的图形。
 * 这样中心点这个参数的作用就很明显了，不必多说想必大家试一下就明白了
 */
// 绘制圆弧
mPaint.setColor(Color.BLUE);
canvas.drawArc(rectF2, 0, 90, true, mPaint);
   </pre>
