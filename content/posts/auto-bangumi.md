---
title: "Jellyfin 自动追番"
date: 2022-07-13T16:34:17+08:00
draft: false
tags:
 - Bangumi
 - 自动化
categories:
 - 技术
---

**该系统主要依赖[蜜柑计划](https://mikanani.me/)的 RSS 订阅**

在 GitHub 上看到个自动追番的项目 [EstrellaXD/Auto_Bangumi](https://github.com/EstrellaXD/Auto_Bangumi)，贡献了一点点代码，加了 `CI/CD`。但自己跑了下，发现在树莓派上跑不起来，可能是 32 位 ARM 架构导致的，加上这个项目和 `qBittrroent` 深度绑定，扩展起来就十分麻烦。比如说，支持 Aria2 或者 Transmission，甚至是对 RSS、其他网站的处理，基本上是不行的。

于是，自己开了个新项目 [RanKKI/Bangumi](https://github.com/RanKKI/Bangumi)，其中对文本处理的代码均来自 [EstrellaXD/Auto_Bangumi](https://github.com/EstrellaXD/Auto_Bangumi)。

一旦花点时间配置好，后续追番都是无成本的。

## 一些 Features

 - 内置[蜜柑计划](https://mikanani.me/)和[动漫花园](https://dmhy.org/)的 RSS 解析器
 - 支持自定义 RSS 插件（用于解析其他 RSS 站点或者非 RSS 网页）
 - 番剧更新提醒（通过 API，比如 iOS 上的 Bark，或者本地脚本触发）
 - 自动筛选清晰度较高的资源
 - 自动过滤已下载过的资源
 - 支持 Aria2, Transmission, qBittorrent
 - 通过 API 下载过完番剧

## 一些坑

在配置这套系统的时候，遇到了难题。最最最主要的是 Jellyfin 刮削不动，一直没有信息产生，于是在本机跑了个代理解决了刮削问题。

## 后续的计划

 - 自定义 Downloader
 - 语言优先级的筛选
 - 自动替换本地视频至较高清的资源（比如目前只有 720P 的放出，在遇到 1080P 的时候还会下载并替换）
 - 完善 API
 - 提供前端网站用于配置和查看信息

## 蜜柑计划

蜜柑计划是一个动漫资源网站，其特点是可以订阅不同番剧、不同字幕组，从而生成一份 RSS 文件

---

Github Repo: [RanKKI/Bangumi](https://github.com/RanKKI/Bangumi)