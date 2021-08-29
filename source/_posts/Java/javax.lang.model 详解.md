---
layout: post
title: javax.lang.model 详解
date: '2019-07-14 16:02'
categories: Java
tags: [Java]
description: javax.lang.model 详解
comments: true
---

# javax.lang.model 详解

## 软件包 javax.lang.model
用来为 Java 编程语言建立模型的包的类和层次结构。
枚举 **SourceVersion**

## 软件包 javax.lang.model.type
用来为 Java 编程语言类型建立模型的接口。

枚举 **TypeKind**

枚举常量摘要
type | 说明
---|---
ARRAY |	数组类型。
BOOLEAN |	基本类型 boolean。
BYTE  |	基本类型 byte。
CHAR  |	基本类型 char。
DECLARED |	类或接口类型。
DOUBLE  |	基本类型 double。
ERROR  |	 无法解析的类或接口类型。
EXECUTABLE |	方法、构造方法或初始化程序。
FLOAT  |	 基本类型 float。
INT  |	 基本类型 int。
LONG  |	 基本类型 long。
NONE  |	 在实际类型不适合的地方使用的伪类型。
NULL  |	ull 类型。
OTHER  | 为实现保留的类型。
PACKAGE |	对应于包元素的伪类型。
SHORT  |	基本类型 short。
TYPEVAR |	类型变量。
VOID  |	对应于关键字 void 的伪类型。
WILDCARD  |	通配符类型参数。
          
接口摘要

type | 说明
---|---
ArrayType |	表示一个数组类型。
DeclaredType |	表示某一声明类型，是一个类 (class) 类型或接口 (interface) 类型。
ErrorType |	表示无法正常建模的类或接口类型。
ExecutableType |	表示 executable 的类型。
NoType	| 在实际类型不适合的地方使用的伪类型。
NullTyp e|	表示 null 类型。
PrimitiveType |	表示一个基本类型。
ReferenceType	|表示一个引用类型。
TypeMirror |	表示 Java 编程语言中的类型。
TypeVariable |	表示一个类型变量。
TypeVisitor<R,P>|	类型的 visitor，使用 visitor 设计模式的样式。
WildcardType |	表示通配符类型参数。

