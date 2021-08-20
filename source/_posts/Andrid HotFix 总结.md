---
layout: post
title: Andrid HotFix 总结
date: '2016-06-14 14:01'
---

# Andrid HotFix 总结

android Hotfix 对于目前APP的线上BUG处理有很多优势，它避免了重新打包，发版等可以节省公司的很多资源，甚至能够避免重大的损失。目前Hotfix方案有阿里开源的AndFix和Dexposed。还有就是基于qq空间分包方案的不同实现，原文地址：[安卓App热补丁动态修复技术介绍](http://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a&scene=1&srcid=1031x2ljgSF4xJGlH1xMCJxO&uin=MjAyNzY1NTU=&key=04dce534b3b035ef58d8714d714d36bcc6cc7e136bbd64850522b491d143aafceb62c46421c5965e18876433791d16ec&devicetype=iMac%20MacBookPro12,1%20OSX%20OSX%2010.10.5%20build%2814F27%29&version=11020201&lang=zh_CN&pass_ticket=7O/VfztuLjqu23ED2WEkvy1SJstQD4eLRqX%2b%2bbCY3uE=)。希望大家能仔细阅读。

 * https://github.com/dodola/HotFix
 * https://github.com/jasonross/Nuwa
 * https://github.com/alibaba/AndFix  
 * https://github.com/alibaba/dexposed

##HotFix&&Nuwa

HotFix和Nuwa都是基于qq空间分包方案的库。目前，HotFix这个项目目前作者并不打算继续维护[详情](https://github.com/dodola/HotFix/issues/9)，推荐使用Nuwa,而Nuwa使用的是gradle1.5以前的构建系统。而目前的构建系统基本都基于gradle2.10以上，这个项目也不能直接使用。

##Dexposed
Dexposed 项目目前只能支持Android 2.3 to 4.4 (no include 3.0)适用范围太窄了。不考虑使用。官方支持情况如下

Runtime | Android Version | Support
------  | --------------- | --------
Dalvik  | 2.2             | Not Test
Dalvik  | 2.3             | Yes
Dalvik  | 3.0             | No
Dalvik  | 4.0-4.4         | Yes
ART     | 5.0             | Testing
ART     | 5.1             | No
ART     | M               | No

##AndFix

AndFix支持从2.3至6.0的所有系统， 包括Arm和x86架构，Dalvik以及ART运行时。AndFix的patch文件格式是.apatch。

关于更多AndFix的原理以及使用，大家可以参考官网  [AndFix](https://github.com/alibaba/AndFix)
