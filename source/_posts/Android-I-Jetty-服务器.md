---
title: Android I-Jetty 服务器
date: 2018-01-30 19:13:04
tags: i-Jetty, 嵌入式服务器, Android, NanoHttpd, Web, http
---

[TOC]

# Android嵌入式服务器

Android应用开发基本都是手机应用作为客户端访问服务器。但是有些时候, 却需要将手机作为服务端对外提供服务。

目前，可用的嵌入式服务器有NanoHttpd和I-Jetty。

NanoHttpd的地址是https://github.com/NanoHttpd/nanohttpd，该框架的好处是简单，轻量，只有一个java文件。坏处也明显: 对于稍复杂的项目，支持力度就不太够，请求并发增多后，稳定性减弱。

I-Jetty是开源Web容器Jetty移植到Android的项目。I-Jetty本身就是一个标准的servlet容器，该框架的好处是拥有标准的ServletApi，也就意味着，我们基于标准的ServletApi开发war，这对于熟悉后台开发的同学来说更简便。如果将来业务变更，我们甚至可以无缝地将war部署到云端。同时，并发稳定性也稳定。

## I-Jetty 在Android的使用

I-Jetty 项目地址：https://github.com/jetty-project/i-jetty。 原来是在googlecode的。

## 下载源码

```
git clone https://github.com/jetty-project/i-jetty.git
```

## 源码结构

- i-jetty

    该目录是i-jetty的核心，实现Android上运行jetty的Android项目。
    我们称之为WebApp的容器。

- example-webapps

  该目录是一个WebApp，也就是war应用的demo项目。我们称之为WebApp项目。  
- console

  该目录是一个控制台项目。是用来控制的。

## 容器应用使用

- 首先配置环境，包括Android, maven等，这些网上很多，请读者自行配置。

- 编译容器项目

  i-jetty 是使用maven构建Android应用的。对于Android开发者来说, 绝大部分基本只接触过ADT+Eclipse 和 Android Studio 这两种开发环境，
  而使用maven构建现在已经很少，而且也不推荐使用了。所以，我们使用Android Studio也就是gradle来构建我们的容器应用。

  1. 创建空项目

        使用Android Studio创建一个空的Android项目，创建时包名使用org.mortbay.ijetty。 创建后，删除res下的所有文件。如果有java代码也都删除。

  2. 拷贝java代码

        进入i-jetty目录，拷贝i-jetty-server目录下src/main/java下 和i-jetty-ui目录下src/下的java源码到我们的源码目录中。

  3. 拷贝res代码

        进入i-jetty-ui目录，拷贝res/下的资源文件到我们的资源res/目录中。
        拷贝resource目录到项目中的src/main/目录下，和java/和res保持同一目录。

  4. 移植依赖

        i-jetty项目的依赖是使用POM文件定义的，我们需要转为gradle方式。

```
    implementation 'javax.servlet:servlet-api:2.5'
    implementation 'org.eclipse.jetty:jetty-client:7.6.0.RC4'
    implementation 'org.eclipse.jetty:jetty-webapp:7.6.0.RC4'
    implementation 'org.eclipse.jetty:jetty-deploy:7.6.0.RC4'
```
如果你使用的是旧版本的gradle, 需改为compile

```
    compile 'javax.servlet:servlet-api:2.5'
    compile 'org.eclipse.jetty:jetty-client:7.6.0.RC4'
    compile 'org.eclipse.jetty:jetty-webapp:7.6.0.RC4'
    compile 'org.eclipse.jetty:jetty-deploy:7.6.0.RC4'
```

  5.  移植AndroidManefest.xml

        拷贝i-jetty-ui的AndroidManefest.xml文件，替换现在的AndroidManefest.xml文件。


  6. 处理旧版本API。

    a. i-jetty-ui的IJettyService使用了Notification的setLatestEventInfo旧版API，我们使用新版的替换。


```
// Notification notification = new Notification(R.drawable.ijetty_stat, text, System.currentTimeMillis());

Notification.Builder builder = new Notification.Builder(IJettyService.this);
Notification notification = builder.setSmallIcon(R.drawable.ijetty_stat)
                                .setContentText(text)
                                .setTicker(text)
                                .setWhen(System.currentTimeMillis())
                                .setContentTitle(getText(R.string.app_name))
                                .setContentText(text)
                                .setContentIntent(contentIntent)
                                .build();
```


## 配置
到此，容器项目到此构建完成。

> 由于要往SD卡读写文件，如果是在api23以上系统运行，因为没有做动态的权限申请请先在设置-应用里给应用赋予SD卡权限。

![](https://note.youdao.com/yws/api/personal/file/WEBbdb93fa85accbe522e166840ebcb3a88?method=getImage&version=1962&cstk=aNW9Q6WS)

到此，服务器已经运行起来了， 点击configure，可以配置我们的服务器。
例如： 端口，ssl, nio 等等。

# Web 应用部署

## example-webapps构建
通常Web应用都是由maven构建的。，我们直接进进入example-webapps/hello应用目录。执行


```
mvn clean package
```

**提示**

```
[INFO] I-Jetty :: Example Webapps Parent .................. SUCCESS [  0.148 s]
[INFO] I-Jetty :: Hello ................................... FAILURE [  0.995 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
```

原因：最初Android版本SDK里的dx.jar都是放在platform-tools/lib/dx.jar这的，现在都是在build-tools下的每个版本里都有自己的dx.jar.


解决：复制SDK里的dx.jar(最好是你构建使用的Android SDK版本)到hello项目的根目录(也就是example-webapps目录下)。修改hello目录下的POM文件，


```
<argument>${project.build.directory}/../../dx.jar</argument>
<!--<argument>${env.ANDROID_HOME}/platform-tools/lib/dx.jar</argument>-->
```

使用<argument>${project.build.directory}/../../dx.jar</argument> 替换

<argument>${env.ANDROID_HOME}/platform-tools/lib/dx.jar</argument>

.

再次执行构建，mvn clean package。

如果没问题，在target目录下,生成了我们的war包。


## 部署war包

adb push target/hello-3.2-SNAPSHOT.war /sdcard/jetty/webapps/hello.war

将war push 到 i-jetty的部署目录并改名。重启i-jetty，使用浏览器访问 http://localhost:8080/hello。

![image](https://note.youdao.com/yws/api/personal/file/WEB3756db5e8fafad5759d772e6d6fb41a7?method=getImage&version=2023&cstk=aNW9Q6WS)


访问http://localhost:8080/sayit/*

![image](https://note.youdao.com/yws/api/personal/file/WEB64a5e3837dba7e0a3382bfaad61e5016?method=getImage&version=2031&cstk=aNW9Q6WS)


源码地址
https://github.com/davyjoneswang/AndroidIJetty
https://github.com/codefar/i-jetty
