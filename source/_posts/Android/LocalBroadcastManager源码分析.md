---
layout: post
title: LocalBroadcastManager源码分析
date: '2018-07-14 17:02'
categories: Android
tags: [Android, LocalBroadcastManager]
description:LocalBroadcastManager源码分析
comments: true
---

[TOC]
# 简介
LocalBroadcastManager是v4兼容包里提供的一个工具。旨在帮助应用在自己的进程内进行部广播注册，发送与处理，
而不是使用Context#sendBroadcast(Intent)发送系统全局广播。

# 优点
LocalBroadcastManager的局部广播（进程内）相对于android.content.Context#sendBroadcast系统的全局广播（扩进程）有很多优点。

- 仅在进程内，不夸进程，分发处理更加高效。
- 私有数据不会泄露。
- 安全，不会被其他人用过恶意广播攻击。


# 类图结构

![image](https://github.com/davyjoneswang/static-res/blob/master/local.png?raw=true)



# 解析

## ReceiverRecord

```
private static class ReceiverRecord {
    final IntentFilter filter;
    final BroadcastReceiver receiver;
    boolean broadcasting;
    
    ReceiverRecord(IntentFilter _filter, BroadcastReceiver _receiver) {
        filter = _filter;
        receiver = _receiver;
    }
}
```

ReceiverRecord代表应用在LocalBroadcastManager注册的一个BroadcastReceiver记录。包含filter过滤器，receiver 广播本身，broadcasting是否在处理中。

## BroadcastRecord
BroadcastRecord代表发送的一个广播Intent和它对应的接受它的BroadcastReceiver集合的记录。

```
private static class BroadcastRecord {
    final Intent intent;
    final ArrayList<ReceiverRecord> receivers;

    BroadcastRecord(Intent _intent, ArrayList<ReceiverRecord> _receivers) {
        intent = _intent;
        receivers = _receivers;
    }
}
```

LocalBroadcastManager 中关键的结构。

```
private final HashMap<BroadcastReceiver, ArrayList<IntentFilter>> mReceivers
        = new HashMap<BroadcastReceiver, ArrayList<IntentFilter>>();
            
private final HashMap<String, ArrayList<ReceiverRecord>> mActions
        = new HashMap<String, ArrayList<ReceiverRecord>>();

private final ArrayList<BroadcastRecord> mPendingBroadcasts
        = new ArrayList<BroadcastRecord>();
```

- mReceivers记录广播接收器和它的过滤器的映射关系。
- mActions记录广播Intent的Action和它的接受处理者的映射关系。LocalBroadcastManager会先根据action去匹配它的接受处理。
- mPendingBroadcasts LocalBroadcastManager以Handler串行队列的形式来处理每个广播，mPendingBroadcasts记录所有他要分发处理的广播和它的处理器。

# 广播处理流程


## 解析Intent

```
final String action = intent.getAction();
final String type = intent.resolveTypeIfNeeded(mAppContext.getContentResolver());
final Uri data = intent.getData();
final String scheme = intent.getScheme();
final Set<String> categories = intent.getCategories();
```

解析出广播的Action、Type、data、scheme、categories字段。

## Action匹配
LocalBroadcastManager先根据Action从mActions中匹配对应的BroadcastReceiver.

## IntentFilter匹配
LocalBroadcastManager从mReceivers中查找在Action匹配匹配出的每一个结果，查看它的IntentFilter是否匹配广播的Intent,去掉不匹配的BroadcastReceiver。剩下的就是所有需要该广播的BroadcastReceiver了。并标记对应ReceiverRecord的广播状态为正在广播，即broadcasting=true。

## 排队广播处理请求
经过匹配后，LocalBroadcastManager将广播的Intent和它的BroadcastReceiver封装成BroadcastRecord，放到mPendingBroadcasts队列中，进行排队等待处理。并将对应的ReceiverRecord的广播状态标记为false,也就是没有处理状态。

## 广播处理

### 异步广播
如果我们发送的是异步广播。LocalBroadcastManager使用Handler, 发送消息。在handleMessage中将现在队列中所有ReceiverRecord请求出队。并依次处理每个ReceiverRecord。也就是直接调用ReceiverRecord中每个BroadcastReceiver的onReceived方法。

### 同步广播
如果我们发送的是同步广播。LocalBroadcastManager依然会使用Handler发送消息，但是在发送消息后会直接在发送广播的线程调用executePendingBroadcasts。

# LocalBroadcastManager问题思考
1. LocalBroadcastManager默认发送的广播都是异步广播。可以在任意线程当中，广播处理在主线程中。所以，异步广播的发送和处理并不同步，可能出现主线程的Looper最终处理到LocalBroadcastManager的消息时，会过了时间较久。也就是消息在消息队列待的时间不确定，有时会很快，有时会很慢。由于APP中忘主线程的消息队列的情况不可控（其它业务，SDK等等），导致广播的实效性相对callback形式可能会差，所以LocalBroadcastManager，异步广播适合时效性并不高的状况。

2. LocalBroadcastManager可以在任意线程发送广播，在sendBroadcast方法里，做了同步处理。但是在主线程调用executePendingBroadcast却没有同步，两个线程会同时使用mPendingBroadcasts，是否会出现同步问题？

3. 有人说 LocalBroadcastManager并不能像EventBus那样选择最终广播处理在哪个线程。不过个人觉得这并不是问题，因为LocalBroadcastManager本身并非完全是一个消息总线的设计，其针对的就是系统广播，而系统广播在Android的范围内，就是肯定运行在主线程当中的。

4. LocalBroadcastManager中的广播的onReveive可能运行在任何线程。
   和系统的广播肯定在主线程还是有差别。

4. 针对LocalBroadcastManager中Handler的实效问题。我们可以考虑发送广播是，使用同步广播, 但是要注意线程问题。

