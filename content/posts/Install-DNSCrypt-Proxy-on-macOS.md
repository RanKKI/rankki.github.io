---
title: 在 macOS 上使用 DNSCrypt-Proxy
date: 2018-06-11 09:17:53
tags:
 - DNS
categories:
 - 技术
---

[DNSCrypt](https://dnscrypt.info)是一个介于客户端和 DNS 服务商之间的认证式通讯协议，通俗来讲，DNSCrypt 阻止了运营商对你的 DNS 恶意劫持。

<!--more-->

DNSCrypt 支持各个平台，因为我用的是 macOS，所以就拿 macOS 举例，其他平台可以去 dnscrypt.info 查看更多信息。

## 开始安装

### 安装 Homebrew
 
 [Homebrew](https://brew.sh)是 macOS 上的软件包管理器，他可以安装苹果没有预装的软件，比如 `$ brew install wget`
 
 首先需要打开终端，输入
 
 ```bash
 $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
 ```
 
 然后就会自动开始安装，期间会让你输入密码并安装 Xcode Command Line Tools，然后喝杯咖啡去吧。回来之后就安装好了。
 
### 安装 DNSCrypt-Proxy

 直接在终端输入 `$ brew install dnscrypt-proxy`，然后稍等一下就安装了。但是在使用之前需要做一点小配置。
 
### 配置 DNSCrypt-Proxy

 无论是 Vim 还是 nano 还是 Visual Studio Code，只要是能编辑文本的软件就好，打开 `/usr/local/etc/dnscrypt-proxy.toml`，然后修改几个选项。
 
#### 应用日志

程序有任何运行错误都可以在这个日志文件中查看。

找到 log_level，修改如下，哦对，macOS 默认是没有`/usr/local/var/log/`文件夹，需要在`/usr/local/var/` 目录下 `mkdir log`

```
log_level = 0
log_file = '/usr/local/var/log/dnscrypt-proxy.log'
```

#### 查询日志。

这个日志记录了每一个 dns 查询的时间，在 `dnscrypt-proxy.toml` 搜索 query_log 并设置为

```
file = '/usr/local/var/log/query.log'
```

#### NX 日志

NX 日志记录着那些不存在的域名，在 `dnscrypt-proxy.toml` 搜索 nx_log 并设置为

```
file = '/usr/local/var/log/nx.log'
```

#### 禁用系统 DNS

不使用 macOS 自带的域名解析。在 `dnscrypt-proxy.toml` 搜索 ignore_system_dns 并设置为

```
ignore_system_dns = true
```

### 启动 DNSCrypt-Proxy

在终端中输入

```
$ sudo brew services start dnscrypt-proxy
```

然后可以查看 dnscrypt-proxy.log 文件，应该可以看到 dnscrypt-proxy is ready.

之后在将 网络设置中的 DNS 改成 `127.0.0.1` 即可。