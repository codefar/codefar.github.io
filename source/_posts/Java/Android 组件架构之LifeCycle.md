---
layout: post
title:Android 组件架构之LifeCycle
date: '2019-01-14 16:02'
categories: Java
tags: [Java]
description: Android 组件架构之LifeCycle
comments: true
---


# Android 组件架构之LifeCycle

## 简介
Lifecycle是google开发的用于生命周期感知的架构组件。它用来管理Acitivity和Fragment的生命周期。使用Lifecycle可以构建感知生命周期的组件（lifecycle-aware），可以根据activity 或者 fragment 生命周期的状态来调整自己的行为.

## 组成


### Lifecycle
Lifecycle 表示生命周期。 他可以添加、删除LifecycleObserver。
```
public abstract class Lifecycle {
    public abstract void addObserver(@NonNull LifecycleObserver observer);

    public abstract void removeObserver(@NonNull LifecycleObserver observer);

    public abstract State getCurrentState();
}
```

### LifecycleObserver

LifecycleObserver 是观察者对象，它能感知生命周期。

LifecycleObserver 是一个空的接口， 它使用在方法上添加注解的形式，来处理对应的生命周期
```
public interface LifecycleObserver {

}

```

## LifecycleOwner
LifecycleOwner代表Android的生命周期对象，通常是
Activity 或 Fragment.

```
public interface LifecycleOwner {
    Lifecycle getLifecycle();
}

```

## 创建感知生命周期观察者

实现LifecycleObserver，添加处理Android生命周期的方法， 并在该方法上添加对应@OnLifecycleEvent注解。

```
public class CustomLifecycleObserver implements LifecycleObserver {
    
    public CustomLifecycleObserver() {

    }

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void handleOnCreate() {
        System.out.println("handleOnCreate");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void handleOnResume() {
        System.out.println("handleOnResume");
    }
}
```

将我们的CustomLifecycleObserver，添加到对应的Lifecycle中，比如：Activity


```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        getLifecycle().addObserver(new CustomLifecycleObserver());
    }
}
```

结果
```
04-04 15:11:53.183 I/System.out: handleOnCreate
04-04 15:11:53.193 I/System.out: handleOnResume
```

## LifecycleRegistry

我们知道，LifecycleObserver需要添加到Lifecycle中才能生效。
LifecycleRegistry继承自Lifecycle。 Support库里的Fragment和AppCompatActivity，都实现了LifecycleOwner。并且都有一个LifecycleRegistry来存储LifecycleObserver。

```
LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

```

## 生命周期分发原理

### Activity分发
在Android中，只有Activity和Fragment等组件才有生命周期，当AppCompatActivity生命周期变化时，是怎么关联到对应的LifecycleObserver呢？

前面说了，Support库里的Fragment和AppCompatActivity都实现了LifecycleOwner。并且都有一个LifecycleRegistry来存储LifecycleObserver。

SupportActivity.java

```
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ReportFragment.injectIfNeededIn(this);
}
```

其实, 生命周期的分发是委托给ReportFragment处理生命周期。ReportFragment是组件库中的一个Fragment，用来分发生命周期。


```
public class ReportFragment extends Fragment {
    private static final String REPORT_FRAGMENT_TAG = "android.arch.lifecycle"
            + ".LifecycleDispatcher.report_fragment_tag";

    //向Activity添加ReportFragment
    
    public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }

    static ReportFragment get(Activity activity) {
        return (ReportFragment) activity.getFragmentManager().findFragmentByTag(
                REPORT_FRAGMENT_TAG);
    }

    private ActivityInitializationListener mProcessListener;

    private void dispatchCreate(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onCreate();
        }
    }

    private void dispatchStart(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onStart();
        }
    }

    private void dispatchResume(ActivityInitializationListener listener) {
        if (listener != null) {
            listener.onResume();
        }
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        dispatchCreate(mProcessListener);
        dispatch(Lifecycle.Event.ON_CREATE);
    }

    @Override
    public void onStart() {
        super.onStart();
        dispatchStart(mProcessListener);
        dispatch(Lifecycle.Event.ON_START);
    }

    @Override
    public void onResume() {
        super.onResume();
        dispatchResume(mProcessListener);
        dispatch(Lifecycle.Event.ON_RESUME);
    }

    @Override
    public void onPause() {
        super.onPause();
        dispatch(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
    }

    private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        //.....
        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
}
```

ReportFragment在其各个生命周期里调用dispatch 也就是调用LifecycleRegistry的handleLifecycleEvent，将生命周期分发出来。

```
    public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }

    private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
    }
    
    private void sync() {
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            Log.w(LOG_TAG, "LifecycleOwner is garbage collected, you shouldn't try dispatch "
                    + "new events from it.");
            return;
        }
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }
    
    private void backwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                mObserverMap.descendingIterator();
        while (descendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                Event event = downEvent(observer.mState);
                pushParentState(getStateAfter(event));
                observer.dispatchEvent(lifecycleOwner, event);
                popParentState();
            }
        }
    }
    
    
```

#### Fragment分发

Fragment会在自己生命周期里分发状态，具体的是perform*方法里。

```
    void performCreate(Bundle savedInstanceState) {
     //
            this.mLifecycleRegistry.handleLifecycleEvent(Event.ON_CREAT);
        }
    }
```



