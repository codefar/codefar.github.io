## Systrace简介

````
官方链接：https://developer.android.com/studio/command-line/systrace.html
````

Systrace是Android4.1提供的UI性能优化工具，Systrace通过在UI绘制过程中，收集和展示系统各个方面的信息，记录

Systrace可以追踪的方面包括：
````
         gfx - Graphics
       input - Input
        view - View System
     webview - WebView
          wm - Window Manager
          am - Activity Manager
          sm - Sync Manager
       audio - Audio
       video - Video
      camera - Camera
         hal - Hardware Modules
         app - Application
         res - Resource Loading
      dalvik - Dalvik VM
          rs - RenderScript
      bionic - Bionic C Library
       power - Power Management
       sched - CPU Scheduling
        freq - CPU Frequency
        idle - CPU Idle
        load - CPU Load
````

## 用法
    
文件的生成。
### 命令行

```
python systrace.py [options] [categories]
```
或者

```
python systrace.py --time=10 -o trace.html gfx
```


### Android Studio 图形界面



## 实战

