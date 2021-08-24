---
layout: post
title:  Android UI 性能优化
date: '2016-06-14 14:01'
categories: Android
comments: true
tags: [Android]
description:  Android UI 性能优化
---

# Android UI 性能优化

## Traceview 


生成.trace 文件

	Traceview 和 CPU CPU Profiler 和 Debug 使用 CPU Profiler 查看 CPU Activity 和函数跟踪。 

	startMethodTracing(“”)
	stopMethodTracing()

	startMethodTracingSampling()
	stopMethodTracing()

	https://developer.android.com/studio/profile/generate-trace-logs

使用 Traceview 查看详细信息


## SysTrace
该systrace命令允许您在系统级别的设备上运行的所有进程中收集和检查时序信息。它结合了来自Android内核的数据，例如CPU调度程序，磁盘活动和应用程序线程，以生成HTML报告。

但是， systrace不会收集有关应用程序进程中代码执行的信息。有关应用程序正在执行的方法以及它使用的CPU资源的更多详细信息，请使用Android Studio CPU分析器。您还可以 使用CPU Profiler 生成跟踪日志并导入和检查它们。

https://developer.android.com/studio/command-line/systrace

该systrace工具在Android SDK工具包中提供，位于android-sdk/platform-tools/systrace/。


```
python systrace.py -o mynewtrace.html sched freq idle am wm gfx view \
    binder_driver hal dalvik camera input res
```


调查UI性能问题
systrace对于检查应用程序的UI性能特别有用，因为它可以分析您的代码和帧速率，以识别问题区域并提出可能的解决方案建议。

> 注意：在运行Android 5.0（API级别21）或更高版本的设备上，渲染帧的工作在UI线程和RenderThread之间分配。
> 在以前的版本中，创建框架的所有工作都在UI线程上完成。


单击框架圆圈会突出显示它，并提供有关系统完成该框架所做工作的其他信息，包括警报。它还会向您显示系统在渲染该帧时执行的方法，因此您可以调查这些方法以获取UI jank的原因.


检测您的应用代码

因为systrace仅在系统级别显示有关进程的信息，所以很难知道应用程序在HTML报告中的给定时间执行的方法。在Android 4.3（API级别18）及更高版本中，您可以使用Trace代码中的类来标记HTML报告中的执行事件。您不需要检测代码来记录跟踪 systrace，但这样做可以帮助您了解应用程序代码的哪些部分可能会导致挂起线程或UI jank。这种方法与使用Debug类不同- Trace该类只是向systrace报表添加标记，而Debug类可以帮助您通过生成.trace文件来检查详细的应用程序CPU使用情况 。

要生成systrace包含已检测跟踪事件的HTML报告，您需要systrace使用-a或--app命令行选项运行并指定应用程序的包名称。


```
public class MyAdapter extends RecyclerView.Adapter<MyViewHolder> {
    ...
    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Trace.beginSection(&quot;MyAdapter.onCreateViewHolder);
        MyViewHolder myViewHolder;
        try {
            myViewHolder = MyViewHolder.newInstance(parent);
        } finally {
            Trace.endSection();
        }
        return myViewHolder;
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        Trace.beginSection(&quot;MyAdapter.onBindViewHolder&quot;);
        try {
            try {
                Trace.beginSection();
                RowItem rowItem = queryDatabase(position);
                mDataset.add(rowItem);
            } finally {
                Trace.endSection();
            }
            holder.bind(mDataset.get(position));
        } finally {
            Trace.endSection();
        }
    }
...
}
```


注意：当您beginSection(String)多次呼叫时，呼叫endSection()仅结束最近调用的beginSection(String)方法。因此，对于嵌套调用，例如上面示例中的调用，您需要确保将每个调用beginSection()与调用 正确匹配endSection()。此外，您无法调用beginSection()一个线程并从另一个线程结束 - 您必须endSection()从同一个线程调用。

