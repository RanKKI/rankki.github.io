---
title: "React Native 全局 JS 代码"
date: 2023-06-26T20:50:40+10:00
draft: false
tags:
 - React Native
 - Day.js
categories:
---

最近在写 RN 项目，使用了新的技术栈 Expo。

> Expo 是一个基于 React Native 的开发工具，可以让你更快的开发 React Native 应用，不需要配置 Android 和 iOS 的开发环境，只需要安装 Expo 的客户端就可以运行你的应用。

在开发的过程中，遇到了一个很诡异的 Bug。在 Dev 环境（即使用 Expo Go 进行预览）下，项目运行正常，但是在打包成 ipa 之后，运行在真机上，就会出现 Crash，

一开始通过看 TestFlight 日志，没发定位问题，报错栈只会到某一个 Invoke 方法下，再具体的内容就没有了。

本以为是某种只在真机上才会出现的 Bug，但在使用 `expo start --no-dev` 进行本地预览时，也会出现 Crash。

同样，没有报错栈，不知道为何发生。

## 日志

通过使用 usb 线连电脑，使用 console.app 可以查到 iOS 设备上的日子。

通过搜索+限定 `process name` 即可看到所有该 App 产生的日志。

```log
2021-06-26 20:50:40.000000+1000 <AppName>[<AppName>][fatal][tid:com.facebook.react.ExceptionsManagerQueue] Unhandled JS Exception: TypeError: undefined is not an object

XXXXXXX_Provider
YYYYYYY
ZZZZZZZZ
```

## 定位问题

根据日志，能得出是某个 Provider 出现了问题，但不知道具体哪一行或者哪个方法，

第一步做的是，注释掉全部内容，然后逐步取消注释，看看哪一行会导致 Crash。（虽然这个方法很笨，但是很好用。）

在找到问题函数后，通过打日志查具体是哪一行出现了问题。

我本以为是 firebase SDK 在真机上出现了问题，但是通过打日志，发现是 Day.js 出现了问题。

## Day.js

Firebase 或者 Firestore 使用的 Datetime，并不好计算，因此我会将其转成 Day.js 对象，保存回数据库的时候，再转回其特定格式

我是通过写一个 Day.js 的插件并定义类型，来实现这个功能的。

```js
declare module 'dayjs' {
  interface Dayjs {
    toFirebase(): Timestamp
  }
  function fromFirebase(timestamp: Timestamp): Dayjs
}

dayjsFactory.fromFirebase = (timestamp: Timestamp | null) => {
    if (!timestamp) return null
    return dayjs(timestamp.toMillis())
}
dayjsClass.prototype.toFirebase = function () {
    return Timestamp.fromMillis(this.valueOf())
}
```

这些代码被保存到一个 `dayjs.ts` 文件，并没有被任何文件引用。

问题就出在这里，这个文件并没有被任何文件引用，因此在打包的时候，被忽略/删除掉，并没有被含在包内。

## 解决方法

在 `index.js` 中手动 `import` 该文件

## 完

看起来是在 Dev 环境下，所有代码都会被打包，因此没有问题，但是在打包成 ipa 之后，会忽略未被引用的文件。