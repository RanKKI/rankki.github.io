---
title: "3DS 游戏安装"
date: 2023-03-23T18:26:10+11:00
draft: false
tags:
 - 3DS
categories:
---

收了台 New 3DS LL，成色十分新，价格也很便宜。到手第一件事就是「破解」，跟着文档[^3]，大概半小时就完成了。

在安装游戏的时候，发现内置的 FBI 安装速度在 `2MB/s`，小游戏还行。对于动辄就 2、3GB 的游戏，光安装就要 20+ 分钟。

经过一番查找[^1]，发现可以在电脑上直接安装到 SD卡 上。

使用这种方法安装真的很快，基本上跑满了 SD 卡的速度，80+MB/s

## 直接安装至 SD 卡

跟着教程[^4]可以 dump 出 `boot9.bin` 和 `movable.sed`，然后直接调用 Python 脚本即可完成安装

```shell
python3 custominstall.py -b boot9.bin -m movable.sed --sd /path/to/sd file.cia file2.cia
```

## CCI 文件

但问题来了，3DS 支持多种文件格式，但 `custominstall.py` 只能安装 `cia` 文件，因此可以用 `3dsconv`[^2] 将 `cci` 转成 `cia`，然后再使用 `custominstall.py` 安装到 SD 卡。


[^1]: [custom-install](https://github.com/ihaveamac/custom-install)
[^2]: [3dsconv](https://github.com/ihaveamac/3dsconv)
[^3]: [3ds.hacks](https://3ds.hacks.guide)
[^4]: [Dump 文件](https://ihaveamac.github.io/dump.html)