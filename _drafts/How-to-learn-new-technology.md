---
layout: post
title: 如何学习新技术？
tags:
    - meta
    - 学习
---

<img src="/img/201603/how-to-learn-new-technology.png" alt="how-to-learn-new-technology" style="height:400px">

> 摘自：[怎样快速学习一门新技术](https://ruby-china.org/topics/19578)

## 是什么？
不要因为所谓“火了”就盲目学习，先弄清楚是什么，可以通过官方定义，或者百科。

再了解有哪些同类技术。和同类技术的对比优劣是什么？

这项技术的组成部分？

## 为什么出现？
这项技术解决了什么问题？没有这项技术之前是怎么解决这些问题的？

## 怎么做？
官方/第三方示例 => 自己编写demo => 看优秀的开源代码/最佳实践/官方建议文档 => 项目实践。

## 分享
深入理解其工作原理，总结最佳实践等，通过博客或者沙龙方式分享。

## 以BUCK为例
快速打包工具。gradle打包太慢，主要是gradle的初始化需要较多时间，另外aapt等sdk工具的原版效率较低，所以打包慢。另有LayoutCast，基于ART的运行时替换技术，只能应用于5.0以上。

包括了对aapt等工具的改进、瘦身、加速。累进式打包（差异检测）。

就是因为gradle太慢了，为了解救安卓开发者。

+ hello world：安装BUCK，下载依赖jar/aar文件，编写BUCK rule
+ BUCK的配置过程存在自动化可能 => OkBuck
+ OkBuck开源项目，文档，改进

## Data binding

## React Native
一个横跨iOS与安卓的库，让iOS和安卓APP可以几乎公用同一套JS代码，但是却使用Native控件，性能更好。其实类似的技术更早国内就有出现，但奈何FB就是NB。

暂且不论React的优势，React Native以跨平台共用JS代码、保持Native性能为主要特点。原理是什么呢？

React则以虚拟dom高性能差异渲染、状态即视图、JFX等著称（实在了解不深，还要逐步加深认识）。



## React

## Kotlin

## Swift

## Golang
