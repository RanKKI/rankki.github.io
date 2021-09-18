---
title: '在 NAS 上安装 TimeMachine'
date: 2019-02-20 00:36:56
tags:
 - Apple
 - TimeMachine
 - NAS
categories:
 - 技术
---

<!--more-->
## 安装

```bash
$ sudo apt-get install netatalk avahi-daemon libnss-mdns
```

---

#### Netatalk

> Netatalk is an OpenSource software package, that can be used to turn a *NIX machine into an extremely high-performance and reliable file server for Macintosh computers.
> Using Netatalk's AFP 3.3 compliant file-server leads to significantly higher transmission speeds compared with Macs accessing a server via SaMBa/NFS while providing clients with the best possible user experience (full support for Macintosh metadata, flawlessly supporting mixed environments of classic Mac OS and OS X clients)

#### Avahi

> Avahi is a system which facilitates service discovery on a local network via the mDNS/DNS-SD protocol suite. This enables you to plug your laptop or computer into a network and instantly be able to view other people who you can chat with, find printers to print to or find files being shared. Compatible technology is found in Apple MacOS X (branded "Bonjour" and sometimes "Zeroconf").
> Avahi is primarily targetted at Linux systems and ships by default in most distributions. It is not ported to Windows at this stage, but will run on many other BSD-like systems. The primary API is D-Bus and is required for usage of most of Avahi, however services can be published using an XML service definition placed in /etc/avahi/services.

#### mDNS

> nss-mdns is a plugin for the GNU Name Service Switch (NSS) functionality of the GNU C Library (glibc) providing host name resolution via Multicast DNS (aka Zeroconf, aka Apple Rendezvous, aka Apple Bonjour), effectively allowing name resolution by common Unix/Linux programs in the ad-hoc mDNS domain .local.
> nss-mdns provides client functionality only, which means that you have to run a mDNS responder daemon seperately from nss-mdns if you want to register the local host name via mDNS. I recommend Avahi.
> nss-mdns is very lightweight (9 KByte stripped binary .so compiled with -DNDEBUG=1 -Os on i386, gcc 4.0), has no dependencies besides the glibc and requires only minimal configuration.
> By default nss-mdns tries to contact a running avahi-daemon for resolving host names and addresses and making use of its superior record cacheing. Optionally nss-mdns can be compiled with a mini mDNS stack that can be used to resolve host names without a local Avahi installation. Both Avahi support and this mini mDNS stack are optional, however at least one of them needs to be enabled. If both are enabled a connection to Avahi is tried first, and if that fails the mini mDNS stack is used.

## 配置

修改 Netatalke 文件。

```bash
$ sudo vim /etc/netatalk/AppleVolumes.default
```

在文件底部加入 `[Folder] [Name] options:tm` ，比如我的 `\mnt\data\timemachine "Time Machine Backup" options:tm`

然后新建一个 Avahi 服务
```bash
$ sudo vim /etc/avahi/services/afpd.service
```

输入一下内容
```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
	<name replace-wildcards="yes">%h</name>
	<service>
        <type>_afpovertcp._tcp</type>
        <port>548</port>
    </service>
    <service>
        <type>_device-info._tcp</type>
 	       <port>0</port>
        <txt-record>model=Xserve</txt-record>
    </service>
</service-group>
```

修改 nss-mdns 配置文件

```bash
$ sudo vim /etc/nsswitch.conf
```

在 hosts 后面加入 mdns4 和 mdns

## 完成设置
```bash
$ sudo service netatalk restart
$ sudo service avahi-daemon restart
```