---
layout: post
title:  Android 知识点
date: '2016-08-14 17:02'
categories: Android
tags: [Android]
description:  Android 知识点
comments: true
---

[TOC]
# Android 知识点

## Android布局中同级节点的触摸事件传递顺序

触摸事件的传递顺序的逻辑应该在父控件的dispatchTouchEvent()方法中, 不同的父控件这个方法有可能不一样, 如果父控件的这个方法的实现改变了, 有可能会改变这个顺序.不过一般不会改的, 所以我们只看ViewGroup的实现, 看源码, 很容易找到把触摸事件传递给子控件的那部分代码, 基本变量命名出现child那堆就是了, 关键的逻辑应该是下面几行


```
final int childrenCount = mChildrenCount;
if (newTouchTarget == null && childrenCount != 0) {
    // 省略部分代码
    final ArrayList<View> preorderedList = buildOrderedChildList();
    final boolean customOrder = preorderedList == null && isChildrenDrawingOrderEnabled();
    final View[] children = mChildren;
    for (int i = childrenCount - 1; i >= 0; i--) {
        final int childIndex = customOrder ? getChildDrawingOrder(childrenCount, i) : i;
        final View child = (preorderedList == null) ? children[childIndex] : preorderedList.get(childIndex);
        // 省略一堆后续判断
    }
}
```

这里可以明确看到定义了一个变量final View child, 这个就是事件将要传递的子控件, 当然后面还要经过一些判断才会把事件分发给它, 不过这里不关心了.

决定这个child的实例的逻辑是

```
final View child = (preorderedList == null) ? children[childIndex] : preorderedList.get(childIndex);,
```

关键的参数有两个, preorderedList和childIndex, 到这里基本可以确定触摸事件的分发顺序就是这两个参数决定的了.

接着看preorderedList, 是调用了方法buildOrderedChildList(), 这个方法的代码就不贴出来了, 如果你没有对子控件设置elevation或者translationZ, 那么就会返回空, 如果设置了的话那么返回一个根据Z轴排序的列表, 一般情况下都是没有设置的, 如果你设置了Z轴的值, 那么在Z轴的值越大就越优先分发事件.

然后看childIndex, 一般情况下这个值就是xml文件中定义的顺序了, 不过我们可以通过方法getChildDrawingOrder()和setChildrenDrawingOrderEnabled(boolean enabled)来自定义子控件的绘制顺序, 如果你设置setChildrenDrawingOrderEnabled(true)那么isChildrenDrawingOrderEnabled()就会返回true, 导致customOrder变量在preorderedList为null的情况下是true, 接着就会调用getChildDrawingOrder()方法来获取当前事件分发的子控件的index.

总结, 你点击的区域有两个View, A和B, 它们大小相同, 位置重合

1. 如果你对A或B设置了**elevation或者translationZ**, 那么会先分发给Z轴上值较大的View, 不设置的View默认是0, 此时index只能是xml上添加的顺序

1. 如果你没有设置Z轴的值, 设置了**setChildrenDrawingOrderEnabled(true)和实现了父控件的getChildDrawingOrder()方法**, 那么顺序就是由这个方法里的实现确定了, 例如在这个方法传入参数是0的时候返回的是A的index, 传入1的时候返回的是B的index, 即使实际上A的index比B大, 那么事件也会先传递给A
 
1. 如果你什么都没干, 就是正常使用, 那么分发顺序就是子控件在xml中的顺序的倒序, 就是后添加的先分发, 实际上如果两个控件重合了, 你看到的也是后添加的控件, 那么自然点击事件也是先分发给后添加的控件了

## setWillNotDraw

ViewGroup默认情况下，出于性能考虑，会被设置成WILL_NOT_DROW，这样，ondraw就不会被执行了。

如果我们想重写一个viewgroup的ondraw方法，有两种方法：

1. 构造函数中，给viewgroup设置一个颜色。

2. 构造函数中，调用setWillNotDraw（false），去掉其WILL_NOT_DRAW flag。

在viewgroup初始化的时候，它调用了一个私有方法：initViewGroup，它里面会有一句setFlags（WILLL_NOT_DRAW,DRAW_MASK）;相当于调用了setWillNotDraw（true），所以说，对于ViewGroup，他就认为是透明的了，如果我们想要重写onDraw，就要调用setWillNotDraw（false）。


## ViewConfiguration
### getScaledPagingTouchSlop
getScaledTouchSlop是一个距离，表示滑动的时候，手的移动要大于这个距离才开始移动控件。如果小于这个距离就不触发移动控件，如viewpager就是用这个距离来判断用户是否翻页

### getScaledMaximumFlingVelocity
获取Fling速度的最大值
### getScaledMinimumFlingVelocity
获取Fling速度的最小值
### hasPermanentMenuKey
判断是否有物理按键

```
//获取Fling速度的最小值和最大值
int minimumVelocity = viewConfiguration.getScaledMinimumFlingVelocity();
int maximumVelocity = viewConfiguration.getScaledMaximumFlingVelocity();
//判断是否有物理按键boolean isHavePermanentMenuKey=viewConfiguration.hasPermanentMenuKey();
```


### dispatchApplyWindowInsets

```
// Now we'll manually dispatch the insets to our children. Since ViewPager
// children are always full-height, we do not want to use the standard
// ViewGroup dispatchApplyWindowInsets since if child 0 consumes them,
// the rest of the children will not receive any insets. To workaround this
// we manually dispatch the applied insets, not allowing children to
// consume them from each other. We do however keep track of any insets
 // which are consumed, returning the union of our children's consumption
```


## scrollTo()和scrollBy() 
参数(x, y)

任何一个控件都是可以滚动的，因为在View类当中有scrollTo()和scrollBy()这两个方法.

crollBy()方法是让View相对于当前的位置滚动某段距离，而scrollTo()方法则是让View相对于初始的位置滚动某段距离。

第一个参数x表示相对于当前位置横向移动的距离，正值向左移动，负值向右移动，单位是像素。第二个参数y表示相对于当前位置纵向移动的距离，正值向上移动，负值向下移动，单位是像素。 


## Scroller 使用

Scroller的基本用法其实还是比较简单的，主要可以分为以下几个步骤：

1. 创建Scroller的实例 
2. 要使View在规定的时间中移动到指定的位置，我们会调用startScroll()方法，startScroll()是Scroller类中的方法，
另外Scroller类中还有一个filing()方法也是很常用的，它主要是处理平滑的移动，一般营造滑动之后的惯性效果，使得View的移动更逼真。

```
/其接收四个/五个参数。如果duration不设置，则为默认。这四个参数都不难理解，这里不再做解释。
 public void startScroll(int startX, int startY, int dx, int dy, int duration) {   
    ...
 }


- startX表示当前视图的x坐标值
- startY表示当前视图的y坐标值
- dx表示在当前视图的x坐标基础上横向移动的距离
- dy表示在当前视图的y坐标基础上纵向移动的距离
- duration表示视图移动的操作在多少时间内执行完场，也就是动画的持续时间(单位：毫秒)

```
单独调用startScroll()而不重写computeScroll()方法是不会看到任何效果的。这两者 必须配合使用,才能有移动的时候的动画效果。

3. 重写computeScroll()方法，并在其内部完成平滑滚动的逻辑 


```
@Override
public void computeScroll() {
    // 第三步，重写computeScroll()方法，并在其内部完成平滑滚动的逻辑
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            invalidate();
    }
    
    // 判断Scroller是否计算完成，没有的话滚动VIew到当前计算的位置，循环调用刷新来形成连续滚动的效果。
}
```


4. **调用startScroll()方法来初始化滚动数据并刷新界面**
```
mScroller.startScroll(getScrollX(), 0, dx, 0);
invalidate();
```

为什么要调用刷新？

View 调用draw的时候会调用 computeScroll(), invalidate触发computeScroll。


## ViewGroup中的Scroller与computeScroll的关系

### Scroller到底是什么？

Scroller只是个计算器，提供插值计算，让滚动过程具有动画属性，但它并不是UI，也不是辅助UI滑动，反而是单纯地为滑动提供计算。

### Scroller只是提供计算，那谁来调用computeScroll使得ViewGroup滑动

computeScroll也不是来让ViewGroup滑动的，真正让ViewGroup滑动的是scrollTo,scrollBy。**computeScroll的作用是计算ViewGroup如何滑动。而computeScroll是通过draw来调用的。**

### computeScroll和Scroller都是计算，两者有啥关系？

computeScroll可以参考Scroller计算结果来影响scrollTo,scrollBy,从而使得滑动发生改变。也就是**Scroller不会调用computeScroll，反而是computeScroll调用Scroller。**

### 滑动时连续的，如何让Scroller的计算也是连续的？

这个就问到了什么时候调用computeScroll了，如上所说computeScroll调用Scroller，只要computeScroll调用连续，Scroller也会连续，实质上computeScroll的连续性由invalidate方法调用，scrollTo,scrollBy都会调用invalidate，而invalidate回去触发draw,从而computeScroll被连续调用，综上，Scroller也会被连续调用，除非invalidate停止调用。

### computeScroll如何和Scroller的调用过程保持一致。

computeScroll参考Scroller影响scrollTo,scrollBy，实质上，为了不重复影响scrollTo,scrollBy，那么Scroller必须终止计算currX，currY。**要知道计算有没有终止，需要通过mScroller.computeScrollOffset() **

## 