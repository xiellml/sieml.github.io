---
title: View Hierarchy and View Events
date: 2018-03-27 11:46:20
tags: view&event
---

##### 事件概述(窗口机制)：

事件由硬件感应,会被Android系统(基于Linux)存储在/dev/input/各个目录下(event0,event1..),然后由WindowManagerService捕获用户输入的事件(按键/触摸),WMS通过共享内存和管道的方式传递给ViewRoot(ViewRoot extends Handler),ViewRoot再dispatch给Application的View.

当有事件从硬件设备输入时,system_service端检测到事件发生时,通过管道通知ViewRoot事件发生,此时ViewRoot再去内存中读取这个事件信息.WindowManagerService->ViewRoot(ViewRoot extends Handler implements ViewParent)->Activity(->PhoneWindow直接执行的是mDecor的分发分发)->DecorView(PhoneWindow的子类);另:(WindowManager为接口,定义管理PhoneWindow的接口)

##### Activity之setContentView

DecorView继承于FrameLayout，然后它有一个子view即LinearLayout，方向为竖直方向，其内有两个FrameLayout，上面的FrameLayout即为TitleBar之类的，下面的FrameLayout即为我们的ContentView，所谓的setContentView就是往这个FrameLayout里面添加我们的布局View的！

### 事件的派发过程：

WindowManagerService -> Activity -> DecorView(PhoneWindow的子类) -> ViewGroup -> View

简述为: Activity -> ViewGroup -> View

### 事件处理过程：

1. 事件截断处理的View:

   onInterceptTouchEvent() return true;(表示不再向子View传递事件,否则传递给子View[当其最里面的子View 的dispatchTouchEvent()返回为false则继续原路返回往外传递])

2. 事件分发处理的View:

   dispatchTouchEvent() return true;(表示当前View要处理[返回true是告诉父View被 我/我的子View 处理了,处理的过程是 我/我的子View 调用onTouchEvent()而且返回true处理的],反之则自己不处理分发给子View)

3. 事件消费处理的View:

   onTouchEvent() return true;(表示事件的终点,否则则不是终点.注: 如果都不处理最后会由Activity返回false作为最后一个调用者,不可点击的View只能返回false)

**伪代码逻辑:**

1. if(ViewGroup.onInterceptTouchEvent()==false ) {

    View.dispatchTouchEvent()

   }

2. if(View.onTouchEvent()==true ) {

    View.dispatchTouchEvent()->

    ViewGroup.dispatchTouchEvent()->

    Activity.dispatchTouchEvent()

   }

3. if(View.onTouchEvent()==false ) {

    ViewGroup.onTouchEvent()->

    Activity.onTouchEvent()

   }

4. if(ViewGroup.onInterceptTouchEvent()==true ) {

   ViewGroup.onTouchEvent()}

   只进行第一次判断为true,

   下次用户再点击不会再调用ViewGroup.onInterceptTouchEvent(),而是直接调用ViewGroup.onTouchEvent(),

   因为多于的判断没有意义(其拦截函数返回都是true)

5. if(ViewGroup.onTouchEvent()==true ){

    if(View.onTouchEvent()==true&&action == ACTION_DOWN) {

    if(某一刻 ViewGroup.onInterceptTouchEvent()==true)

    那么下次子View就会收到一个ACTION_CANCEL

    }

   }

资料：

- [Android完美解析setContentView 你真的理解setContentView吗？](https://blog.csdn.net/nugongahou110/article/details/49662211)
- [android的窗口机制分析——事件处理](http://blog.csdn.net/windskier/article/details/6966264)
- [Android中与ViewRoot相关的一些概念](https://blog.csdn.net/gc_gongchao/article/details/45798221)
- [《深入理解Android 卷III》第四章 深入理解WindowManagerService](http://blog.csdn.net/innost/article/details/47660193)
- [使用WindowManager添加View——悬浮窗口的基本原理](http://www.cnblogs.com/cpacm/p/4087690.html?utm_source=tuicool&utm_medium=referral)