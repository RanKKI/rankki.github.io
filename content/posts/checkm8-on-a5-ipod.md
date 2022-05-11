---
title: "在 iPod A5 设备上使用 Checkm8 漏洞"
date: 2022-05-11T10:16:07+08:00
tags:
 - Arduino
 - checkm8
categories:
 - 技术
---

> 漏洞的详细分析可以看 [Checkm8 漏洞研究](https://xuanxuanblingbling.github.io/ios/2020/07/10/checkm8/) 和 [iPhone史诗级漏洞checkm8攻击原理浅析](https://zhuanlan.zhihu.com/p/87456653)，感觉讲的很可以，但因为我不是搞二进制的，不是很能看懂。

# 前言

手上有一台 A5，有 iCloud 锁的 iPod，一直没用。2019 年的时候知道了 ipwndfu 和 check1rn 两个项目，本以为能解决这台 iPod，但 CPU 处理器 `S5L8942` 并没有支持，于是就搁置了。

后来得知了一个新项目`checkm8-a5`, 用于处理 A5 的设备，但需要准备 Arduino 和 USB Host shield，根据 [Issue#15](https://github.com/a1exdandy/checkm8-a5/issues/15) 的介绍。这个芯片很特别，但电脑系统的行为会和预期的不一样，所以导致漏洞的利用会失败。

 > it is only for arduino because the A5 chipset is very specific, so an OS like macOS or Windows (or even Linux) will mess things up because we need exact control over the USB Port. [来源](https://github.com/a1exdandy/checkm8-a5/issues/15#issuecomment-873496921)

# 设备

在网购了 Arduino 和 USB Host shield 之后，等待数日之后便开始尝试了。

设备不被 USB Host shield 识别，在大量的搜索后，发现需要用电烙铁焊掉三个触电，使电路连通。

![usb_host_shield_fix_small.jpg](https://s2.loli.net/2022/05/11/VoL8YRWOGPhmjyK.jpg)

# Checkm8

将项目 `checkm8-a5` 下载下来并刷入 Arduino，连接好 USB Host shield，并将 iPod 进入 DFU 模式，链接上即可在串口看到日志，等待一段时间就可以进入 Bypass 阶段。

# Bypass

越过 iCloud 锁需要用到另一个软件 `Silver`，作者是 `appletech752`。 由于闭源产品加上这个性质，我试用了一台旧的 MacBook 去运行，以免对主力机有影响。

软件的 UI 十分简单，但 `checkm8` 漏洞的利用不一定成功，因此需要重复无数次，直到成功。

# 后续

bypass 的操作只是针对本机的，但对于 Apple 来讲，该设备还是属于 `锁` 的状态，因此刷机会导致 iCloud 锁依旧存在。不过应该没有人会在给 A5 的设备升级刷机了吧。

# Links
 - [ipwndfu](https://github.com/axi0mX/ipwndfu)
 - [checkm8-arduino](https://github.com/DSecurity/checkm8-arduino#building)
 - [checkm8-a5](https://github.com/a1exdandy/checkm8-a5)
 - [USB_Host_Shield_2.0](https://github.com/felis/USB_Host_Shield_2.0)
 - [USB Host Shield 修复缺陷](https://esp8266-notes.blogspot.com/2017/08/defective-arduino-usb-host-shield-boards.html?m=1)
 - [Silver](https://www.appletech752.com/downloads.html)

