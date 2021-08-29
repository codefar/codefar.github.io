---
title: React Native-启动流程分支-准备
categories: React Native
comments: true
tags: [React Native]
date: 2017-01-13 12:00:00
---
# RN启动流程分支-准备

为了简化分析，使用集成RN ARR的方式准备环境。方式：
https://reactnative.cn/docs/integration-with-existing-apps/

Java 层实例创建，其中核心代码如下
```
public class MainActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {

    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        SoLoader.init(getApplicationContext(), /* native exopackage */ false);
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setCurrentActivity(this)
                .setBundleAssetName("index.android.bundle")
                .setJSMainModulePath("index")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "MyReactNativeApp", new Bundle());

        setContentView(mReactRootView);
    }
}

```
我们可以看出，启动流程主要分3部分

1. 创建ReactRootView
1. 创建ReactInstanceManager
1. 启动ReactApplication

