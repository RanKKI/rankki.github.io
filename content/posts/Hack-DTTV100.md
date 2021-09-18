---
title: Hack DTTV100 机顶盒
date: 2018-12-27 17:36:21
tags:
 - MITM
categories:
 - 技术
---

### 背景

联通给了个 IPTV 机顶盒， 看了下是 Android 4.4 系统的，在 Google 一圈之后，发现有人是通过硬件 hack 拿到权限的。但实际上这个机顶盒里面自带一个商店，需要下载， 那只要来个中间人替换应用市场的 apk 就行了。然后正好手上有个树莓派，（虽然说不用也行。 ）

### 开始 hack

#### 准备硬件

 - 树莓派（刷好系统的）
 - 网线
 - Wi-Fi 适配器（Pi3 可以忽略）

#### 测试软件

 树莓派设置好 Wi-Fi 连接， 安装 FruityWifi，并设置好 in(eth0) 和 out(wlan0),

 out 是树莓派和 AP 连接的， in 则是等下和机顶盒连接。

 ![Fruity Wifi](https://i.loli.net/2018/12/27/5c24bdf1128a0.png)

 之后安装 bettercap 或者其他类似的软件，可以嗅 http 连接的。

#### 监听

 - 打开机顶盒，进入，并且把下载好的应用市场卸载掉。
 - 使用 bettercap
 - Duang！链接出来了。

 ![IPTV Market URL](https://i.loli.net/2018/12/27/5c24bec1d2b30.png)

 可以看到有一个 http 的 get 请求，
 
 `http://210.12.0.175:7084/launcher/data/1254621050/DSMClient20180727.apk`

#### 替换文件
 随后在 Fruity Wifi 里把 eth0 的地址设置成 210.12.0.175，就像下图

 ![Fruity Wifi](https://i.loli.net/2018/12/27/5c24bdf1128a0.png)

 之后 ssh 登陆到树莓派，新建一个`/launcher/data/1254621050` 目录，然后把其他应用市场的文件放进去并重命名为`DSMClient20180727.apk`
 
 并用 Python 也好，其他语言框架也好，在目录上搭建一个 file server，并设置端口为 7084

### Hack!
 打开机顶盒，并点击应用市场，这时候看树莓派那边的 file server 就能看到一个 get 记录，等待一会，进入应用列表就能发现，已经安装好了
