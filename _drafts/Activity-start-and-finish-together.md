---
layout: post
title: 当startActivity遇上finish
tags:
    - Android Framework
---

## Activity finish过程
分析安卓[6.0.1_r10源码](http://androidxref.com/6.0.1_r10/){:target="_blank"}

+ Activity::finish() =>
+ Activity::finish(boolean) =>
+ ActivityManagerProxy::finishActivity() = binder =>
+ ActivityManagerService::finishActivity() =>
+ ActivityStack::requestFinishActivityLocked =>
+ ActivityStack::finishActivityLocked =>
+ ActivityStack::startPausingLocked =>
+ ApplicationThreadProxy::schedulePauseActivity = binder =>
+ ActivityThread$H::handleMessage =>
+ ActivityThread::handlePauseActivity =>
+ ActivityThread::performPauseActivity =>  调用onSaveInstanceState，onPause等生命周期函数
  + --> Instrumentation::callActivityOnPause 
  + --> Activity::onPause
+ ActivityManagerProxy::activityPaused = binder =>
+ ActivityManagerService::activityPaused() =>
+ ActivityStack::activityPausedLocked =>
+ ActivityStack::completePauseLocked =>
+ ActivityStack::finishCurrentActivityLocked =>
+ ActivityStack::destroyActivityLocked =>
+ ApplicationThreadProxy::scheduleDestroyActivity = binder =>
+ ActivityThread$H::handleMessage =>
+ ActivityThread::handleDestroyActivity =>
+ ActivityThread::performPauseActivity =>  调用onDestroy等生命周期函数
+ ActivityManagerProxy::activityDestroyed = binder =>
+ ActivityManagerService::activityDestroyed() =>
+ ActivityStack::activityDestroyedLocked =>
+ ActivityStack::resumeTopActivityLocked =>



http://www.jianshu.com/p/72059201b10a

原来的activity会不会显示出来？




是否与startActivity和finish的调用顺序有关？

binder通信机制，是串行的还是并行的？

ActivityStack::xxxLocked，从名字推断是同步互斥的方法调用，但是它的locked机制是什么？

ActivityStack::mResumedActivity，ActivityStack::mPausingActivity，它们的含义，以及维护/修改过程？

ActivityStack::mResumedActivity，当前正在运行的activity（已resume尚未pause）

ActivityStack::mPausingActivity，pause activity时，在启动新的activity之前，用来引用被pause的activity
