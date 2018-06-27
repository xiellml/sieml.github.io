---
title: Bounce View with Multi-gesture
date: 2018-04-27 17:16:53
tags: views
---



找了些demo，大都能实现回弹效果，但多手势可能部分不支持或者不够完善：

[OverScroll-Everywhere](https://github.com/Mixiaoxiao/OverScroll-Everywhere)

​	回弹和惯性越界
	视图类类似果冻scrollscale
	支持多点multigesture

[SmoothRefreshLayout](https://github.com/dkzwm/SmoothRefreshLayout)
	视图类类似果冻scrollscale
	拉伸回弹overscroll
	手势刷新refresh
	不是通过多点触控实multigesture
	
[PullRefreshLayout][https://github.com/genius158/PullRefreshLayout]
	同SmoothRefreshLayout，但是是拦截了多点触控实现multigesture
	
[SmartRefreshLayout](https://github.com/scwang90/SmartRefreshLayout)
	同SmoothRefreshLayout，侧重刷新炫酷效果，可能不是通过多点触控实multigesture
	
[overscroll-decor和OverScrollView](https://github.com/EverythingMe)
	overscroll-decor所有视图的回弹&不支持多点
	OverScrollView支持多点
	
OverScroll-Everywhere这个比较好用的，可以学习一下多手势事件流程，完！