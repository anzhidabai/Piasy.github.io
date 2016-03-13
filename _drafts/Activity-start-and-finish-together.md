---
layout: post
title: 同时调用startActivity和finish会发生什么？
---

http://www.jianshu.com/p/72059201b10a

原来的activity会不会显示出来？

是否与startActivity和finish的调用顺序有关？

binder通信机制，是串行的还是并行的？

ActivityStack::xxxLocked，从名字推断是同步互斥的方法调用，但是它的locked机制是什么？

ActivityStack::mResumedActivity，ActivityStack::mPausingActivity，它们的含义，以及维护/修改过程？
