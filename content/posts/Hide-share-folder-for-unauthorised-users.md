---
title: Samba - 对没有权限的用户隐藏文件(夹)
date: 2019-05-05 12:25:27
tags:
 - Samba
 - Linux
 - OMV
categories:
 - 技术
---

最近在搞Openmediavault, 然后在设置每个用户的访问权限，

但是发现就算用户没有某个文件夹的访问权限， 用户通过Finder(macOS) 访问的时候，还是会显示这个文件夹，然后过一会, Finder会提示找不到这个文件夹。

这样就很破坏用户体验， 

于是在duckduckgo了一会，找到了 [Hiding samba share from browse list for unauthorised users](https://serverfault.com/questions/144339/hiding-samba-share-from-browse-list-for-unauthorised-users) 这个post

只要在`smb.conf`里的`[Global]`添加一行 `access based share enum = yes` 就可以了

在OMV里面的话，在SMB/CIFS设置的， 有一个Extra options，在这里把`access based share enum = yes`加进去，然后save， apply changes，

就可以实现对没有权限的用户隐藏掉文件夹， 