---
layout: post
title: 这可能是目前最详细的安卓task, launchMode, intent flag测试分析与总结了
tags:
    - 安卓开发
    - PoC
---

最近一直在补充framework以及更深入的安卓开发知识，看到[老罗的博客](http://blog.csdn.net/luoshengyang/article/details/6714543)以及[developer文档](http://developer.android.com/guide/components/tasks-and-back-stack.html)关于task, launchMode, intent flag的分析说明之后，不禁想要自己动手测试一下，验证他们所说的是否属实，个人认为当属目前最全面的测试与总结了（欢迎补充与质疑），完整代码可以在[Github](https://github.com/Piasy/AndroidPlayground/tree/master/taskdemo)获取。

## TL; DR
+ 同一task内的activity可以是来自不同进程的activity
+ 栈内的activity不会重新排序，只能push或者pop
+ standard模式允许多实例，可以在不同的task
+ singleTask的activity只会存在一个实例
+ singleTask的activity如果设置了独立的taskAffinity属性值，启动时就会在新的task中，否则会在已有task中
+ singleTask的activity启动时，它会在目标task（新的task或者已有task）中查看是否已经存在相应的activity实例，如果存在，就会把位于这个activity实例上面的activity全部销毁（pop, destroy）掉，即最终这个activity实例会位于任务的堆栈顶端中

## task
task是一个从用户角度出发的概念，它是一些activity的组合，它们组合起来是为了让用户完成某一件工作（或者说操作）。

task内的activity们以栈的形式组织起来，也就是back stack了。栈内的activity不会重新排序，只能push或者pop。栈内的activity可以来自不同的app，因此可以是运行在不同的进程，但是它们都属于同一个task内。

安卓系统是实时多task系统，用户可以随意在多个task之间切换。当一个task的栈内所有activity都pop之后，task也就销毁了。有时系统为了回收内存，会销毁activity，但是task不会销毁。

activity在manifest中有launchMode选项，可以配置其启动的模式。标准模式下，允许一个activity同时存在多个实例，既可以在同一个task，也可以在不同task。manifest可以指定launchMode，intent可以指定intent flag，前者供被启动者使用，后者供启动者使用，同时使用时后者会覆盖前者。

## launchMode

+ standard，默认模式，允许多实例
+ singleTop，相比于standard，有新的启动请求时，只有在目标activity处于当前栈顶时，才会调用`onNewIntent()`而不创建新实例，其他情况都和standard一致
+ singleTask，（这段摘自[老罗的博客](http://blog.csdn.net/luoshengyang/article/details/6714543)）设置了"singleTask"启动模式的Activity，它在启动的时候，会先在系统中查找属性值affinity等于它的属性值taskAffinity的任务存在；如果存在这样的任务，它就会在这个任务中启动，否则就会在新任务中启动。因此，如果我们想要设置了"singleTask"启动模式的Activity在新的任务中启动，就要为它设置一个独立的taskAffinity属性值。如果设置了"singleTask"启动模式的Activity不是在新的任务中启动时，它会在已有的任务中查看是否已经存在相应的Activity实例，如果存在，就会把位于这个Activity实例上面的Activity全部结束掉，即最终这个Activity实例会位于任务的堆栈顶端中。
+ singleInstance，和singleTask相比，不同点在于singleInstance activity所在的task只会有这一个activity
+ 返回导航：singleTask和singleInstance启动的activity，尽管可能不在同一个task，但是仍然会回到原来的activity；但是singleTask可能会存在[back stack“拼接”的情况](http://developer.android.com/images/fundamentals/diagram_backstack_singletask_multiactivity.png)

## launchMode验证测试
主要测试的是singleTask这一属性，其他的文档描述都比较清楚，也不容易产生误解。两组实验，不设置taskAffinity和设置taskAffinity时，singleTask的行为。对比两者“最近任务”，dumpsys，logcat，返回导航的差异。

### 测试1

> 不设置taskAffinity，MainActivity -> SingleTaskFirstActivity -> SimpleActivity -> SingleTaskSecondActivity -> 选择图片Intent，再一路返回。

最近任务内只能看到一个TaskDemo的任务，dumpsys的结果如下：

<img src="/img/201603/test_result_no_task_affinity_dumpsys.jpg" alt="dumpsys的结果">

所有的activity均在同一个task内，验证了老罗的结论。另外注意标红的两处，它们的ProcessRecord对象不同，ImageGallery和SingleTaskActivity处于不同的进程，也验证了前文所述“可以运行在不同的进程”。

logcat日志如下：

<img src="/img/201603/test_result_no_task_affinity_logcat.png" alt="logcat的结果">

一路返回依次回到SingleTaskSecondActivity, SimpleActivity, SingleTaskFirstActivity,MainActivity, 桌面。 

### 测试2

> 设置SingleTaskFirstActivity的taskAffinity为单独的值，MainActivity -> SingleTaskFirstActivity。

通过最近任务可以看到，有了两个TaskDemo的任务。而dumpsys结果如下：

<img src="/img/201603/test_result_task_affinity_dumpsys.jpg" alt="dumpsys的结果2">

可以看到，MainActivity和SingleTaskFirstActivity运行在了两个不同的task里面（栈底为MainActivity的t225, 栈底为SingleTaskFirstActivity的t226），但是他们仍属于同一进程。

> 我们接着启动SimpleActivity，再通过最近任务切换回t225，并且也启动SimpleActivity，这时通过dumpsys结果如下：

<img src="/img/201603/test_result_multiple_instance_dumpsys.jpg" alt="dumpsys的结果3">

SimpleActivity同时存在两个实例，他们的hashCode是不同的，目前处于resumed状态的是在t225中的实例。

而通过查看logcat：

<img src="/img/201603/test_result_multiple_instance_logcat.png" alt="logcat的结果2">

我们同样发现SimpleActivity被创建了两个实例。

> 此时我们通过最近任务切换回t226，并在此启动SingleTaskSecondActivity，查看dumpsys，然后再切换回t225也启动SingleTaskSecondActivity，查看dumpsys和logcat：

<img src="/img/201603/test_result_multiple_instance_dumpsys2.jpg" alt="dumpsys的结果4">
<img src="/img/201603/test_result_multiple_instance_dumpsys3.jpg" alt="dumpsys的结果5">
<img src="/img/201603/test_result_multiple_instance_logcat2.png" alt="logcat的结果3">

我们可以看到尽管第一次启动SingleTaskSecondActivity是在t226，但是SingleTaskSecondActivity实例却运行在t225中，为什么呢？因为SingleTaskSecondActivity并没有设置taskAffinity属性，所以它和MainActivity将会运行在同一个task中！和老罗的分析一致。而第二次启动时，并未创建新的SingleTaskSecondActivity实例，而是调用了它的onNewIntent方法，和文档描述一致。

现在我们测试一下返回导航，依次按下返回键，观察每次回到的activity，以及每次dumpsys的结果，以及logcat的结果：

第一次回到了t225的SimpleActivity，dumpsys：
<img src="/img/201603/test_result_multiple_instance_back_dumpsys.jpg" alt="第1次dumpsys的结果">

第二次回到了t225的MainActivity，dumpsys：
<img src="/img/201603/test_result_multiple_instance_back_dumpsys2.jpg" alt="第2次dumpsys的结果">

第三次回到了t226的SimpleActivity，dumpsys：
<img src="/img/201603/test_result_multiple_instance_back_dumpsys3.jpg" alt="第3次dumpsys的结果">

第四次回到了t226的SingleTaskFirstActivity，dumpsys：
<img src="/img/201603/test_result_multiple_instance_back_dumpsys4.jpg" alt="第4次dumpsys的结果">

最后一次返回就回到了桌面。

logcat日志如下：
<img src="/img/201603/test_result_multiple_instance_back_logcat.jpg" alt="依次返回logcat的结果">

从上面的返回路径来看，确实验证了[developer文档上关于back stack“拼接”的情况](http://developer.android.com/images/fundamentals/diagram_backstack_singletask_multiactivity.png)的描述，即：并不是按照most recent first的顺序返回的，而是优先把一个task的back stack（t225） pop完毕之后，再pop下一个task的back stack（t226）。

### 测试3
> 基于测试2的版本，MainActivity -> SingleTaskFirstActivity -> SimpleActivity，再切换回MainActivity，再启动SingleTaskFirstActivity。

查看最近任务，仍然有两个task，但是查看dumpsys以及logcat：
<img src="/img/201603/test_result_destroy_top_dumpsys.jpg" alt="测试3 dumpsys的结果">
<img src="/img/201603/test_result_destroy_top_logcat.png" alt="测试3 logcat的结果">

我们可以看到，第一次启动的SimpleActivity就被pop（destroy）了，从而把未在栈顶的SingleTaskFirstActivity提到了栈顶。这也验证了back stack内的activity不会重新排序，最会pop和push的事实。

## Intent flag
只要搞清楚了launchMode中容易产生误解的地方，对于intent flag来说，只需要记住一点即可：intent flag可以覆盖launchMode的设置。为了避免内容过于冗长，本文就不对intent flag进行详细的测试分析了。

## 总结
+ 同一task内的activity可以是来自不同进程的activity
+ 栈内的activity不会重新排序，只能push或者pop
+ standard模式允许多实例，可以在不同的task
+ singleTask的activity只会存在一个实例
+ singleTask的activity如果设置了独立的taskAffinity属性值，启动时就会在新的task中，否则会在已有task中
+ singleTask的activity启动时，它会在目标task（新的task或者已有task）中查看是否已经存在相应的activity实例，如果存在，就会把位于这个activity实例上面的activity全部销毁（pop, destroy）掉，即最终这个activity实例会位于任务的堆栈顶端中
