---
title: Android组件前台 后台?
date: 2015-12-20 18:20:51
tags:  
	- Android
---

# Is my Android app currently foreground or background?

Update, 2015-03-27: Finally got around to having another look at this, attempting to take into account the feedback from commenters.

I just drafted a new version that tries to respond immediately using onStart/onStop when possible, and deals with edge cases like received phone-calls using a delayed Runnable posted to a Handler like the original.

This time I posted an AndroidStudio project as a new github repo rather than just a gist of the interesting bit.

I'm not convinced the defensive WeakReferences to Listeners are strictly necessary, but it seems pear cider brings out my cautious side (its Friday night, what can I say).

Fair warning: I haven't tested this exhaustively, YMMV.

Update, 2015-01-30: Lots of interesting discussion in the comments. Nobody, myself included, is particularly happy with the non-determinism inherent in posting runnables to a handler with an arbitrary delay.

Graham Borland pointed out that if you use onStart/onStop rather than onResume/onPause, you no longer need clever strategies or hacks to determine whether you really have gone background, but others have raised edge cases that complicate matters: phone calls trigger onPause but not onStop, and configuration changes (e.g. rotating the device) call onPause->onStop->onStart->onResume which would toggle our state from foreground to background and back to foreground again.

Original post:

Android doesn't directly provide a way to know if your app is currently foreground or background, by which I mean actively running an Activity (screen on, user present, and your app currently presenting UI to the user).

Obviously if you're coding in an Activity then for almost all of the time (e.g. in any callbacks other than onPause, onStop, or onDestroy) you already know you are foreground, however if you have Service's or BroadcastReceiver's that need to adjust their behaviour when the app is foreground vs. background you need a different approach.

Since API level 14 (Android 4, ICS) we can easily obtain this information by hooking into the activity lifecycle events using Application.registerActivityLifecycleCallbacks.

Using this method we can register a single listener that will be called back whenever an activity lifecycle method is called on any activity in our application. We could call our listener class Foreground, and hook it in to the activity lifecycle methods by providing a custom Application class for our application:

    class MyApplication extends Application {
        public void onCreate(){
            Foreground.init(this ;
        }
    }

Of course, we need to register this custom Application class in our Manifest.xml:

	<application
	    android:label="@string/app_name"
	    android:theme="@style/AppTheme"
	    android:name=".MyApplication">

So, what does our Foreground class look like? Let's begin by creating a class that implements ActivityLifecycleCallbacks and allows only one instance of itself to be created via a static method:

	class Foreground implements Application.ActivityLifecycleCallbacks {

	    private static Foreground instance;

	    public static void init(Application app){
	        if (instance == null){
	            instance = new Foreground();
	            app.registerActivityLifecycleCallbacks(instance);
	        }
	    }

	    public static Foreground get(){
	        return instance;
	    }

	    private Foreground(){}

        // TODO: implement the lifecycle callback methods!

	}

This approach of using Singleton's is used a lot in Android programming, as it is a technique recommended by Google.

OK, so we have a class that we can initialise from our Application and then retrieve from any code in our app using Foreground.get(). Now we need to implement the lifecycle callbacks to track the foreground/background status of our app.

To do that we'll use the onActivityPaused/onActivityResumed method-pair, using paused to signal a potential shift to background, and resumed to know we are in the foreground.

	private boolean foreground;

	public boolean isForeground(){
	    return foreground;
	}

	public boolean isBackground(){
	    return !foreground;
	}

	public void onActivityPaused(Activity activity){
	    foreground = false;
	}

	public void onActivityResumed(Activity activity){
	    foreground = true;
	}

// other ActivityLifecycleCallbacks methods omitted for brevity
// we don't need them, so they are empty anyway ;)
Nice, so now from any code in our application we can test whether we're currently foreground or not, like this:

	Foreground.get().isForeground()

Cool. Are we done? We-ell, depends.

## There are three potential issues here: ##

- The app might go to background at any time, so it would be nice if we could get notified instead of having to continually poll the isForeground method.

- When an application transitions between two Activities there is a brief period during which the first Activity is paused and the second Activity has not yet resumed ... during this period isForeground will return false, even though our application is the foreground app.

- Application.registerActivityLifecycleCallbacks is only available from API-level 14 onwards.

Can we address both of these issues? You betcha!

First lets make it possible to get notified of foreground/background state transitions. We'll add a Listener interface to our Foreground class:

	class Foreground implements Application.ActivityLifecycleCallbacks {

	    public interface Listener {
	        public void onBecameForeground();
	        public void onBecameBackground();
	    }
	    ...
	}
We'll also need to manage any registered listeners and allow listeners to be added and removed. We'll manage registered listeners using a thread-safe and efficient List implementation from java.util.concurrent - CopyOnWriteArrayList:

	private List listeners = new CopyOnWriteArrayList();

	public void addListener(Listener listener){
	    listeners.add(listener);
	}

	public void removeListener(Listener listener){
	    listeners.remove(listener);
	}

And, of course, we'll need to notify our listeners whenever we transition between foreground and background states, which we'll do by updating our onActivityPaused and onActivityResumed methods:

	public void onPause(){
	    foreground = false;
	    for (Listener l : listeners){
	        try {
	            l.onBecameBackground();
	        } catch (Exception exc) {
	            Log.e("Foreground", "Unhappy listener", exc);
	        }
	    }
	}

	public void onResume(){
	    foreground = true;
	    for (Listener l : listeners){
	        try {
	            l.onBecameForeground();
	        } catch (Exception exc) {
	            Log.e("Foreground", "Unhappy listener", exc);
	        }
	    }
	}

Allright, now we're able to register listeners with our Foreground class which will be called-back when we transition from foreground to background and vice-versa.

Bear in mind that the callback is invoked from the lifecycle callbacks and therefore on the main thread. Remember the golden rule of Android development:
>  do not block the main thread. If you don't know what that means you should buy my book :)

Right, that's problem 1 sorted, what about problem 2? (What, you forgot it already? I mean the brief period between onPause being called in Activity A before onResume is called in Activity B).

OK, the issue here is that if we blindly update our foreground/background state in onActivityPaused and onActivityResumed we will always have a period where we're reporting incorrect values. Worse, if we're firing events we'll even tell everyone who's listening that we just went background when we didn't really!

Lets fix that by giving ourselves a brief period of grace before announcing that we've gone background. This is, like many things in engineering, is a compromise - in this case between immediacy and correctness. We'll accept a small delay in order not to falsely report that we went to background.

To do this we'll use one of the nice features of Android's Handler class - the ability to post a Runnable onto the main-thread's event-loop to be executed after a specified delay.

Things are getting a bit more complex now, and we've some extra state to juggle. We're going to introduce another boolean to track whether we're paused or not, and we'll also need to keep a reference to the Runnable that we post to the main thread, so that we can cancel it when necessary.

	private boolean foreground = false, paused = true;
	private Handler handler = new Handler();
	private Runnable check;

A quick note on Handler's: A Handler created with the no-arg constructor will perform all of its work on the thread that created it. Since we're instantiating this Handler inline in the Foreground class, and the Foreground instance is being created on the main thread during our Application's onCreate method callback, any work we post to this Handler will execute on the main thread.

Here's what our updated onActivityPaused and onActivityResumed methods look like:

	@Override
	public void onActivityResumed(Activity activity) {
	    paused = false;
	    boolean wasBackground = !foreground;
	    foreground = true;

	    if (check != null)
	        handler.removeCallbacks(check);

	    if (wasBackground){
	        Log.i(TAG, "went foreground");
	        for (Listener l : listeners) {
	            try {
	                l.onBecameForeground();
	            } catch (Exception exc) {
	                Log.e(TAG, "Listener threw exception!", exc);
	            }
	        }
	    } else {
	        Log.i(TAG, "still foreground");
	    }
	}

	@Override
	public void onActivityPaused(Activity activity) {
	    paused = true;

	    if (check != null)
	        handler.removeCallbacks(check);

	    handler.postDelayed(check = new Runnable(){
	        @Override
	        public void run() {
	            if (foreground && paused) {
	                foreground = false;
	                Log.i(TAG, "went background");
	                for (Listener l : listeners) {
	                    try {
	                        l.onBecameBackground();
	                    } catch (Exception exc) {
	                        Log.e(TAG, "Listener threw exception!", exc);
	                    }
	                }
	            } else {
	                Log.i(TAG, "still foreground");
	            }
	        }
	    }, CHECK_DELAY);
	}

A couple of things worth pointing out here:

onActivityPaused schedules a Runnable to execute after CHECKDELAY milliseconds (CHECKDELAY is set to 500), and captures the Runnable in the check member variable so it can be cancelled if necessary
onActivityResumed removes (cancels) the check callback if there is one, to cancel the pending notification of going background.
So now we have a nice neat mechanism for making direct checks for foreground/background status (Foreground.get().isBackground(), etc), and for being notified of changes to this status using the Listener interface.

## To support API levels below 14  ##

To support API levels below 14 we'd need to hook our Foreground class more directly from the onPause and onResume methods of each individual Activity. This is most easily done by extending all activities in our application from a common base class and implementing the calls to Foreground from there.

For completeness, here's the github gist containing the full code for the Foreground class we've just explored.

	package com.sjl.util;

	import android.app.Activity;
	import android.app.Application;
	import android.content.Context;
	import android.os.Bundle;
	import android.os.Handler;
	import android.util.Log;

	import java.util.List;
	import java.util.concurrent.CopyOnWriteArrayList;

	/**
	 * Usage:
	 *
	 * 1. Get the Foreground Singleton, passing a Context or Application object unless you
	 * are sure that the Singleton has definitely already been initialised elsewhere.
	 *
	 * 2.a) Perform a direct, synchronous check: Foreground.isForeground() / .isBackground()
	 *
	 * or
	 *
	 * 2.b) Register to be notified (useful in Service or other non-UI components):
	 *
	 *   Foreground.Listener myListener = new Foreground.Listener(){
	 *       public void onBecameForeground(){
	 *           // ... whatever you want to do
	 *       }
	 *       public void onBecameBackground(){
	 *           // ... whatever you want to do
	 *       }
	 *   }
	 *
	 *   public void onCreate(){
	 *      super.onCreate();
	 *      Foreground.get(this).addListener(listener);
	 *   }
	 *
	 *   public void onDestroy(){
	 *      super.onCreate();
	 *      Foreground.get(this).removeListener(listener);
	 *   }
	 */

Foreground.java

	public class Foreground implements Application.ActivityLifecycleCallbacks {

	    public static final long CHECK_DELAY = 500;
	    public static final String TAG = Foreground.class.getName();

	    public interface Listener {
	        public void onBecameForeground();
	        public void onBecameBackground();
	    }

	    private static Foreground instance;
	    private boolean foreground = false, paused = true;
	    private Handler handler = new Handler();
	    private List<Listener> listeners = new CopyOnWriteArrayList<Listener>();
	    private Runnable check;

	    /**
	     * Its not strictly necessary to use this method - _usually_ invoking
	     * get with a Context gives us a path to retrieve the Application and
	     * initialise, but sometimes (e.g. in test harness) the ApplicationContext
	     * is != the Application, and the docs make no guarantees.
	     *
	     * @param application
	     * @return an initialised Foreground instance
	     */
	    public static Foreground init(Application application){
	        if (instance == null) {
	            instance = new Foreground();
	            application.registerActivityLifecycleCallbacks(instance);
	        }
	        return instance;
	    }

	    public static Foreground get(Application application){
	        if (instance == null) {
	            init(application);
	        }
	        return instance;
	    }

	    public static Foreground get(Context ctx){
	        if (instance == null) {
	            Context appCtx = ctx.getApplicationContext();
	            if (appCtx instanceof Application) {
	                init((Application)appCtx);
	            }
	            throw new IllegalStateException(
	                "Foreground is not initialised and " +
	                "cannot obtain the Application object");
	        }
	        return instance;
	    }

	    public static Foreground get(){
	        if (instance == null) {
	            throw new IllegalStateException(
	                "Foreground is not initialised - invoke " +
	                "at least once with parameterised init/get");
	        }
	        return instance;
	    }

	    public boolean isForeground(){
	        return foreground;
	    }

	    public boolean isBackground(){
	        return !foreground;
	    }

	    public void addListener(Listener listener){
	        listeners.add(listener);
	    }

	    public void removeListener(Listener listener){
	        listeners.remove(listener);
	    }

	    @Override
	    public void onActivityResumed(Activity activity) {
	        paused = false;
	        boolean wasBackground = !foreground;
	        foreground = true;

	        if (check != null)
	            handler.removeCallbacks(check);

	        if (wasBackground){
	            Log.i(TAG, "went foreground");
	            for (Listener l : listeners) {
	                try {
	                    l.onBecameForeground();
	                } catch (Exception exc) {
	                    Log.e(TAG, "Listener threw exception!", exc);
	                }
	            }
	        } else {
	            Log.i(TAG, "still foreground");
	        }
	    }

	    @Override
	    public void onActivityPaused(Activity activity) {
	        paused = true;

	        if (check != null)
	            handler.removeCallbacks(check);

	        handler.postDelayed(check = new Runnable(){
	            @Override
	            public void run() {
	                if (foreground && paused) {
	                    foreground = false;
	                    Log.i(TAG, "went background");
	                    for (Listener l : listeners) {
	                        try {
	                            l.onBecameBackground();
	                        } catch (Exception exc) {
	                            Log.e(TAG, "Listener threw exception!", exc);
	                        }
	                    }
	                } else {
	                    Log.i(TAG, "still foreground");
	                }
	            }
	        }, CHECK_DELAY);
	    }

	    @Override
	    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {}

	    @Override
	    public void onActivityStarted(Activity activity) {}

	    @Override
	    public void onActivityStopped(Activity activity) {}

	    @Override
	    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {}

	    @Override
	    public void onActivityDestroyed(Activity activity) {}
	}
