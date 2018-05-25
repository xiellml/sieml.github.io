---
title: Adb Dumpsys
date: 2018-03-26 12:12:47
tags:
---

# 1. 查看帮助

adb shell dumpsys -h

Activity manager dump options:
[-a][-c] [-p package][-h] [cmd] …//省略处为可以跟的参数(有-a/-c/-p)
cmd may be one of://代表可以跟的命令
a[ctivities]: activity stack state//activity的栈信息
r[recents]: recent activities state//最新的activity的信息
b[roadcasts][PACKAGE_NAME] [history [-s]]: broadcast state
i[ntents][PACKAGE_NAME]: pending intent state
p[rocesses][PACKAGE_NAME]: process state
o[om]: out of memory management
perm[issions]: URI permission grant state
prov[iders] [COMP_SPEC …]: content provider state
provider [COMP_SPEC]: provider client-side state
s[ervices] [COMP_SPEC …]: service state
as[sociations]: tracked app associations
service [COMP_SPEC]: service client-side state
package [PACKAGE_NAME]: all state related to given package
all: dump all activities
top: dump the top activity
write: write all pending state to storage
track-associations: enable association tracking
untrack-associations: disable and clear association tracking
cmd may also be a COMP_SPEC to dump activities.
COMP_SPEC may be a component name (com.foo/.myApp),
a partial substring in a component name, a
hex object identifier.
-a: include all available server state.
-c: include client state.
-p: limit output to given package.

# 2. 常见命令

1. adb shell dumpsys activity top
   显示处于前台activity(活动状态的栈顶)的信息。
2. adb shell dumpsys activity | grep “xxx”
   过滤目前的所有的activity栈信息。
3. adb shell dumpsys activity -p “包名”
   过滤某个包的信息。

dumpsys命令很强大，可以慢慢探索。