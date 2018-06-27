---
title: NodeJs and Hexo Theme
date: 2018-04-27 17:53:54
tags: script
---



0. Nodejs: an open-source, cross-platform JavaScript run-time environment that executes JavaScript code server-side.
1. Swig: A simple, powerful, and extendable JavaScript Template Engine
   模板引擎：中间件，使用户界面与业务数据（内容）分离而产生的，生成特定格式的文档（如HTML）
2. Grulp: an open-source JavaScript toolkit, used as a streaming build system in front-end web development.
   使用： https://www.cnblogs.com/Tom-yi/p/8036730.html



3. Deno js框架：

   运行于v8, 使用了js/py/c/c++/js(deno1还用了golang)等语言，工具构建使用的是*depot_tools*，Ninja



1. ejs:
  （写一个自己的主题）http://www.showonne.com/(http://www.showonne.com/2016/07/27/%E5%86%99%E4%B8%80%E4%B8%AA%E8%87%AA%E5%B7%B1%E7%9A%84Hexo%E4%B8%BB%E9%A2%98/)
  （如何写一个自己的主题）https://www.jianshu.com/p/a142eb105279
  （从零开始制作 Hexo 主题）http://www.ahonn.me/2016/12/15/create-a-hexo-theme-from-scratch/
  (美化NeXT主题)https://www.jianshu.com/p/f054333ac9e6
2. swig（nodejs模板引擎）:(注意：与Simplified Wrapper and Interface Generator有所区别，指和c/c++跟任何脚本语言相集成。当你有一个C/C++代码库需要被多种语言调用时，这将是个非常不错的选择。http://wiki.jikexueyuan.com/project/interpy-zh/c_extensions/swig.html)
  !!!(Hexo模板主题开发 入门念叨)http://www.zzfly.net/hexo-theme/
  (Hexo 主题制作指南)http://www.360doc.com/content/16/0913/16/33651124_590545274.shtml
3. 手动编写博客和样式：
  (/index.html + Cayman theme )https://github.com/jasonlong/cayman-theme





0-1. github pages(http://wiki.jikexueyuan.com/project/github-pages-basics/github-page.html)

GitHub Pages 是什么
GitHub Pages 是通过我们网站托管和发布的公开静态网页。

你可以通过 Automatic Page Generator 在线创建和发布 GitHub Pages。如果你更喜欢本地操作，你可以使用 Mac 或者 Windows 平台的 GitHub App，或者使用 命令行。

Pages 是通过 HTTP 服务的，不是 HTTPS，所以你不应该使用它处理敏感的事务，像发送密码或者信用卡号码。

警告：GitHub Pages 网站是在互联网上公开的，即使它们所在的库是私有的。如果你有敏感的数据在 Page 库，你应该在发布之前删除它。

想要获取更多信息，可以查看 Github Pages 网页，或者这里的 相关帮助文档。

0-2. 了解githubPages+hexo搭建博客的原理(https://blog.csdn.net/sunshine940326/article/details/57413678)