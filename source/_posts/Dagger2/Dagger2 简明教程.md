---
layout: post
title: Dagger2 简明教程
date: '2020-02-14 17:02'
categories: Android
tags: [Android, Dagger2]
description: Dagger2 使用
comments: true
---

# Dagger2 简明教程

[TOC]

# 第一章：依赖注入与控制反转

面向对象编程需要处理各种依赖关系，多个对象或者组件互相依赖。通常，我们都会自己通过new来创建依赖对象。但是，这样通常会带来一系列的问题。
1. 对象和它的依赖项形成强依赖，如果我们要替换依赖项的实现，那我们就不得不去修改对象。
2. 在对对象进行测试时，我们不得不先创建它的依赖项，如果对象依赖项很多，那么将非常痛苦。


依赖注入与控制反转是一种思想。在这个世界里，对象的依赖不是有自己提供，而是由外部容器提供，容器由所有它需要的依赖，容器找到它的依赖，并将它们注入给对象。

**控制反式IOC**: 控制反转是将组件间的依赖关系从程序内部提到外部由容器来管理。

**依赖注入D**: 依赖注入是指将组件的依赖通过外部以参数或其他形式由容器注入，并非有组件自己提供。通常依赖注入有两种方式：设值注入和构造注入。设值注入指通过set的方法传入，构造注入指通过构造函数的方式传入参数。


# 第二章：JDK中的依赖注入

JDK本身提供了依赖注入的框架，但是JDK没有提供实现。我们先看下JDK中的框架。

![image]([https://img-blog.csdn.net/20170720213543917?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYnJpYmx1ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# 第三章：Dagger2


> 
> 在下面的内容中，我们称需要注入的对象称为对象。那些注入的对象我们称为依赖。
>  ```
>  Class A {
>    B b;
>  }
>  ```
>  我们称A为对象，B为依赖。
>


Dagger2是依赖注入与控制反转思想的一种Java实现。除Dagger2外，java中最重量的该框架就是Spring。 Spring是JavaWeb开发最火最优秀的框架。

思考一下： 如果抛却这些框架，自己实现这样一个框架。那么框架需要做哪些最基本的事？ 

1. 框架需要知道对象需要哪些依赖。 
2. 框架需要知道如何生成这些依赖。
3. 框架需要知道如何把依赖注入给对象。

有了这些，我们就可以有一个基本的框架实现。

在额外思考一下，多个对象都依赖一个依赖项，框架在生成依赖时，：

1. 如何做到依赖是单例。
2. 如何区分出不用的依赖。


那我们来领略一下Dagger2是怎么做的吧。

## Dagger2 简介
Dagger是一个完全静态的，Java和Android的编译时依赖注入框架。它改编了由Square 创建的早期版本，现在由Google维护。

## Dagger2 简明使用

1. 引入依赖

```
    //纯java依赖
    compile 'com.google.dagger:dagger:2.16'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.16'
    
    //下面的可不添加
    //Android依赖库-非必须项
    compile 'com.google.dagger:dagger-android:2.16'
    annotationProcessor 'com.google.dagger:dagger-android-processor:2.16'
    //Android support依赖库-非必须项
    compile 'com.google.dagger:dagger-android-support:2.16' 
```

2. 基本使用

## Daggaer2 剖析

按照我们提出的问题，我们一步一步梳理Dagger2是如何解决这些问题的。

### 问题1 -》框架如何知道对象需要哪些依赖。

```
public class MainActivity extends AppCompatActivity {

    @Inject
    Boss boss;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

```

### 问题2 -》框架需要知道如何生成这些依赖。

1. 使用@Inject注解在类的构造函数上

对于我们自己写的类， 我们可以直接类的构造函数上添加@Inject 来表示提供依赖。

```
public class Boss {

    @Inject
    public Boss() {
    }
}

```
2. 使用@module注解

对于第三发类等我们不能修改的类或者其它不能添加@Inject的类， 我们可以使用模块来表示提供依赖. Dagger2使用@Module和@Provides来提供依赖。


```
@Module
public class ActivityModule {

    @Provides
    public ExecutorService provideExecutorService() {
        return Executors.newSingleThreadExecutor();
    }

    @Provides
    public ThreadFactory provideThreadFactory() {
        return Executors.defaultThreadFactory();
    }

}
```

### 问题3 -》框架需要知道如何把依赖注入给对象

Dagger2 通过编著Component注解的类完成注入工作。形式如下

```
@Component()
public interface ActivityComponent {
}
```
build一下工程，将会为我们生成如下的类。

### 生成的类

1. 提供依赖的类，Dagger2为每一个提供依赖的类生成一个工厂类来生成依赖。
他们的继承广西是这样的

```
graph BT
CLASS_Factory-->Factory
Factory-->Provider
```

```
public final class Boss_Factory implements Factory<Boss> {
  private static final Boss_Factory INSTANCE = new Boss_Factory();

  @Override
  public Boss get() {
    return provideInstance();
  }

  public static Boss provideInstance() {
    return new Boss();
  }

  public static Boss_Factory create() {
    return INSTANCE;
  }

  public static Boss newBoss() {
    return new Boss();
  }
}
```

使用module注解的类，生成的类名为 模块名+方法名字+Factory

```
public final class ActivityModule_ProvideExecutorServiceFactory
    implements Factory<ExecutorService> {
  private final ActivityModule module;

  public ActivityModule_ProvideExecutorServiceFactory(ActivityModule module) {
    this.module = module;
  }

  @Override
  public ExecutorService get() {
    return provideInstance(module);
  }

  public static ExecutorService provideInstance(ActivityModule module) {
    return proxyProvideExecutorService(module);
  }

  public static ActivityModule_ProvideExecutorServiceFactory create(ActivityModule module) {
    return new ActivityModule_ProvideExecutorServiceFactory(module);
  }

  public static ExecutorService proxyProvideExecutorService(ActivityModule instance) {
    return Preconditions.checkNotNull(
        instance.provideExecutorService(),
        "Cannot return null from a non-@Nullable @Provides method");
  }
}


public final class ActivityModule_ProvideThreadFactoryFactory implements Factory<ThreadFactory> {
  private final ActivityModule module;

  public ActivityModule_ProvideThreadFactoryFactory(ActivityModule module) {
    this.module = module;
  }

  @Override
  public ThreadFactory get() {
    return provideInstance(module);
  }

  public static ThreadFactory provideInstance(ActivityModule module) {
    return proxyProvideThreadFactory(module);
  }

  public static ActivityModule_ProvideThreadFactoryFactory create(ActivityModule module) {
    return new ActivityModule_ProvideThreadFactoryFactory(module);
  }

  public static ThreadFactory proxyProvideThreadFactory(ActivityModule instance) {
    return Preconditions.checkNotNull(
        instance.provideThreadFactory(),
        "Cannot return null from a non-@Nullable @Provides method");
  }
}


```


2. 需要注入的类，Dagger2为每一个需要注入的类生成一个实现MembersInjector的类，来完成注入工作。

```
graph BT
CLASS_MembersInjector-->MembersInjector
```

```
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
  private final Provider<Boss> bossProvider;

  public MainActivity_MembersInjector(Provider<Boss> bossProvider) {
    this.bossProvider = bossProvider;
  }

  public static MembersInjector<MainActivity> create(Provider<Boss> bossProvider) {
    return new MainActivity_MembersInjector(bossProvider);
  }

  @Override
  public void injectMembers(MainActivity instance) {
    injectBoss(instance, bossProvider.get());
  }

  public static void injectBoss(MainActivity instance, Boss boss) {
    instance.boss = boss;
  }
}

```

在生成的类中，Dagger会为每个需要注入的成员，
* 添加一个Provider类型的成员。
* 构造函数和静态工厂方法添加一个对应的Provider类型的参数。
* 添加inject*的方法，并在injectMembers调用。
* 

3. 关联起来二者的component
Dagger2会生成对应的类实现或者继承我们的Component注解的类。

```
public final class DaggerActivityComponent implements ActivityComponent {
  private DaggerActivityComponent(Builder builder) {}

  public static Builder builder() {
    return new Builder();
  }

  public static ActivityComponent create() {
    return new Builder().build();
  }

  public static final class Builder {
    private Builder() {}

    public ActivityComponent build() {
      return new DaggerActivityComponent(this);
    }
  }
}

```

可以看到，我们的类使用了静态的工厂方法和Builder方法的设计模式。

到现在为止，我们只是看到了基本的实现。而且component类里有没有真的注入。接下来，我们修改Componet来完成注入。

```
@Component()
public interface ActivityComponent {
    void inject(MainActivity mainActivity);
}

```

Build工程后，可以看到生成的类增加了下面两个方法。实际上，Dagger2会每一个需要注入的成员 增加这样两个方法。


```
void inject(MainActivity mainActivity)
和 
private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectBoss(instance, new Boss());
    return instance;
  }
```


```
public final class DaggerActivityComponent implements ActivityComponent {
  private DaggerActivityComponent(Builder builder) {}

  public static Builder builder() {
    return new Builder();
  }

  public static ActivityComponent create() {
    return new Builder().build();
  }

  public static final class Builder {
    private Builder() {}

    public ActivityComponent build() {
      return new DaggerActivityComponent(this);
    }
  }
}

//变更后

public final class DaggerActivityComponent implements ActivityComponent {
  private DaggerActivityComponent(Builder builder) {}

  public static Builder builder() {
    return new Builder();
  }

  public static ActivityComponent create() {
    return new Builder().build();
  }

  @Override
  public void inject(MainActivity mainActivity) {
    injectMainActivity(mainActivity);
  }

  private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectBoss(instance, new Boss());
    return instance;
  }

  public static final class Builder {
    private Builder() {}

    public ActivityComponent build() {
      return new DaggerActivityComponent(this);
    }
  }
}
```


到此，好像所有的类都有了，但少了一点，为MainActivity注入的方法，好像没人调用。我们增加最后一步，来完成所有的工作。

```
public class MainActivity extends AppCompatActivity {
    @Inject
    Boss boss;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerActivityComponent.create().inject(this);
        System.out.println(boss);
    }
}

```

运行代码我们就可以看到boss，已经被设置了值了。


```
06-21 11:45:31.189 1710-1710/davy.dagger2learn I/System.out: davy.dagger2learn.Boss@8c15cb2
```

来看下，那些基类的代码

```

import javax.inject.Provider;
public interface Factory<T> extends Provider<T> {
}

public interface MembersInjector<T> {
    void injectMembers(T instance);
}


```

上面说到了使用@Inject提供注解来提供依赖的类BOSS。 但是使用@Module注解的类 并没有说到。下面说这个问题。

首先，修改MainActivity，添加

```
@Inject
ThreadFactory threadFactory;
```

接下来，build工程，发现报错了信息如下：

```
错误: [Dagger/MissingBinding] java.util.concurrent.ThreadFactory cannot be provided without an @Provides-annotated method.
java.util.concurrent.ThreadFactory is injected at
davy.dagger2learn.MainActivity.threadFactory
davy.dagger2learn.MainActivity is injected at
davy.dagger2learn.ActivityComponent.inject(davy.dagger2learn.MainActivity)
```

意思是：使用@Provides注解的方法提供的类, 无法提供依赖。

正确的方式是。使用Provides提供依赖 需要在Compoent中将Module添加进来。

修改如下：

```
@Component(modules = {ActivityModule.class})
public interface ActivityComponent {
    void inject(MainActivity mainActivity);
}
```

再次运行，发现功能正常了。

查看生产的代码

```
public final class DaggerActivityComponent implements ActivityComponent {
  private ActivityModule activityModule;

  private DaggerActivityComponent(Builder builder) {
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static ActivityComponent create() {
    return new Builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {
    this.activityModule = builder.activityModule;
  }

  @Override
  public void inject(MainActivity mainActivity) {
    injectMainActivity(mainActivity);
  }

  private MainActivity injectMainActivity(MainActivity instance) {
    MainActivity_MembersInjector.injectBoss(instance, new Boss());
    MainActivity_MembersInjector.injectThreadFactory(
        instance,
        ActivityModule_ProvideThreadFactoryFactory.proxyProvideThreadFactory(activityModule));
    return instance;
  }

  public static final class Builder {
    private ActivityModule activityModule;

    private Builder() {}

    public ActivityComponent build() {
      if (activityModule == null) {
        this.activityModule = new ActivityModule();
      }
      return new DaggerActivityComponent(this);
    }

    public Builder activityModule(ActivityModule activityModule) {
      this.activityModule = Preconditions.checkNotNull(activityModule);
      return this;
    }
  }
}

```

我们发现，

* 对于使用@Inject在构造函数的类，Dagger2直接使用new 创建类。
* 对于使用Module的类，是在DaggerActivityComponent添加module 成员来提供依赖。

# 第四章：总结

上面，我们总结并学会了Dagger2的基本用法。
1. 使用@Inject在成员上，标识需要注入
2. 使用@Inject在构造函数上，标识可以提供依赖。
3. 使用@Module和@Provides标注类和方法，标识提供依赖。
4. 使用@Component标注类，并添加inject方法，来将上面提供的依赖和需要注入的类结合起来，
5. 使用Daggee*Component来完成注入工作。