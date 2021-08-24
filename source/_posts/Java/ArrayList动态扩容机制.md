---
layout: post
title:ArrayList动态扩容机制
date: '2019-02-14 16:02'
categories: Java
tags: [Java]
description: ArrayList动态扩容机制
comments: true
---

# ArrayList动态扩容机制

ArrayList动态扩容机制

```
首先，来看下ArrayList中关于容量的代码
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;

/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

```
添加元素时，进行扩容

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

```

ensureCapacityInternal用来确认扩容。入参是：需要的最小的容量。

```
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```

可以看到，先确定了最小容量为DEFAULT_CAPACITY和所需最小容量的最大值，即10.或者minCapacity
```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

```
如果当前容量不够存放元素，就进行真正的扩容
```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

grow的扩容算法。
```
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1);
首先，将原来容量（现在数组的长度）扩大为1.5倍。

 if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
如果扩大1.5倍后还不够存放新元素。则将新容量设置为需要的容量。
（如果newCapacity溢出， if (newCapacity - minCapacity < 0) 为false 不会走的）

 if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
如果新容量超过了MAX_ARRAY_SIZE（Integer.MAX_VALUE - 8）。
调整新容量，则将新容量调整为Integer.MAX_VALUE
（如果newCapacity溢出，
if (newCapacity - MAX_ARRAY_SIZE > 0) 为true, 溢出变成正值了）

elementData = Arrays.copyOf(elementData, newCapacity);
将旧元素复制到新的数组中。
```
番外：
 可以看到JDK中使用 if (newCapacity - minCapacity < 0)这样的代码，判断数字大小。
那为什么不用 if (newCapacity < minCapacity) 呢?
```
int a = Integer.MAX_VALUE;
int b = Integer.MIN_VALUE;
if (a < b) {
    System.out.println("a < b");
}
if (a - b < 0) {
    System.out.println("a - b < 0");
}

//print a - b < 0
```
Now, having said that, consider that the array has a length that is really close to Integer.MAX_VALUE. The code in ArrayList goes like this:

```
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1);
if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
```

原因是：
oldCapacity 可能很接近Integer.MAX_VALUE 。所以 newCapacity (oldCapacity + 0.5 * oldCapacity) 可能溢出并且变成Integer.MIN_VALUE (i.e. negative). 所以减去minCapacity 向下溢出变成正数

https://stackoverflow.com/questions/33147339/difference-between-if-a-b-0-and-if-a-b