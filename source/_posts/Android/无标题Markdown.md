LocalBroadcastManager源码分析

# JavaPoet深入浅出
## 简介
JavaPoet 是生成Java 源文件的工具。

JavaPoet官方地址：https://github.com/square/javapoet
## API 速揽
类图： image

在JavaPoet中，我们通过各种Spec类最终生成我们的JavaFile. Spec类型包括如下。


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
生成的代码就是

package com.example.helloworld;

import static com.mattel.Hoverboard.Boards.*;
import static com.mattel.Hoverboard.createNimbus;
import static java.util.Collections.*;
参考：

https://juejin.im/entry/58fefebf8d6d810058a610de
https://github.com/square/javapoet