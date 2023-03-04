---
title: "在旧芯片组使用 NVMe SSD 安装 Ubuntu"
date: 2023-03-04T10:19:00+11:00
draft: false
tags:
 - NVMe
 - Linux
 - H61
categories:
---

# 前情提要

手上有一个 H61 芯片组的主板的 NAS， 4 个 SATA 口和硬盘位。毕竟主板那么旧了，也没有 M.2 接口。最一开始是把 OS 安装到了 U 盘上，后来搬家，U 盘丢了，就说顺手升级一下系统盘。

于是乎，买了 PCI-E 转 M.2 接口的 adapter 和一块 NVMe M.2 的 WD SN570.

# 普通安装

直接做了个 Ubuntu Server 的安装盘，并直接在 NAS 上进行安装，发现找不到盘。于是，将 NVMe SSD 拆到 PC 上进行安装，同时将引导安装到 SSD 上。

当还是启动不来，一番搜索得出[^1]

> 可能是主板太老，BIOS 没有对 NVMe 盘的支持。但可以通过 Clovers 这种引导启动

## 安装 Clovers

使用这种方法的坏处就是，你需要一个 U 盘做启动盘，但系统还在 SSD 上， 相比于 U 盘做系统盘，速度好了很多。

但刷完引导发现引导里并没有 Ubuntu Server。

## 盘没有读取成功

进入到安装盘带的 Shell 里，运行 `fdisk --list` 和 `lshw` 发现根本找不到 NVMe 的设备。

通过各种调试，发现需要在 BIOS 把 PCI Express Port 从 `auto` 改成 `enabled`，能找到设备了，但引导依旧没用。

# 使用 SATA 盘做引导

在搜索 `linux pci-e nvme` 关键词的时候，看到一个 Linux 可以把 `/boot` 目录做到 SATA 硬盘中，系统 `/` 还是放在 NVMe SSD 中，

于是重新安装。

## 找不到盘

进入系统之后，显示找不到系统盘 (`/` 目录对应的盘)。

OS 是通过每个盘独立、唯一的 UUID 标识符加载的。因为系统盘加载失败，会回退到 `/boot` 里的 busybox, 查看 `/dev/` 下的盘符，没有找到 `NVMe` 系统盘。


### 解决方案

通过安装盘 `installer` 进入到 `shell` 里， 修改 `boot` 目录下 `grubenv` 文件

注意这里需要使用工具 `grub2-editenv` 修改，不能直接用过 vim 之类的软件修改。

`grub2-editenv grubenv set nvme_load=YES`

重启，就能顺利的进入系统了。


[^1]: [2020最新攻略：免刷BIOS 老电脑用NVMe固态盘装系统！](https://www.jianshu.com/p/5ee140b47642)