---
title: "Build in Public #1"
date: 2023-07-23T22:16:35+10:00
draft: false
tags:
categories:
---

距离第一个独立开发产品已经上线 1 年多了，也时不时的在更新内容和增加新的功能。

IAP 除了一开始加的赞助（并没有很多付费）；现在也加了 AI Chat 的功能，使用是 OPENAI 的 GPT 模型 + Vector 数据库实现的更精准的对话。在此基础上，也使用 Firestore 作为数据库存了一些数据。

## 基础数据

### 下载

大部分用户（68.5%）是通过 App Store 搜索下载的，12% 的用户是通过浏览下载，巅峰是在 2022 年 4 月，应该是上了每周编辑推荐。剩余的 18.8% 是通过网站或者 App 的 Referrer 下载。

### 卸载

不出意外，合乎情理的，卸载占比最多的用户群里，是在 App Store 浏览时下载的用户。其中 16.97% 的选择了卸载，而通过搜索来的用户，其目的性更强，并且知道该 App 的作用、功能，因此，安装/卸载比相对于浏览来的用户低 41.42%。

### 活跃用户

这里的数据有一点看不懂了，通过浏览下载的用户的活跃比，相对于搜索的用户高 13%。

### App Referrer

下载来源最多的 App 竟然是 WeChat...

### 曝光 & 转化

除去上了每周推荐的那一周，平均每月搜索曝光 10k 左右，普通浏览在 500 - 1000 之间。

但转化率很低，搜索带来的转化在 12%，浏览 3.32%，综合平均 8.98%

最近会尝试通过换 App icon + AB Test 来尝试提高转化率。

> 在同类 App 中，即 Reference Apps，转化率高于 6.5% 就超越了 75% 的 App。

<!-- ## AI Chat

### 请求

![](https://s2.loli.net/2023/07/23/IKrk8ZysoPuT5lp.png) -->