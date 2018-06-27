---
title: Module Name means What on the MVP
date: 2018-04-27 17:37:25
tags:
---

presenter(发出命令态):  中间者-对接V和P

​	操作(对数据的动作) + 修饰词(状态/时机/用户操作行为) + 宾语(UI组件)



view(结果呈现态)：视图-响应用户行为，呈现对应结果画面

​	操作(对数据的动作) + 结果(操作的结果)

​	或者，修饰词过去式(状态/时机/用户操作行为) + 宾语(UI组件)

​	或者，set/lay + 宾语(组件/数据对象<常使用在init初始化时>)



model(操作逻辑态): 处理员-分离出网络，磁盘等数据读写操作并提供接口将结果反馈给P层

​	操作(对数据的动作) + 宾语(某个数据实体/封装对象)+[from/on/by/ + 数据对象提供源]



延伸概念：

业务逻辑：[定义输入输出规则，走预定的流程](http://www.cnblogs.com/zhaoxiaolei/archive/2012/04/06/2434112.html)

​	DAO :[Data Access Objects](https://www.jianshu.com/p/798bac217046)属于业务逻辑层，但是是指的专门访问数据的逻辑接口