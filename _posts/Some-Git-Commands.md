---
title: Some Git Commands
date: 2018-03-26 12:11:20
tags:
---

# 1. 常用命令

1. git branch <新分支名> //创建新分支
2. git checkout <分支名> //切换到某分支
3. git checkout -b <新分支名> <基于分支> //先基于分支创建新分支，再切换到新分支(基于分支可以是从远端取过来的缓存分支,如：origin/master)
4. git push <远端名> <本地分支>：<远端分支> //推送本地提交到远端(如省掉本地分支，含义变为删除远端分支)
5. git pull <远端名> <远端分支>：<本地分支> //拉取远端分支更新到本地，再合并到本地分支(如果是要合并到当前分支，则可以省略”:<本地分支>”)
6. git fetch <远端名> <远端分支> //如果省略远端分支则会取回所有远端分支(所有取回的分支则会以”origin/分支”命名)
7. git merge <本地缓存分支> //本地缓存分支形如”origin/分支”
8. git rebase <本地缓存分支> //本地缓存分支形如”origin/分支”, 含义同git merge但是不保留merge的痕迹

# 2. 关联仓库

有两种方式：
A) git clone <本地文件夹>
B) git init
git remote add <远端名>