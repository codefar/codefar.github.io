---
title: Charles
date: 2016-05-13 16:48:27
tags: 
	Charles Mac 抓包
---

#总结：Mac下使用Charles抓包

Charles一个扩平台的能够允许开发者查看本地机器到网络上所有的Http/Https数据的http代理，监视，反向代理工具.

> Charles is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet. This includes requests, responses and the HTTP headers (which contain the cookies and caching information).

##下载与安装  
 [Charles官网](https://www.charlesproxy.com/) 从官网直接下载即可。如果不能下载,百度网盘共享地址: [charles-proxy-3.11.5b5.dmg](https://pan.baidu.com/s/1bp2rNiB)
安装即可。

##配置
Charles界面中一个session代表了一个抓取信息的会话，里面包含所有的信息。
   
* 配置代理端口。在菜单上选择Proxy->Proxy Settings...，然后配置端口，默认：8888
* 配置手机代理。查看你的Mac的IP地址。 然后，将手机和Mac连在同一WIFI网络中。配置手机的代理IP为你的Mac的IP，端口为第一步设置的端口，默认8888
* 操作手机，然后你就能在Charles中查看你的Http请求了。

##配置Https
如果你使用Https请求，你会发现里面不能查看，配置抓取HTTPS请求。

* 在菜单上选择Proxy->SSl Proxy Settings...,在SSL Proxying 选项卡选择Add， Hosts和Port都填写*（抓取所有的请求），点击OK，然后退出。
* 在菜单选择Help-> SSl Proxying 选择Install charles root certificate, 出现秘钥窜界面，此时，显示刚才的证书不受信任。点击证书，选择信任。退出，此时会要求输入你的密码，输入即可。
* 在菜单选择Help-> SSl Proxying 选择Install charles root certificate on mobile device or remote broser, 按照给出的提示操作。
    1. 配置你的手机的代理 
    2. 然后用手机访问http://www.charlesproxy.com/getssl/ 会让你下载证书。
    3. 下载完后安装。安装方法：进入手机设置-安全-从存储设备安装-> 随便输入一个名字, 按提示安装即可。此处，可能要求设置锁屏安全，设置即可（使用Nexus5要求如此）。

大功告成！可以查看HTTPS请求了。
