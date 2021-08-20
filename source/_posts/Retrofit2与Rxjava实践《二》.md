---
title: Retrofit2与Rxjava实践《二》
date: 2016-05-20 23:49:26
tags:
---
接着上一结的继续说,实际使用Http过程中，我们经常会遇一些通用的问题。下面一一列举一些。
# BaseResult类问题
在实际的项目中，我们使用的接口返回的数据，大多都是下面的类型
```java
{
    	errorCode: 0,
    	errorMessage: "",
    	data: {}
}
```
</pre>
其中，errorCode用来表示业务处理结果是否正确，
errorMessage是当errorCode表示业务处理结果不正确时的异常信息，通常都是结果不正确时才会有该字段，而data是正常的业务数据。

如果遇到这种时，那我们该怎么办呢？通常，我们只需要其中的dat来刷新我们的页面或者作其他处理。

首先，我们可以创建一个范型基础类
```java
public class HttpResult<T> {
    private int errorCode;
    private String errorMessage;
    private T data;
｝
```
其中，T代表将来不同业务实际的数据模型。

类比上面提到的IpEntity,我们可以创建如下类
```java
public class HttpResult<T> {
    private int code;
    private T data;
｝
```
对应的IPEntity的T就是DataBean，而我们对应的API接口就变成
```java
@GET("/service/getIpInfo.php")
Call<HttpResult<DataBean>> searchIp(@Query("ip") String ip);
或者
@GET("/service/getIpInfo.php")
Observable<HttpResult<DataBean>> searchIp(@Query("ip") String ip);
```

使用时就变成
```java
ipService.searchIp("63.223.108.42").enqueue(new Callback<HttpResult<DataBean>>() {
    @Override
    public void onResponse(Call<HttpResult<DataBean>> call, Response<HttpResult<DataBean>> response) {
        ipInfo.setText(response.body().toString());
    }

    @Override
    public void onFailure(Call<HttpResult<DataBean>> call, Throwable t) {
        ipInfo.setText(t.toString());
    }
});
或者
ipService.searchIp("63.223.108.42")
        .subscribeOn(Schedulers.io())
        .unsubscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<HttpResult<DataBean>>() {
    @Override
    public void onCompleted() {
        Toast.makeText(MainActivity.this, "get Ip address!", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onError(Throwable e) {
        ipInfo.setText(e.toString());
    }

    @Override
    public void onNext(HttpResult<DataBean> ipEntity) {
        ipInfo.setText(ipEntity.toString());
    }
});
```
# Header问题
实际使用过程中, 我们通常都会在所有的请求的中添加一些固定的header，此时我们就可以使用OKHtttp的Interceptor拦截器机制。
例如：如果在每个请求中添加token,
 httpClient.addNetworkInterceptor(new Interceptor() {
            @Override
            public Response intercept(Chain chain) throws IOException {
                //原始请求对象
                Request originRequest = chain.request();
                //自定义处理过的请求对象
                Request lastRequest = originRequest.newBuilder().addHeader("token","xxxxx").build();
                //请求结果
                Response response = chain.proceed(lastRequest);
                //做一些其它的事情
                String token = response.header("token");
                return response;
            }
        });
