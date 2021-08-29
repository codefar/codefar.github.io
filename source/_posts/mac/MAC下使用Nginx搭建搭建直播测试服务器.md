---
title: MAC下使用Nginx搭建搭建直播测试服务器
date: 2018-01-30 19:15:30
tags: Android 直播技术
---

Android 直播技术

## MAC下使用Nginx搭建搭建直播测试服务器

我们使用brew安装Nginx和rtmp模块

```
brew tap homebrew/nginx
brew install nginx-full --with-rtmp-module
```

## 配置NGINX的RTMP模块

打开/usr/local/etc/nginx/nginx.conf，在文件尾部添加

```
rtmp {
    server {
        listen 1935;
        application zbcs {
            live on;
	        record off;
        }
    }
}
```

## 测试安装是否成功

```
nginx -s reload
```

打开浏览器，访问：http://localhost:8080/，如果看到nginx的欢迎页，说明成功。

###  安装android端采集

使用 Android Studio 打开 git@github.com:davyjoneswang/RtmpRecoder.git

***

注意修改MainActivity中的private static final String STREAM_URL = "rtmp://192.168.1.201:1935/zbcs/room"；讲192.168.1.201修改为你的计算器的IP。（如果运行在Aandroid5 以上手机，注意先给权限）

***

## 观看

使用能观看RTMP的播放器播放

如果使用VLC（自行下载安装）。方法如下

打开VLC，菜单：FILE -> Open Network-> 输入地址rtmp://192.168.1.201:1935/zbcs/room-> Open





附

## MAC使用FFMPEG推流

1. 安装FFMPEG

   brew install ffmpeg。

2. 推流

   ffmpeg -re -i ddd.mp4 -vcodec libx264 -acodec aac -f flv rtmp://localhost:1935/zbcs/room

   ​

   ​
