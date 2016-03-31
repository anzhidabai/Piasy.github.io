---
layout: post
title: 深入理解 RecyclerView 系列之二：ItemAnimator
tags:
    - 安卓开发
    - RecyclerView
---

本文继上篇 [ItemDecoration](http://blog.piasy.com/2016/03/26/Insight-Android-RecyclerView-ItemDecoration/){:target="_blank"} 之后，是深入理解 RecyclerView 系列的第二篇，关注于 ItemAnimator，主要是结合 [RecyclerView Animators](https://github.com/wasabeef/recyclerview-animators){:target="_blank"} 这个库的使用，总结如何自己编写自定义的 ItemAnimator。本文涉及到的完整代码可以在[ Github 获取](https://github.com/Piasy/AndroidPlayground/tree/master/recyclerviewadvanceddemo){:target="_blank"}。

可滑动，滑动时新加入数据不自动滚到最新（类似微信聊天消息列表），自动消失 + fade out 消失动效

## 先看看类结构
+ `DefaultItemAnimator` extends `SimpleItemAnimator` extends `RecyclerView.ItemAnimator`
+ `FadeInAnimator` extends `BaseItemAnimator` extends `SimpleItemAnimator` extends `RecyclerView.ItemAnimator`
+ `RecyclerView.ItemAnimator` 定义了一系列 API 用于开发 item view 的动效
  + `animateDisappearance`, `animateAppearance`, `animatePersistence`, `animateChange` 这4个 API 用来对 item view 进行动画显示
  + `recordPreLayoutInformation`, `recordPostLayoutInformation` 这2个 API 用来记录 item view 在 layout 前后的状态信息，这些信息封装在 `ItemHolderInfo` 或者其子类中，并将传递给上述4个动画API中，以便进行动画展示
  + `runPendingAnimations` 可以用来延迟动画到下一帧，此时就需要在上述4个 API 的实现中返回 `true`，并且自行记录延迟的动画信息，以便在下一帧时播放
  + `dispatchAnimationStarted` 和 `dispatchAnimationFinished` 是用来进行状态同步和事件通知的，子类必须在动画开始时调用 dispatchAnimationStarted，结束时调用 dispatchAnimationFinished，当然如果不展示动画，那就只需要直接调用 dispatchAnimationFinished
+ `SimpleItemAnimator` 则对 `RecyclerView.ItemAnimator` 的 API 进行了一次封装
  + 把父类定义的4个动画 API 转换为了 `animateRemove`, `animateAdd`, `animateMove`, `animateChange` 这4个，为什么这样？这一次封装就把对 preLayoutInfo 和 postLayoutInfo 的处理的公共代码封装了起来，把 ItemHolderInfo 转换为了 left, top, x, y 这样的位置信息，这样，大部分动画只需要根据位置变化信息的实现，就不必重复这样的转换代码了
  + 但是如果位置信息对于动画的展示不够，那就需要自己重写 `RecyclerView.ItemAnimator` 的相应动画 API 了
  + 同时也定义了一系列 `dispatch***` 和 `on***` API，用于进行事件回调
+ `DefaultItemAnimator` 是 RecyclerView 包中的一个默认实现，而 `BaseItemAnimator` 则是 RecyclerView Animators 库中 animator 的基类，它们都继承自 `SimpleItemAnimator`，两者具有很大相似性，只分析后者
  + 

## 脚注

