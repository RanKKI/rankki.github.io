---
title: "白鹭 Egret 微信项目在高帧率设备上 Timer 不准确的解决"
date: 2020-12-05T21:14:46+08:00
tags:
 - Egret
 - WeChat
categories:
 - 技术
---

目前实习在的项目组的产品，经常有玩家反馈某个地方倒计时会变快，内部也有小伙伴发现了这个问题，在 iPad Pro 上更是达到了 2 倍速， 但因为受影响的用户很少很少，一直没修。上周，因为日程上不是很忙，便找了一天去修复这个 bug。

经过测试， web 端不会出现这个情况，但微信小游戏上会有速度变快。

首先是在白鹭的开发者论坛上，找了个一篇帖子[基于系统时间的计时器DateTimer(不受FPS影响)](https://bbs.egret.com/thread-27732-1-1.html)看了下代码， 没有问题， 根据时间戳计算差值，然后再 dispatch 事件。 便直接放进项目里尝试，确实解决了倒计时变快的问题，但这很怪异，白鹭不应该会有这么奇怪的 bug, 然后我的 Leader 拉了个群， 其中包括引擎组和隔壁项目组的大佬们，讨论这个问题。

其中一位指出，白鹭也是这么做的，通过查看源码，确实是的， 但在注释里标记了，最高支持到 60 FPS, 而 iPad Pro 是 120 FPS 的设备，因此导致了 1000ms Delay 的 Timer，会在 1 秒内触发两次，而我们的倒计时并没有计算时间的差值，而是直接 `timer--` 所以导致速度异常。

而 Web 端没有问题是因为项目有限制 FPS 到 60， 但这个限制对微信小游戏不好使，解决方法是手动使用 `wx.setPreferredFramesPerSecond(fps)`, 限制 FPS 到 60。

## 分析

ergret-core/src/egret/utils/Timer.ts [source](https://github.com/egret-labs/egret-core/blob/0ffe47cb869e0b4b562f3b20b6f4bfd9ddfdd86a/src/egret/utils/Timer.ts)

```ts {hl_lines=[109], linenostart=101}
public set delay(value: number) {
    if (value < 1) {
        value = 1;
    }
    if (this._delay == value) {
        return;
    }
    this._delay = value;
    this.lastCount = this.updateInterval = Math.round(60 * value);
}
```

```ts {hl_lines=[253, 254, 256], linenostart=247}
/**
* @private
* Ticker以60FPS频率刷新此方法
*/
$update(timeStamp: number): boolean {
    let deltaTime = timeStamp - this.lastTimeStamp;
    if (deltaTime >= this._delay) {
        this.lastCount = this.updateInterval;
    }
    else {
        this.lastCount -= 1000;
        if (this.lastCount > 0) {
            return false;
        }
        this.lastCount += this.updateInterval;
    }
    this.lastTimeStamp = timeStamp;
    this._currentCount++;
    let complete = (this.repeatCount > 0 && this._currentCount >= this.repeatCount);
    if (this.repeatCount == 0 || this._currentCount <= this.repeatCount) {
        egret.TimerEvent.dispatchTimerEvent(this, egret.TimerEvent.TIMER);
    }
    if (complete) {
        this.stop();
        TimerEvent.dispatchTimerEvent(this, TimerEvent.TIMER_COMPLETE);
    }
    return false;
}
```

白鹭在设置 delay 的时候，会对 `updateInterval` 赋值 `60 * dealy`, 然后 `$update` 方法中，会对这个值做计算，然后调用事件， 但 `$update` 方法是以系统默认的刷新率刷新的。

比如在 60fps 设备上用 1/60s 调用 `$update`，而在 120fps 设备上是以 `1/120s` 调用，但 `updateInterval` 却还保持在 `60 * dealy`，因此导致在 120fps 设备上，设置的 dalay 从原本 1000ms，变成了 500ms, 也就导致了 1s 内发了两次事件
