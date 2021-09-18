---
title: Hello World
date: 2018-06-10 23:34:23
tags:
 - Hexo
 - Blog
categories: 
 - 技术
---

开始用[Hexo](https://hexo.io/)写 Blog 了。部署在 GitHub Pages 上，也没有遇到什么坑，还是总结下吧，顺便练练写作。

<!--more-->

### 安装
安装问题可以在[官方文档](https://hexo.io/zh-cn/docs/index.html)中找到，并且有简体中文的翻译。

### 部署
我是部署在了 GitHub Pages 上，所以理论上最好的部署方式是

 - 默认分支是你的 blog,
 - source 或者其他分支，可以上传 blog 的 hexo 项目文件夹。

这样可以保证如果你电脑上的 hexo 项目被你不小心删掉了，也有一个备份。但是请注意如果你使用的一些插件包括了一些密钥，请小心上传。

### 域名
部署在 GitHub Pages 的话，访问域名是你项目的名字，` <你的 gihtub 用户名>.github.io `

当然 GitHub Pages 也支持自定域名，并且在 18 年 5 月份支持自定域名的 HTTPS.

> GitHub Pages is the best way to quickly publish beautiful websites for you and your projects. Just edit, push, and your changes are live. GitHub Pages has supported custom domains since 2009, and sites on the *.GitHub.io domain have supported HTTPS since 2016. Today, custom domains on GitHub Pages are gaining support for HTTPS as well, meaning over a million GitHub Pages sites will be served over HTTPS. [查看更多](https://blog.github.com/2018-05-01-github-pages-custom-domains-https/)

如果使用独立域名的话，需要在 GitHub Pages 设置一个 CNAME，项目设置中可以设置，但是更推荐在命令行中进入你的项目中的 source 文件夹并使用 `echo <你的域名> > CNAME`，然后再重新生成并部署上传到 GitHub. 这样可以防止你的 CNAME 文件在部署的时候被删除。

在设置好 CNAME 之后，需要去域名商中添加两个 CNAME 解析. Host 设置为 `@` 和 `www`，在 Value 那里填写你的项目地址 `<你的 gihtub 用户名>.gtihub.io`，然后坐等生效就好了～

还有一点，GitHub Pages 的自定域名的 HTTPS 并不会立即生效。通常需要等上一段时间，不超过 24 小时 (应该)，之后再去你的项目设置里，找到 Enforce HTTPS，并打上勾，这时候你就可以通过 `https://<你的域名>`访问你的 Blog 了。

~~最后一点，自定域名似乎并不会自动跳转到 https，你想要加一个 301 跳转。先去域名服务商的 dns 那里设置一个 URL 跳转~~ 在你的主题模版中添加

```html
<script type="text/javascript">
    var host = "<你的域名>";
    if ((host == window.location.host) && (window.location.protocol != "https:"))
        window.location.protocol = "https";
</script>
```

到`<head>`标签中，这样就完成了所有的准别工作了～开始写 Blog 吧。