---
layout: post
title: Dagger2 简明教程2
date: '2020-02-15 17:02'
categories: Android
tags: [Android, Dagger2]
description: Dagger2 使用2
comments: true
---

# 参考文章

https://medium.com/@harivigneshjayapalan/dagger-2-for-android-beginners-introduction-be6580cb3edb


# 第五章  依赖范围

JDK中关于依赖注入提供的支持。

```
javax.inject
    Inject
    Named
    Provider
    Qualifier
    Scope
    Singleton
```

Provider为接口，其他的都是注解。我们前面接触过Inject和Provider。下面来说说其它几个。

## @Named注解

## @Singleton

## @Qualifier

## @Scope

# 第六章  Component 结构

Component 管理依赖实例，提供依赖除了前面说的方法，还有一个重要的来源 就是其它Component 。 多个Component之间的关系可以用object graph描述，我称之为依赖关系图。在 Dagger2 中 Component 的组织关系分为两种：

* 依赖关系，一个 Component 依赖其他 Compoent 提供的依赖实例，具体使用是用在Component中的dependencies声明。

* 继承关系，一个 Component 继承其它 Component 提供更多的依赖，SubComponent 就是继承关系的体现。


依赖关系

```

@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    void inject(Application app); 
    Application application(); // 必须声明出来，暴露出来给其它模块用的依赖
}

@Component(dependencies = ApplicationComponent.class)
public interface ActivityComponent {
    void inject(MainActivity activity);
}

```

> ApplicationComponent 和 ActivityComponent 如果其中一个声明了作用域的话，另外一个也必须声明，而且它们的 Scope 不能相同，ApplicationComponent 的生命周期 >= ActivityComponent 的。ActivityComponent 的 Scope 不能是 @Singleton。
Dagger 2 中 @Singleton 的 Component 不能依赖其他的 Component。

* 被依赖的Component需要把暴露的依赖实例用显式的接口声明，
* 依赖关系中的Component的Scope不能相同，因为它们的生命周期不同。

继承关系

```
@Module(subcomponents = SonComponent.class)
public class ApplicationModule {
    @Provides
    Application provideCar() {
        return app;
    }
}

@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {
    void inject(Application app);   // 继承关系中不用显式地提供暴露依赖实例的接口
    
    //需要将SubComponent 追加到 被依赖的Component中
    ActivityComponent addSub(ActivityModule activitymodule);
}

@SubComponent(modules = ActivityModule.class)
public interface ActivityComponent {
    void inject(MainActivity activity);

    @ActivityComponent.Builder
    interface Builder { 
         // SubComponent 必须显式地声明 ActivityComponent.Builder，parent Component 需要用 Builder 来创建 SubComponent
        ActivityComponent build();
    }
}
。
```

@SubComponent的写法与@Component一样，只能标注接口或抽象类，与依赖关系一样，SubComponent 与 parent Component 的 Scope 不能相同，只是 SubComponent 表明它是继承扩展某 Component 的。


怎么表明一个 SubComponent 是属于哪个 parent Component 的呢？只需要在 parent Component 依赖的 Module 中的subcomponents加上 SubComponent 的 class，然后就可以在 parent Component 中请求 SubComponent.Builder。
。

### 依赖关系 vs 继承关系

相同点：

两者都能复用其他 Component 的依赖

有依赖关系和继承关系的 Component 不能有相同的 Scope

区别：

依赖关系中被依赖的 Component 必须显式地提供公开依赖实例的接口，而 SubComponent 默认继承 parent Component 的依赖。

依赖关系会生成两个独立的 DaggerXXComponent 类，而 SubComponent 不会生成 独立的 DaggerXXComponent 类。

在 Android 开发中，Activity 是 App 运行中组件，Fragment 又是 Activity 一部分，这种组件化思想适合继承关系，所以在 Android 中一般使用 SubComponent。
。
