---
layout: post
title:深入浅出 JavaPoet
date: '2019-11-14 16:02'
categories: Java
tags: [Java]
description:深入浅出 JavaPoet
comments: true
---

深入浅出 JavaPoet

# 简介

JavaPoet 是生成Java 源文件的工具。

JavaPoet官方地址：https://github.com/square/javapoet

## API 速揽

类图：
![image](https://note.youdao.com/yws/api/personal/sync?method=download&fileId=WEBf1df54081b60e10574b2792e45c92d68&version=2243&cstk=cTSa6w9t&keyfrom=web)

在JavaPoet中，我们通过各种Spec类最终生成我们的JavaFile. Spec类型包括如下。

类 | 描述 | 备注
---|---|---
JavaFile | A Java file containing a single top level class.  | JavaFile代表一个最终生成的Java源码文件, 源码文件里包含Class,接口，枚举.
TypeSpec | A generated class, interface, or enum declaration. | 用来生成Class, 接口，枚举 对象
MethodSpec | A generated constructor or method declaration | 用来生成方法
FieldSpec | A generated field declaration. | 用来生成域（成员）声明
ParameterSpec |A generated parameter declaration. | 用来生成参数
AnnotationSpec | A generated annotation on a declaration. | 用来生成注解

JavaPoet中数据类型

分类 | 	生成的类型 | 	JavaPoet 写法 | 	等效的 Java 写法）
---|---|---|---
内置类型 | 	int	 | TypeName.INT | 	int.class
数组类型 | 	int[] | 	ArrayTypeName.of(int.class)	 | int[].class
需要引入包名的类型 | 	java.io.File  | 	ClassName.get(“java.io”, “File”) | 	java.io.File.class
参数化类型 （ParameterizedType） | 	List  | 	ParameterizedTypeName.get(List.class, String.class) | 	–
类型变量（WildcardType） | 用于声明泛型	T	 |  TypeVariableName.get(“T”) | 	–
通配符类型	 | ? extends String	 | WildcardTypeName.subtypeOf(String.class) | 	–

- List.class 等价于 ClassName.get("java.util", "List")。
- ParameterizedTypeName.get(List.class, String.class) 
等价于
ParameterizedTypeName.get(ClassName.get("java.util", "List"), ClassName.get("java.lang", "String"))。
 
JavaPoet其它类

类 | 描述 | 备注
---|---|---
LineWrapper | Implements soft line wrapping on an appendable. | 用来生成行
CodeBlock | A fragment of a .java file, | 源码中的代码块
TypeName | Any type in Java's type system| Java中的类型名

# 使用

## 引入

Gradle
```
compile 'com.squareup:javapoet:1.10.0'
```
Maven
```
<dependency>
  <groupId>com.squareup</groupId>
  <artifactId>javapoet</artifactId>
  <version>1.10.0</version>
</dependency>
```

## 实例

### HelloWorld

我们要生成的源码文件如下：
```
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```

```

import com.squareup.javapoet.JavaFile;
import com.squareup.javapoet.MethodSpec;
import com.squareup.javapoet.TypeSpec;
import javax.lang.model.element.Modifier;

MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

### 类型占位符
在 Statement 中使用类型占位符.

```
MethodSpec.methodBuilder("method")
    .addStatement("$T file", File.class)            // File file;
    .addStatement("$L = null", "file")              // file = null;
    .addStatement("file = new File($S)", "foo/bar") // file = new File("foo/bar");
    .build();
```

* $T 代表Types就是类型替换, 用参数的类型替换$T 。
    
    一般用于 ("$T foo", List.class) => List foo. 使用$T ，JavaPoet 会自动帮你补全文件开头的 import. 
    直接写 ("List foo") 虽然也能生成 List foo， JavaPoet不会自动帮你添加 import java.util.List.

* $L 代表 Literals，就是用参数的字面值替换$L。
    
    示例：("abc$L123", "FOO") => abcFOO123. 也就是直接替换.
* $S 是字符串替换, 用字符串参数替换$S。

    示例: ("$S.length()", "foo") => "foo".length() 注意 $S 是将参数替换为了一个带双引号的字符串。 **需要注意的是：与$L 相比，$S是带双引号的。**
* $N 代表 Names。自引用，也就是引用我们生成的代码。  
    
    比如你之前定义了一个函数 MethodSpec methodSpec = MethodSpec.methodBuilder("foo").build(); 现在你可以通过 $N 获取这个函数的名称 ("$N", methodSpec) => foo.

```
public String byteToHex(int b) {
  char[] result = new char[2];
  result[0] = hexDigit((b >>> 4) & 0xf);
  result[1] = hexDigit(b & 0xf);
  return new String(result);
}

public char hexDigit(int i) {
  return (char) (i < 10 ? i + '0' : i - 10 + 'a');
}
```


```
MethodSpec hexDigit = MethodSpec.methodBuilder("hexDigit")
    .addParameter(int.class, "i")
    .returns(char.class)
    .addStatement("return (char) (i < 10 ? i + '0' : i - 10 + 'a')")
    .build();

MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();
```

### 生成代码，流程控制

* JavaPoet生成代码

```
void main() {
  int total = 0;
  for (int i = 0; i < 10; i++) {
    total += i;
  }
}
```
可以使用MethodSpec的addCode来添加代码。

```
MethodSpec main = MethodSpec.methodBuilder("main")
    .addCode(""
        + "int total = 0;\n"
        + "for (int i = 0; i < 10; i++) {\n"
        + "  total += i;\n"
        + "}\n")
    .build();
```

* JavaPoet生成控制流代码：

生成for 循环

可以使用MethodSpec的beginControlFlow来开始控制流，endControlFlow结束控制流。

```
MethodSpec main = MethodSpec.methodBuilder("main")
    .addStatement("int total = 0")
    .beginControlFlow("for (int i = 0; i < 10; i++)")
    .addStatement("total += i")
    .endControlFlow()
    .build();
```
*  

## Import static

JavaPoet支持静态引用。

```
ClassName namedBoards = ClassName.get("com.mattel", "Hoverboard", "Boards");

MethodSpec beyond = MethodSpec.methodBuilder("beyond")
    .returns(listOfHoverboards)
    .addStatement("$T result = new $T<>()", listOfHoverboards, arrayList)
    .addStatement("result.add($T.createNimbus(2000))", hoverboard)
    .addStatement("result.add($T.createNimbus(\"2001\"))", hoverboard)
    .addStatement("result.add($T.createNimbus($T.THUNDERBOLT))", hoverboard, namedBoards)
    .addStatement("$T.sort(result)", Collections.class)
    .addStatement("return result.isEmpty() ? $T.emptyList() : result", Collections.class)
    .build();

TypeSpec hello = TypeSpec.classBuilder("HelloWorld")
    .addMethod(beyond)
    .build();
    
JavaFile.builder("com.example.helloworld", hello)
    .addStaticImport(hoverboard, "createNimbus")
    .addStaticImport(namedBoards, "*")
    .addStaticImport(Collections.class, "*")
    .build();
```
生成的代码就是


```
package com.example.helloworld;

import static com.mattel.Hoverboard.Boards.*;
import static com.mattel.Hoverboard.createNimbus;
import static java.util.Collections.*;
```


参考：
- https://juejin.im/entry/58fefebf8d6d810058a610de
- https://github.com/square/javapoet
