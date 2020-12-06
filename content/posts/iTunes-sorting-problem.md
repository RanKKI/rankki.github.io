---
title: iTunes 中日英歌曲排序问题
date: 2020-10-18 14:20:02
tags:
 - 脚本
 - iTunes
categories:
 - 技术
---

因为个人原因， 我基本是下载歌曲并整理导入 iTunes，然后同步到各个设备上。 同时，我的设备语言是了英文，这就导致，所有英文歌曲按照 A-Z 排序，然后所有日文和中文歌曲在 # 的位置里。

搜索了下相关问题，知道 iTunes 有一个标签， sorting name， 即 iTunes 可以按照这个变量进行排序，比如将中文转换成拼音，或者日文转成罗马音。 这样就可以正常排序了。 

解决方法是有了，但曲库都是以几百首、几千上万计算的， 不可能一个一个的去改。 作为程序员，要遵守

> 宁愿写 1，2个小时的脚本，也不愿意花半小时手动完成任务

当然，写脚本是为了以后，新增歌曲的时候，可以自动化的完成编辑，可以省很多时间。

macOS (从 10.5 开始）提供了接口，叫做 `ScriptingBridge`, 位于

> /System/Library/Frameworks/Python.framework/Versions/Current/Extras/lib/python/PyObjC/

可以通过这个 bridge 获取 iTunes 的文件并对信息进行更改， 随后可以使用其他库或者脚本，转成拼音（或者罗马音）

<script src="https://gist.github.com/RanKKI/b3de56d43339edf8258763801d9091c5.js"></script>