---
layout: post
title:OkHttp3 源码解析
date: '2020-5-14 16:02'
categories: OkHttp3
tags: 
-OkHttp3
-Android
description: OkHttp3 源码解析
comments: true
---

#  OkHttp3 源码解析

[TOC]

## 简介
OkHttp是一款用于Android和Java程序的HTTP & HTTP/2 客户端。  官方地址：https://github.com/square/okhttp

OkHttp 有很多的优点：

1. 在Http/2上，允许对同一个 host 的请求共同一个 socket。
2. 连接池的使用减少请求延迟（如果 HTTP/2 不支持）
3. 透明的 GZIP 压缩减少数据量大小
4. 响应的缓存避免重复的网络请求

> 本文使用分析基于3.10.0版本，最新版本可从官方地址查询

## 使用配置

### 添加依赖

```
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.10.0</version>
</dependency>
```
或者

```
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
```

### ProGuard
如果使用ProGuard混淆代码，添加下面代码到混淆配置文件。

```
-dontwarn okhttp3.**
-dontwarn okio.**
-dontwarn javax.annotation.**
-dontwarn org.conscrypt.**
# A resource is loaded with a relative path so the package of this class must be preserved.
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase
```
## 基本用法

Get
```
OkHttpClient client = new OkHttpClient();

同步请求

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).execute();
  return response.body().string();
}

异步请求

client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
    }

    @Override
    public void onResponse(Call call, okhttp3.Response response) throws IOException {
    }
});

```

POST
```
public static final MediaType JSON
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}
```

1. 创建OkHttpClient对象，OkHttpClient支持使用Builder模式创建。
2. 创建一个 Request 对象Request 也是通过 Builder 模式来创建，可以配置请求相关参数，包括 url等。
3. 生成CALL，通过OkHttpClient.newCall(request)生成一个 Call对象，Call代表了真正被执行的请求。
4. 执行请求，如果是同步请求，调用CALL的execute 方法，返回一个Response对象， Response代表了响应结果。


## 流程分析

### OkHttpClient创建

```
  final Dispatcher dispatcher;  // 请求的分发器
  final @Nullable Proxy proxy; // 代理
  final List<Protocol> protocols;  // http协议
  final List<ConnectionSpec> connectionSpecs; 
  final List<Interceptor> interceptors;
  final List<Interceptor> networkInterceptors;
  final EventListener.Factory eventListenerFactory;
  final ProxySelector proxySelector;
  final CookieJar cookieJar;
  final @Nullable Cache cache;
```
