---
title: 白鹭引擎的反向遮罩
date: 2021-04-07T20:30:46+08:00
tags:
 - Egret
categories:
 - 技术
---

```typescript
let container: egret.DisplayObjectContainer = new egret.DisplayObjectContainer();
let blendShape = new egret.Shape(); // 用来作为遮挡背景
blendShape.graphics.beginFill(0x000000, 0.6);
blendShape.graphics.drawRect(0, 0, this.width, this.height);
blendShape.graphics.endFill();
container.addChild(blendShape);
container.addChild(target);
target.blendMode = egret.BlendMode.ERASE;
let renderTexture:egret.RenderTexture = new egret.RenderTexture();
renderTexture.drawToTexture(container);

let blendBitmap = new egret.Bitmap(renderTexture);
this.addChild(blendBitmap);
blendBitmap.touchEnabled = true; // 允许点击
blendBitmap.pixelHitTest = true; // 是否开启精确像素碰撞。设置为true显示对象本身的透明区域将能够被穿透。
this.blendBitmap = blendBitmap
target.blendMode = egret.BlendMode.NORMAL
blendBitmap.anchorOffsetX = target.x
blendBitmap.anchorOffsetY = target.y
blendBitmap.x = target.x
blendBitmap.y = target.y
```
