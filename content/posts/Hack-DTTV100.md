---
title: Hack DTTV100 机顶盒
date: 2018-12-27 17:36:21
tags:
 - MITM
categories:
 - 技术
---

### 背景

联通给了个IPTV机顶盒， 看了下是 Android 4.4 系统的, 在 Google 一圈之后，发现有人是通过硬件 hack 拿到权限的。但实际上这个机顶盒里面自带一个商店，需要下载， 那只要来个中间人替换应用市场的 apk 就行了。然后正好手上有个树莓派，（虽然说不用也行。 ）

### 开始hack

#### 准备硬件

 - 树莓派（刷好系统的）
 - 网线
 - Wi-Fi 适配器（Pi3可以忽略）

#### 测试软件

 树莓派设置好Wi-Fi连接， 安装 FruityWifi, 并设置好 in(eth0) 和 out(wlan0),

 out 是树莓派和 AP 连接的， in则是等下和机顶盒连接。

 ![Fruity Wifi](https://i.loli.net/2018/12/27/5c24bdf1128a0.png)

 之后安装 bettercap 或者其他类似的软件，可以嗅http连接的。

#### 监听

 - 打开机顶盒，进入，并且把下载好的应用市场卸载掉.
 - 使用bettercap
 - Duang! 链接出来了.

 ![IPTV Market URL](https://i.loli.net/2018/12/27/5c24bec1d2b30.png)

 可以看到有一个http的get请求, 
 
 `http://210.12.0.175:7084/launcher/data/1254621050/DSMClient20180727.apk`

#### 替换文件
 随后在Fruity Wifi里把eth0的地址设置成210.12.0.175，就像下图

 ![Fruity Wifi](https://i.loli.net/2018/12/27/5c24bdf1128a0.png)

 之后ssh登陆到树莓派，新建一个` /launcher/data/1254621050` 目录，然后把其他应用市场的文件放进去并重命名为`DSMClient20180727.apk`
 
 并用Python也好，其他语言框架也好，在目录上搭建一个file server，并设置端口为 7084

### Hack!
 打开机顶盒，并点击应用市场，这时候看树莓派那边的 file server 就能看到一个get记录, 等待一会，进入应用列表就能发现，已经安装好了