---
title: Monash Uni 课程
date: 2018-12-23 10:44:29
tags:
 - Tool
categories:
 - 技术
---

原来写过一个爬虫，爬[Monash 2019 Handbook](http://www.monash.edu/pubs/2019handbooks/index.html)里课程的信息，比如有没有需要前置课程，有没有考试，有没有和其他课冲突。

后来就完善了一下代码，并且将起整理出来，同时也写了个小前端，

本来是用 python + flask 写的，但后来发现并没有什么需要后端操作的，

于是就把数据用 json 格式储存在 github 上，并且用[Vue.js](https://vuejs.org)写了个前端，用于搜索，同时不需要独立服务器。 

CSS 框架用的是[MDUI](https://www.mdui.org), HTTP 请求部分用的是[Axios](https://github.com/axios/axios), CDN 用的是[Showfom](https://css.loli.net)家的。

代码的位置是在[github/rankki/moansh](https://github.com/RanKKI/monash)这里，然后使用的地址是在这个域名下[rankki.xyz/monash](https://rankki.xyz/monash/)


![显示界面](https://i.loli.net/2018/12/23/5c1ef8165918f.png)

![搜索界面](https://i.loli.net/2018/12/23/5c1ef81657d93.png)

信息包括了课的代码，名字，faculty，exam 和平时的其他 assessment 成绩，以及哪个学期有课，

然后，因为是写的代码爬的数据，有可能会有信息不对， 如果有发现不对的，可以在 [Telegram](http://t.me/RanKKI_L) 联系到我。