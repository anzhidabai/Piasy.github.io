---
layout: post
title: 安卓通知栏消息点击启动Activity时“Intent flag残留”问题
---

今天开发的过程中偶遇一个系统的bug：响应通知栏消息，启动一个新的Activity，之前代码写得有问题，为intent设置了`Intent.FLAG_ACTIVITY_CLEAR_TASK`这个flag，删除这行代码重新安装之后，竟然发现修改不起作用，启动Activity的行为依然是清除了之前所有的Activity，疑惑之余，对这个问题进行了进一步的测试，总结成此文。

## 功能需求
首先说一下响应通知栏消息的需求：如果APP之前启动过目标Activity并且没有finish，那点击通知栏消息，不能导致有多个目标Activity在栈中，也就是说，点击通知栏消息启动目标Activity之后，再点返回键，不能又回到之前的目标Activity实例中。

## 实现方式
因为之前没有考虑到这一问题，所以最初的代码不能满足上述功能需求，会启动多个目标Activity，后来为了满足这个功能需求，尝试了各种flag，当时就是因为这个系统的bug，尝试各种flag都达不到效果，颇为头疼，当时没有注意到这是系统的bug，所以莫名其妙的就设置了`Intent.FLAG_ACTIVITY_CLEAR_TASK`这个flag。

是的，我现在已经不知道当时为什么要用这个flag了，这个flag根本不可能符合需求呀！结束新启动的目标Activity后必然导致整个APP结束，真是给自己立了个flag。

今天为了解决会导致整个APP结束的bug，再次遭遇“Intent flag残留”这一系统bug，并且意识到了它的存在。然后我就这个问题进行了较为全面的测试，并且针对功能需求，找到了一个折衷解决方案。

## PoC
[PoC工程地址](https://github.com/Piasy/AndroidPlayground/tree/d19354008e398cad41377015131bff0976c29e61/notificationtest)

工程包含两个Activity：`MainActivity`和`NotifyActivity`，前者是LaunchActivity，包含一个Button，点击后，在通知栏发送一个通知，该通知点击后将启动`NotifyActivity`，与此同时，立即启动一个`NotifyActivity`。这两个Activity在`AndroidManifest.xml`文件中均未设置任何`launchMode`，两处启动`NotifyActivity`的Intent（立即启动，点击通知启动）均未设置任何flag。

测试过程为：安装PoC工程app，点击Button，这时将跳转到`NotifyActivity`，并且通知栏会出现通知，按home键回到桌面，然后点击通知栏消息，这时将再次启动`NotifyActivity`；然后按返回键，观察app行为。

### 先用Nexus 6测试
手机环境：Nexus 6，CM系统12.1-20151106-NIGHTLY-shamu，安卓系统5.1.1。

首先，初始状态下，第一次按下返回键，将回到首先启动的`NotifyActivity`，再次按下返回键，将回到`MainActivity`，再次按下返回键，将回到桌面。

现在，在创建通知栏通知的代码中，为启动`NotifyActivity`的Intent设置`Intent.FLAG_ACTIVITY_CLEAR_TASK`这个flag，即取消[MainActivity.java的43行](https://github.com/Piasy/AndroidPlayground/blob/d19354008e398cad41377015131bff0976c29e61/notificationtest%2Fsrc%2Fmain%2Fjava%2Fcom%2Fgithub%2Fpiasy%2Fnotificationtest%2FMainActivity.java#L43)注释。这时app的行为不变！是的，不变！不过也没什么好震惊的，上文已经描述这个bug了。

这里我直接把下一步的测试方法说出来，虽然这种方法能够使代码的更改生效，但这并不能作为解决方法，因为这种方法是：先卸载再装新版/重启手机。

进行先卸载再装新版/重启手机操作后，app的行为是：第一次按下返回键，就直接回到了桌面，这就是`Intent.FLAG_ACTIVITY_CLEAR_TASK`应该的作用。

好了，反过来试一下，把取消注释的那行代码再次注释，然后再测一下：行为不变，第一次按下返回键，就直接回到了桌面。

再次重复先卸载再装新版/重启手机操作后，app的行为恢复了没有flag的行为：先回到老的`NotifyActivity`，再回到`MainActivity`，再回到桌面。

当然，为了验证确实代码改动生效了，使用反编译工具JEB，查看对apk文件反编译的代码，确实改了，也尝试过直接用adb命令安装改了代码之后打包的apk文件，都能确认这个bug。

期间还尝试过
+  杀死进程（在“设置-应用”中强制停止，而不是在最近任务中操作），没用
+  清除数据，没用

### 再用HTC desire 816t测试
手机环境：HTC desire 816t，ROM版本2.34.1403.3，安卓系统5.0.2。

重复上述过程，确认了bug。

### 再用红米2测试
手机环境：HM 2LTE-CT，MIUI版本7.0.11.0，安卓系统版本4.4.4。

重复上述过程，确认了bug。

### 再用华为荣耀3C测试
手机环境：HONOR H30-L01，EMUI版本2.3，安卓系统版本4.4.2。

重复上述过程，有了新的变化。

首先，修改代码之后覆盖安装依然无效，但是，卸载之后重新安装，这次变成点击通知栏消息无法启动到新的`NotifyActivity`了。重启后将达到预期的效果：点击通知栏消息将启动新的`NotifyActivity`，第一次点击返回键，将回到桌面。

### 最后使用Nexus 6模拟器，6.0系统
重复上述过程，和Nexus 6真机，htc行为一致，确认了bug。

## 解决办法
如果app没有上线，那就不会有任何问题，但是如果已经上线并且有很多用户了，那显然不能寄希望于用户重启手机，很多人甚至从不重启手机。

折衷解决办法就是，创建一个新的“中继Activity”，新版点击通知栏消息时，启动“中继Activity”，且不要设置任何flag，在“中继Activity”中启动目标Activity，并设置相应flag，由于新的“中继Activity”在老版本用户的手机中从未出现过，所以不会受到这个bug的影响，已在线上项目实测通过，就不在PoC项目中演示了。

## 小结
安卓生态下的机型、系统、ROM不计其数，我就不做更多的测试了。

+  首先第一点，4.4.2有这个bug，卸载重装后将导致点击通知栏消息无法启动相应Activity，只能重启。
+  从4.4.4起，这个bug得到了一定程度的修复，卸载重装将会得到预期效果。
+  最新6.0系统依然没有解决这个bug。
+  关于原因，可以参考下[Android issue 61850](https://code.google.com/p/android/issues/detail?id=61850) #7楼的回答，这个我也还没有研究明白，待深入。
+  此外，上文提到了未对两个Activity在`AndroidManifest.xml`中声明任何`launchMode`，经测试，`launchMode`没有上述bug，修改之后覆盖安装，立即生效。