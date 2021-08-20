---
title: Retrofit2与Rxjava实践《－》
date: 2016-05-19 23:49:26
tags:
---

# Retrofit2的基本使用

## 添加Retrofit2和butterknife等依赖

```groovy
// build.gradle
buildscript {
    repositories {
       mavenCentral()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}

// app/build.gradle
apply plugin: 'com.neenbedankt.android-apt'
dependencies {
      compile 'io.reactivex:rxjava:1.1.0'
      compile 'io.reactivex:rxandroid:1.1.0'
      compile 'com.squareup.retrofit2:retrofit:2.0.2'
      compile 'com.squareup.retrofit2:converter-gson:2.0.2'
      compile 'com.squareup.retrofit2:adapter-rxjava:2.0.2'
      compile 'com.squareup.okhttp3:logging-interceptor:3.2.0'  

      compile 'com.jakewharton:butterknife:8.0.1'
      apt 'com.jakewharton:butterknife-compiler:8.0.1'
}
```

 IpEntity.java

```
public class IpEntity {
    /**
     * code : 0
     * data : {"country":"美国","country_id":"US","area":"","area_id":"","region":"","region_id":"","city":"","city_id":"","county":"","county_id":"","isp":"","isp_id":"","ip":"63.223.108.42"}
     */
    private int code;
    private DataBean data;
    // getter and setter ...
}
```



```
public class DataBean {
    private String country;
    private String country_id;
    private String area;
    private String area_id;
    private String region;
    private String region_id;
    private String city;
    private String city_id;
    private String county;
    private String county_id;
    private String isp;
    private String isp_id;
    private String ip;
    // getter and setter ...
}    
```

MainActivity.java

```java
   public class MainActivity extends AppCompatActivity {
       @BindView(R.id.search)
       Button button;
       @BindView(R.id.ip_info)
       TextView ipInfo;

       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
           ButterKnife.bind(this);
       }

       @OnClick(R.id.search)
       public void getIpInfo(){

       }
   }
```

   activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Search Ip"
        android:id="@+id/search"
        android:layout_alignParentRight="true"
        android:layout_alignParentLeft="true" />

    <TextView
        android:text="Hello World!"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/ip_info"/>
</LinearLayout>

```

AndroidManefest.xml添加网络权限

```xml
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```

定义API接口,已淘宝IP查询为例

```java
import retrofit2.Call;
import retrofit2.http.GET;
import retrofit2.http.Query;

public interface IpService {
    @GET("service/getIpInfo.php")
    Call<IpEntity> searchIp(@Query("ip") String ip);
}
```

实现接口API 并添加HttpLoggingInterceptor记录请求

```java
import okhttp3.OkHttpClient;
import okhttp3.logging.HttpLoggingInterceptor;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class ServiceGenerator {
    public static final String API_BASE_URL = "http://ip.taobao.com";
    private static final OkHttpClient.Builder httpClient = new OkHttpClient.Builder();
    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl(API_BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create());

    public static <S> S createService(Class<S> serviceClass) {
        HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
        interceptor.setLevel(HttpLoggingInterceptor.Level.BASIC);
        httpClient.addInterceptor(interceptor);

        Retrofit retrofit = builder.client(httpClient.build()).build();
        return retrofit.create(serviceClass);
    }
}
```
调用查询IP

```java
@OnClick(R.id.search)
public void getIpInfo() {
    final IpService ipService = ServiceGenerator.createService(IpService.class);
    ipService.searchIp("63.223.108.42").enqueue(new Callback<IpEntity>() {
        @Override
        public void onResponse(Call<IpEntity> call, Response<IpEntity> response) {
            ipInfo.setText(response.body().toString());
        }

        @Override
        public void onFailure(Call<IpEntity> call, Throwable t) {
            ipInfo.setText(t.toString());
        }
    });
}
```

结果：

------

D/OkHttp  (31286): --> GET http://ip.taobao.com/service/getIpInfo.php?ip=63.223.108.42 http/1.1
D/OkHttp  (31286): <-- 200 OK http://ip.taobao.com/service/getIpInfo.php?ip=63.223.108.42 (210ms, unknown-length body)

------



## Post方式

提交表单，使用@FormUrlEncoded

```
@FormUrlEncoded
@POST("/service/getIpInfo.php")
Observable<IpEntity> searchIp(@Field("ip") String ip);
//或者
@FormUrlEncoded
@POST("/service/getIpInfo.php")
Observable<IpEntity> searchIp(@FieldMap Map<String,String> ip);
```

# RxJava使用

```java
private static Retrofit.Builder builder =
        new Retrofit.Builder()
                .baseUrl(API_BASE_URL)
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create());
```

在ServiceGenerator为添加 .addCallAdapterFactory(RxJavaCallAdapterFactory.create())

修改ApiService

```java
public interface IpService {
    @GET("/service/getIpInfo.php")
    Observable<IpEntity> searchIp(@Query("ip") String ip);
}
```

修改调用方式

```java
ipService.searchIp("63.223.108.42")
        .subscribeOn(Schedulers.io())
        .unsubscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<IpEntity>() {
    @Override
    public void onCompleted() {
        Toast.makeText(MainActivity.this, "get Ip address!", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onError(Throwable e) {
        ipInfo.setText(e.toString());
    }

    @Override
    public void onNext(IpEntity ipEntity) {
        ipInfo.setText(ipEntity.toString());
    }
});
```
