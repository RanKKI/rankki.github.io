---
title: "Cocos Creator 3.3 版本写 2D 游戏，点击事件无反应" 
date: 2021-09-20T11:30:38+08:00
draft: false
tags:
 - Cocos Creator
categories:
 - 技术
---

周末闲来无事，学习下 Cocos Creator 3.3 (以下简称 CC)，之前接触的版本是 2.X。3.X 最大的改变是合并了 CC 3D 这个项目，官方的介绍是


> Cocos Creator 3.0 集成了原有 2D 和 3D 两套产品的所有功能，带来了诸多重大更新，将做为 Creator 之后的主力版本。同时 v3.0 还延续了 Cocos 在 2D 品类上轻量高效的优势，并且为 3D 重度游戏提供高效的开发体验。 [来源](https://docs.cocos.com/creator/3.0/manual/zh/release-notes/upgrade-guide-v3.0.html)


相较于 2D 游戏的 x & y 轴， 3D 游戏多了 z 轴的概念，~~毕竟是 3D 空间~~

## 现象

在写游戏中，经常需要给某个节点增加一个缓动的动画，基本上是这么写

```ts
tween(this.node)
    .to(5 / 24, { scale: new math.Vec3(1.1, 1.1) })
    .to(5 / 24, { scale: new math.Vec3(1.0, 1.0) })
    .start()
```

这就是一个很普通的放大再缩小的动画，但在这个动画被出发之后，我发现这个节点的点击事件不好使了。 本来以为是哪个把这个节点上 Button 组件设置为禁止点击了。但通过删除相关代码，延迟打印`interactable`，结果都是证明了，这个组件没有问题。

随后，给 Button 组件[[cocos/ui/button.ts](https://github.com/cocos-creator/engine/blob/v3.3.2/cocos/ui/button.ts)] 的点击事件打断点

```ts {linenostart=788}
protected _onTouchBegan (event?: EventTouch) {
    if (!this._interactable || !this.enabledInHierarchy) { return; }

    this._pressed = true;
    this._updateState();
    if (event) {
        event.propagationStopped = true;
    }
}
```

发现这个断电未生效， 这个方法没有被触发到，所以就导致了点击事件没有被派发出来。

那就很神奇了， 于是就顺着找，找到了派发点击事件的地方 [[cocos/core/scene-graph/node-event-processor.ts](https://github.com/cocos-creator/engine/blob/v3.3.2/cocos/core/scene-graph/node-event-processor.ts)]

```ts {linenostart=66}
function _touchStartHandler (this: EventListener, touch: Touch, event: EventTouch) {
    const node = this.owner as Node | null;
    if (!node || !node._uiProps.uiTransformComp) {
        return false;
    }

    touch.getUILocation(pos);

    if (node._uiProps.uiTransformComp.isHit(pos, this)) {
        event.type = SystemEventType.TOUCH_START.toString();
        event.touch = touch;
        event.bubbles = true;
        node.dispatchEvent(event);
        return true;
    }

    return false;
}
```

继续打断点，发现点击开始的事件收到了，但在`74`行的`isHit(pos, this)`判断中，走了`else`分支，因此导致这个事件没有从`node`上派发出去。

`isHit`是 uiTransform [[cocos/2d/framework/ui-transform.ts](https://github.com/cocos-creator/engine/blob/v3.3.2/cocos/2d/framework/ui-transform.ts)] 的一个方法，用于计算当前节点的点击。(代码比较多，就只展示出问题的一部分)

```ts {linenostart=407}
...
// 将一个摄像机坐标系下的点转换到世界坐标系下
camera.node.getWorldRT(_mat4_temp);
const m12 = _mat4_temp.m12;
const m13 = _mat4_temp.m13;
const center = legacyCC.visibleRect.center;
_mat4_temp.m12 = center.x - (_mat4_temp.m00 * m12 + _mat4_temp.m04 * m13);
_mat4_temp.m13 = center.y - (_mat4_temp.m01 * m12 + _mat4_temp.m05 * m13);
Mat4.invert(_mat4_temp, _mat4_temp);
Vec2.transformMat4(cameraPt, point, _mat4_temp);

this.node.getWorldMatrix(_worldMatrix);
Mat4.invert(_mat4_temp, _worldMatrix);
if (Mat4.strictEquals(_mat4_temp, _zeroMatrix)) {
    continue;
}
...
```

通过断点一步一步的检查， 发现程序进入了`420`行的`if`分支。做了下交叉对比，在没有播放上面的缓动动画之前，这里是没问题的，播放之后就进入了这个分支。

那肯定是变量值有问题，比对后发现`_worldMatrix`的第 11 位 (m10，第 3 行，第 3 列) 不一致。

--- 
播放动画之前
|      |      |   |   |
|------|------|---|---|
| 1    | 0    | 0 | 0 |
| 0    | 1    | 0 | 0 |
| 0    | 0    | 1 | 0 |
| -555 | -884 | 0 | 1 |

---
播放动画之后
|      |      |   |   |
|------|------|---|---|
| 1    | 0    | 0 | 0 |
| 0    | 1    | 0 | 0 |
| 0    | 0    | 0 | 0 |
| -555 | -884 | 0 | 1 |

然后将第二组 Matrix 带入计算器，去计算 Invert Matrix，确实不存在，所以导致`Mat4.strictEquals(_mat4_temp, _zeroMatrix)`成立。

那下面的问题就是，什么导致了 m10 位变成了 0，以及，猜想是否正确。

---

在调用 Node 的`getWorldMatrix`方法时，会先调用`updateWorldTransform`更新节点的世界变换信息，里面处理 Rotation & Scale 引起了注意。.

```ts {linenostart=569}
if (dirtyBits & TransformBit.RS) {
    Mat4.fromRTS(child._mat, child._lrot, child._lpos, child._lscale);
    Mat4.multiply(child._mat, cur._mat, child._mat);
    if (dirtyBits & TransformBit.ROTATION) {
        Quat.multiply(child._rot, cur._rot, child._lrot);
        NodePool.setVec4(child._poolHandle, NodeView.WORLD_ROTATION, child._rot);
    }
    Mat3.fromQuat(m3_1, Quat.conjugate(qt_1, child._rot));
    Mat3.multiplyMat4(m3_1, m3_1, child._mat);
    child._scale.x = m3_1.m00;
    child._scale.y = m3_1.m04;
    child._scale.z = m3_1.m08;
    NodePool.setVec3(child._poolHandle, NodeView.WORLD_SCALE, child._scale);
}
```


`Mat4.fromRTS(child._mat, child._lrot, child._lpos, child._lscale);`里存在`m10`计算的过程,

```ts
public static fromRTS(out: Out, q: Quat, v: VecLike, s: VecLike) {
    ...
    const sz = s.z;
    ...
    out.m10 = (1 - (xx + yy)) * sz;
    ...
}
```

而变量`s`正是这个节点的缩放信息。

那问题出在那里就很明确了，如果该节点的`z-Scale`等于`0`，那`m10`必然会等于`0`，再回过头看之前写的缓动动画

```ts
tween(this.node)
    .to(5 / 24, { scale: new math.Vec3(1.1, 1.1) })
    .to(5 / 24, { scale: new math.Vec3(1.0, 1.0) })
    .start()
```

巧了，Vec3 对象，就没设置 z 上的信息，而 Vec3 的初始值又是 0，也就导致了以上一连串的问题

```ts
constructor (x?: number | Vec3, y?: number, z?: number) {
    super();
    if (x && typeof x === 'object') {
        this.x = x.x;
        this.y = x.y;
        this.z = x.z;
    } else {
        this.x = x || 0;
        this.y = y || 0;
        this.z = z || 0;
    }
}
```

2D 游戏没有第三个轴 Z，因此这个轴上的信息并不能反馈到画面上，但 CC 底层计算又会考虑这些。

后续的修改方案就简单了，也不能复现出问题了。

```ts
tween(this.node)
    .to(5 / 24, { scale: new math.Vec3(1.1, 1.1, 1.0) })
    .to(5 / 24, { scale: new math.Vec3(1.0, 1.0, 1.0) })
    .start()
```