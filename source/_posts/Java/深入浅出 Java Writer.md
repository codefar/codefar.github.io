---
layout: post
title: 深入浅出 Java Writer
date: '2019-10-14 16:02'
categories: Java
tags: [Java]
description: 深入浅出 Java Writer
comments: true
---

深入浅出 Java Writer

# 简介

Java Writer属于JavaPoet另一个分支，也是生成Java 源文件的工具。Java Writer相比JavaPoet，更加简单，简单到整个库只有一个文件。

Java Writer官方地址：https://github.com/square/javapoet/tree/javawriter_2

> JavaWriter is a utility class which aids in generating Java source files.
> 

# 使用

## 引入

Gradle
```
compile 'com.squareup:javawriter:2.5.1'
```
Maven
```
<dependency>
  <groupId>com.squareup</groupId>
  <artifactId>javawriter</artifactId>
  <version>2.5.1</version>
</dependency>
```

## 实例

### 

我们要生成的源码文件如下：
```
package com.example;

public final class Person {
  private String firstName;
  private String lastName;
  /**
   * Returns the person's full name.
   */
  public String getName() {
    return firstName + " " + lastName;
  }
}
```

```
File outFile;/// .java file
OutputStreamWriter writer = new OutputStreamWriter(new FileOutputStream(outFile));
JavaWriter writer = new JavaWriter(writer);
writer.emitPackage("com.example")
    .beginType("com.example.Person", "class", EnumSet.of(PUBLIC, FINAL))
    .emitField("String", "firstName", EnumSet.of(PRIVATE))
    .emitField("String", "lastName", EnumSet.of(PRIVATE))
    .emitJavadoc("Returns the person's full name.")
    .beginMethod("String", "getName", EnumSet.of(PUBLIC))
    .emitStatement("return firstName + \" \" + lastName")
    .endMethod()
    .endType();
    .close();
```


参考：
- https://juejin.im/entry/58fefebf8d6d810058a610de
- https://github.com/square/javapoet/tree/javawriter_2
