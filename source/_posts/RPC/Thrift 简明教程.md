[TOC]

# Thrift 简明教程

# 语法教程

Thrift 采用IDL（Interface Definition Language）来定义通用的服务接口，然后通过Thrift提供的编译器，可以将服务接口编译成不同语言编写的代码，通过这个方式来实现跨语言的功能。

## 文档结构
每个Thrift文档包含0或更多个Header定义， 然后，后面跟着0或多个类型定义。

### Header定义

每个头部是Thrift Include、C++ Include 或者 Namespace。

Thrift Include  
include指令使得来自另一个thrift文件的所有符号都可见（带有前缀），并将相应的include语句添加到为此Thrift文档生成的代码中。

C++ Include  
C ++包含为此Thrift文档的C++代码生成器的输出添加自定义C++包含。

Namespace  
命名空间声明了哪些命名空间/包/模块/等。将为目标语言声明此文件中的类型定义。
命名空间范围指示命名空间适用的语言; 
命名空间范围包括： 'cpp' | 'java' | 'py' | 'perl' | 'rb' | 'cocoa' | 'csharp'
范围“*”表示命名空间适用于所有目标语言。
 
 
#### include 指令

用 include 指令包含其他 thrift 文件：

```
include "another.thrift"
```

#### namespace 指令
namespace 指令指定语言相关的名称空间：

```
namespace cpp nscpp # 指定C++的名称空间为 nscpp
namespace java nsjava # 指定Java的包为 nsjava
```

### 类型定义
类型定义包括：

```
Const | Typedef | Enum | Senum | Struct | Union | Exception | Service
```

## 常量
常量用const表示：

```
const i32 MATHPATH = 256
```

## typedef 指令
typedef 指令用来指定类型别名，和 C语言一样。

```
typedef i32 MyInteger
```

## 枚举
用 enum 定义枚举。枚举类型是32位的整数。值是可选的，从1开始。

```
enum Operation {
    ADD = 1,
    SUBTRACT = 2,
    MULTIPLY = 3,
    DIVIDE = 4
}
```

## Senum（Slist） 
过时，不使用了，由string替代。

## Struct 结构体
结构体用 struct 表示，是基本的复合数据类型，由若干字段组成。每个字段由整型ID，类型，名称和可选的默认值组成。字段可以声明为 optional，表示如果没有被设置，则不进行序列化。

```
struct Work {
    1: i32 num1 = 0,
    2: i32 num2,
    3: Operation op,
    4: optional string comment,
}
```

## Union 联合体
Union 和 struct 类似，除了它提供了一种方法来传输一组可能的字段中的一个字段。就像C ++中的union {}一样。因此，Union 成员被隐含地视为optional可选的）。

## Exception 异常

异常类似于结构，除了它们旨在与目标语言中的异常处理机制集成。
在异常中，每个字段的名称必须是唯一的。

```
exception InvalidOperation {
    1: i32 whatOp,
    2: string why
}
```

## Service 服务

服务用 service 关键字定义。服务可以用 extends 关键字继承其他服务。service 由一系列方法组成。方法由返回值，参数列表和一个可选的异常列表组成。参数列表和异常列表的语法和结构体的语法一样。

```
service Calculator extends shared.SharedService {

   void ping(),

   i32 add(1:i32 num1, 2:i32 num2),

   i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),

   /**
    * oneway 表示客户端发送请求后不需要响应。onway 方法返回值必须是void 
    */
   oneway void zip()

}
```

oneway 方法必须返回void, 不能抛出异常。

## 方法与类型

方法由 可选的"oneway"标识 + 方法返回值类型 + 标识名 + 参数列表 + 异常列表组成。

方法类型包括 void 或 数值类型。

异常抛出：由 throws关键字 + 异常列表 组成。

### 常量类型
整形常量、Double常量、列表常量[]、 map常量 {}。

### 基本类型

<html>
<!--在这里插入内容-->
<table>
<thead>
<tr>
  <th>类型</th>
  <th>描述</th>
</tr>
</thead>
<tbody><tr>
  <td><code>bool</code></td>
  <td>布尔类型，1byte</td>
</tr>
<tr>
  <td><code>i8</code></td>
  <td>有符号整形，8bits，即byte</td>
</tr>
<tr>
  <td><code>i16</code></td>
  <td>有符号整型，16bits</td>
</tr>
<tr>
  <td><code>i32</code></td>
  <td>有符号整型，32bits</td>
</tr>
<tr>
  <td><code>i64</code></td>
  <td>有符号整型，64bits</td>
</tr>
<tr>
  <td><code>double</code></td>
  <td>浮点型，64bits</td>
</tr>
<tr>
  <td><code>string</code></td>
  <td>字符串</td>
</tr>
<tr>
  <td><code>binary</code></td>
  <td>Blob，byte数组</td>
</tr>
<tr>
  <td><code>map&lt;t1,t2&gt;</code></td>
  <td>map</td>
</tr>
<tr>
  <td><code>list&lt;t1&gt;</code></td>
  <td>有序列表</td>
</tr>
<tr>
  <td><code>set&lt;t1&gt;</code></td>
  <td>无重复元素的容器</td>
</tr>
</tbody></table>
</html>

## 容器类型
集合中的元素可以是除了service之外的任何类型，包括exception。

1. list<T>: 一系列由T类型的数据组成的有序列表，元素可以重复
1. set<T>: 一系列由T类型的数据组成的无序集合，元素不可重复
1. map<K, V>: 一个字典结构，key为K类型，value为V类型，相当于Java中的HMap<K,V>
2. 
### 可选与必选
thrift提供两个关键字required，optional，分别用于表示对应的字段时必填的还是可选的。例如：

```
struct People {
    1: required string name;
    2: optional i32 age;
}
```


## 注释
```
# 脚本注释
 
/*
* 多行注释
*/

// 单行注释
```


# 编译Thrift

```
thrift-r --gen cpp ICalc.thrift
```

# 实战教程




