---
title: Cookie Basic Introduction
date: 2018-04-27 18:09:44
tags:
---



1. 服务器生成cookie，以键值对Set-Cookie:k1=v1;k2=v2[...][;expires=DATA][;domain=DOMAIN][;path=PATH][;secure]（注意secure仅用于https,只是一个key标记，没有value值）发送给客户端，
    以便下次使用时服务器会认识客户端（同一个人），就不需要再次发送验证请求了（譬如登录）。

2. 客户端保存上次从服务器获得的cookie, 下次请求就将其所需信息带上到header中给Server验证


参考：
https://www.cnblogs.com/zhujiabin/p/6207913.html
http://www.360doc.com/content/17/0224/11/6695445_631622883.shtml
https://bbs.csdn.net/topics/391934186