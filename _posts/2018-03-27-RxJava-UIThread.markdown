---
layout:     post
title:      "RxJava 1.x - How to Observe on the Main UI Thread"
subtitle:   "An alternative to AndroidSchedulers.mainThread() for RxJava 1.x"
date:       2018-03-27 13:06:00
author:     "Tom"
header-img: "img/2018-03-27_cover.jpg"
header-mask: 0.5
catalog: true
tags:
    - Android
    - Java
    - RxJava
    - quick-tip
---

This is a quick-tip for using RxJava 1.x to observe on the UI Thread.

In RxJava 2.x this is very easily achieved using the android scheduler:

```java
Observable.create(new ObservableOnSubscribe<Boolean>() {
    @Override
    public void subscribe(ObservableEmitter<Boolean> observableEmitter) throws Exception {
        ...
        observableEmitter.onNext(true);
        observableEmitter.onComplete();
    }
})
.subscribeOn(Schedulers.newThread())
.observeOn(AndroidSchedulers.mainThread())
.subscribe();
```

Here we create a new observable that returns a Boolean value when it completes. The observable subscribes on a new Thread (`Schedulers.newThread()`), and is observed on the main Android thread (`AndroidSchedulers.mainThread()`). This is a typical construction in RxJava 2.x if you need to perform actions on the UI thread after your observable has finished processing on a seperate thread, avoiding those pesky *Fatal signal 6 (SIGABRT)* errors in Android.

Unfortunately RxJava 1.x seemingly has no drop-in replacement for this. Of course if you're able to, use RxJava 2.x as it is far superior to its predecessor. But if that is not an option, then a solution posted by [Zapl to StackOverflow](https://stackoverflow.com/a/21256419/6136287) may be your answer.

The idea is to create an executor that runs on the Android main loop. If you haven't encountered executors I'd recommend [getting an overview to have a better understanding of async in Java](https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html). 

Our main thread executor will be:

```java
class UiThreadExecutor implements Executor {
    private final Handler mHandler = new Handler(Looper.getMainLooper());

    @Override
    public void execute(Runnable command) {
        mHandler.post(command);
    }
}
```

We can further add to this to make a true drop-in replacement for RxJava 2's `AndroidSchedulers.mainThread()`:

```java
class AndroidSchedulers {
    public static Scheduler mainThread() {
        return Schedulers.from(new UIThreadExecutor())
    }
}
```

Add these classes to your project and you're ready to go! You can now use `AndroidSchedulers.mainThread()` just as you would in an RxJava 2 project.

—— Tom 2017.10