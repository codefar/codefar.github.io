---
layout: post
title:Java性能监控系列 —— java.lang.instrument
date: '2019-08-14 16:02'
categories: Java
tags: [Java]
description: Java性能监控系列 —— java.lang.instrument
comments: true
---

# Java性能监控系列 —— java.lang.instrument

https://blog.csdn.net/shi1122/article/details/7981194#

Instrumentation是Java5提供的新特性。使用Instrumentation，开发者可以构建一个代理，用来监测运行在JVM上的程序。监测一般是通过在执行某个类文件之前，对该类文件的字节码进行适当的修改进行的。

java.lang.instrument中需要关注的是ClassFileTransformer和Instrumentation接口。每个代理类必须实现ClassFileTransformer接口，这个接口提供了一个transform方法：
      
      
```
byte[] transform(ClassLoader loader,
                 String className,
                 Class<?> classBeingRedefined,
                 ProtectionDomain protectionDomain,
                 byte[] classfileBuffer)
                 throws IllegalClassFormatException

```

通过这个方法，代理可以得到虚拟机载入的类的字节码，并可对其进行修改，完成字节码级的修改。   
各个参数的含义为：
- loader：将被转换的类的类装载器，如果是启动类装载器则此参数可以为空；
- className：类名字，不过这是JVM规范定义的全限名字如java/util/List
- protectionDomain:保护域，跟安全有关；
- classFileBuffer：这个便是被代理类字节码流，正是通过操作这个buffer完成对字节码的修改；

对于函数的返回值，如果返回null，则表示不对类的字节码做任何的修改，否则应该返回修改过的byte[]对象。  

除了实现ClassFileTransformer接口外，我们还需要提供一个公共的静态方法：

```
public static void premain(String agentArgs, Instrumentation inst)
```

一般会在这个方法中创建一个代理对象，通过Instrumentation对象的addTransformer()方法，将创建的代理对象再传递给虚拟机。

一个简单的演示实例：

```
public class HelloWorld implements ClassFileTransformer {
 
	@Override
	public byte[] transform(ClassLoader loader, String className,
			Class<?> classBeingRedefined, ProtectionDomain protectionDomain,
			byte[] classfileBuffer) throws IllegalClassFormatException {
		
		System.out.println("java.lang.instrument, hello world!");
		
		return null;
	}
	
	public static void premain(String args,Instrumentation inst){
		inst.addTransformer(new HelloWorld());
	}
 
}

```

监控类：

```
public class Example {
	
	public static void main(String[] args){
		System.out.println("main class of proxy!");
	}
}

```

将agent类HelloWorld编译成可运行的jar(helloworld.jar)，这里注意在manifest文件中添加premain入口，也即其配置为：

```
Manifest-Version: 1.0
Premain-Class: cn.dstrace.instrument.HelloWorld
```

完成这些步骤以后，通过java -javaagent:helloworld.jar cn.dstrace.instrument.Example运行，则可看到结果：

```
java -javaagent:helloworld.jar cn.dstrace.instrument.Example
```


