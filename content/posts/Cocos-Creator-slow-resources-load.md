---
title: "修复 Cocos Creator 加载资源过慢"
date: 2021-09-10T11:56:35+08:00
draft: false
tags:
 - Cocos Creator
categories:
 - 技术
---

在写 Cocos Creator 2.4.6 项目时候 (以下简称 CC)，需要增加一些反馈的音效。比如说按钮点击时，播放动画时。但在处理播放声音的时候，遇到个奇怪的问题。在有些设备上，声音一直会有延迟，感觉上来说，少则 100ms 多则 500ms，这点的体验就十分的差了，也是完全不能接受的。

<!--more-->

这个项目是打包成微信小游戏的，所有资源都在包内，不存在远程加载的问题。

但，CC 不可能不对资源进行缓存，同时资源也是在本地，按道理来说，不应该会这么慢。第一次慢能理解，毕竟要进行加载，但二次应当是无感的。

---

# TLDR;

CC 在资源加载 `cc.resources.load` 执行了太多操作。 设计上十分的粗燥。在加载已经加载过的资源时，CC 在取缓存之前做了太多无意义的操作。因此，解决方案是自己在业务层手动封装一个 `loadRes` 之类的方法，直接那文件的 uuid，然后从 assets 里通过 uuid 拿资源，不走 CC 自己的那一套。

```ts
export async function loadRes<T extends cc.Asset>(path: string, type: typeof cc.Asset): Promise<T> {
    if (!path) {
        return
    }
    let bundle = cc.resources
    let bundleName = "resources"
    if (!bundle.getInfoWithPath(path)) {
        bundleName = "remote"
        bundle = cc.assetManager.getBundle(bundleName)
        if (!bundle) {
            bundle = await new Promise((resolve, reject) => {
                cc.assetManager.loadBundle(bundleName, (err, bundle) => {
                    err ? reject(err) : resolve(bundle)
                })
            })
        }
    }
    const res = await getResFromBundle<T>(bundleName, path, type)
    if (res) {
        return res
    }
    return new Promise((resolve, reject) => {
        bundle.load(path, type, (err, asset) => {
            err ? reject(err) : resolve(asset as T)
        })
    })
}

export async function getResFromBundle<T extends cc.Asset>(bundle: string, path: string, type: typeof cc.Asset): Promise<T> {
    if (!path) {
        return null
    }
    const uuid = pathToUUID(bundle, path, type)
    if (!uuid) {
        return null
    }
    const res = cc.assetManager.assets.get(uuid)
    if (res) {
        return res as T
    }
    return null
}

function pathToUUID(bundleName: string, path: string, type: typeof cc.Asset): string {
    const bundles = cc.assetManager.bundles
    const getFromBundle = (name: string): string => {
        if (!bundles.has(name)) {
            return null
        }
        const bundle = bundles.get(name)
        const info = bundle.getInfoWithPath(path, type)
        if (!info) {
            return null
        }
        if (info.redirect) {
            return getFromBundle(info.redirect)
        }
        return info.uuid
    }
    return getFromBundle(bundleName)
}
```
