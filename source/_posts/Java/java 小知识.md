---
layout: post
title:简单比较 getName()、getCanonicalName()、getSimpleName() 的异同
date: '2019-06-14 16:02'
categories: Java
tags: [Java]
description: 简单比较 getName()、getCanonicalName()、getSimpleName() 的异同
comments: true
---

## 简单比较 getName()、getCanonicalName()、getSimpleName() 的异同


```
package testName;

class Name{
	class Inner{}
}
 
public class testGetName {
 
	public static void main(String[] args) throws Exception {
		System.out.println("Name.class.getCanonicalName(): " + Name.class.getCanonicalName());
        System.out.println("Name.class.getName():          " + Name.class.getName());
        System.out.println("Name.class.getSimpleName():    " + Name.class.getSimpleName());
 
        System.out.println("Name.Inner.class.getCanonicalName(): " + Name.Inner.class.getCanonicalName());
        System.out.println("Name.Inner.class.getName():          " + Name.Inner.class.getName());
        System.out.println("Name.Inner.class.getSimpleName():    " + Name.Inner.class.getSimpleName());
 
        System.out.println("args.getClass().getCanonicalName(): " + args.getClass().getCanonicalName());
        System.out.println("args.getClass().getName():          " + args.getClass().getName());
        System.out.println("args.getClass().getSimpleName():    " + args.getClass().getSimpleName());
 
	}
 

```


输出结果如下：

```
Name.class.getCanonicalName(): testName.Name
Name.class.getName():          testName.Name
Name.class.getSimpleName():    Name


Name.Inner.class.getCanonicalName(): testName.Name.Inner
Name.Inner.class.getName():          testName.Name$Inner
Name.Inner.class.getSimpleName():    Inner


args.getClass().getCanonicalName(): java.lang.String[]
args.getClass().getName():          [Ljava.lang.String;
args.getClass().getSimpleName():    String[]


```


``
1、 ROUND_UP：远离零方向舍入。向绝对值最大的方向舍入，只要舍弃位非0即进位。

2、 ROUND_DOWN：趋向零方向舍入。向绝对值最小的方向输入，所有的位都要舍弃，不存在进位情况。

3、 ROUND_CEILING：向正无穷方向舍入。向正最大方向靠拢。若是正数，舍入行为类似于ROUND_UP，若为负数，舍入行为类似于ROUND_DOWN。Math.round()方法就是使用的此模式。

4、 ROUND_FLOOR：向负无穷方向舍入。向负无穷方向靠拢。若是正数，舍入行为类似于ROUND_DOWN；若为负数，舍入行为类似于ROUND_UP。

5、 HALF_UP：最近数字舍入(5进)。这是我们最经典的四舍五入。

6、 HALF_DOWN：最近数字舍入(5舍)。在这里5是要舍弃的。

7、 HAIL_EVEN：银行家舍入法。

``