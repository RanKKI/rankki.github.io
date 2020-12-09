---
title: 对于 Bilibili 抽奖的数据分析
date: 2020-06-06 12:07:37
tags:
 - DataAnalysis
 - Bilibili
categories:
 - 技术
---

最近开始深入数据分析的坑, 虽然在学了FIT1043, 但这门课是第一年的课，加上比较水(但又是必修课)，
就没有学到东西，最近闲来无事，正好在哔哩哔哩参加了`@毕导THU`的抽奖， 并且有33个名额，所以想着来做一次分析。

> 声明下, 以下分析均为个人观点. 并且数据仅供学习使用


# 前期准备

## 分析API

首先，需要获取数据, 在抽奖动态页面中进行抓包，可以看到一个对`/lottery_notice`的请求, 查看内容可以看到所有中奖用户的uid,

```json
{
    "code": 0,
    "data": {
        "first_prize": 3,
        "second_prize": 10,
        "third_prize": 20,
        "lottery_result": {
            "first_prize_result": [
                {
                    "uid": 21696250,
                    "name": "Alan婪",
                    "face": "https://i1.hdslb.com/bfs/face/d03761481bdf0adec61e876a56309c1e2e444721.jpg"
                }
            ],
            "second_prize_result": [
                {
                    "uid": 17660688,
                    "name": "城西吖吖",
                    "face": "https://i1.hdslb.com/bfs/face/798c3f5751c934df20af1cb60b1ec7ba8e05f318.jpg"
                }
            ],
            "third_prize_result": [
                {
                    "uid": 4134700,
                    "name": "weacho",
                    "face": "https://i0.hdslb.com/bfs/face/9280e500dba7cf754c9e82e3428a3192de64e47c.jpg"
                }
            ]
        },
        "first_prize_cmt": "iPad Air 3 64G",
        "second_prize_cmt": "Kindle青春版",
        "third_prize_cmt": "100元现金红包",
    }
}
```

以及, 打开用户动态，可以看到另一个请求`/space_history`, 这个API返回了, 用户的信息和发送、转发的动态。

```json
{
    "code": 0,
    "msg": "",
    "message": "",
    "data": {
        "has_more": 0,
        "attentions": {
            "uids": [
                ...
            ]
        },
        "cards": [
            {
                "desc": {
                    "uid": 21696250,
                    "type": 1,
                    "rid": 397377349256764900,
                    "acl": 0,
                    "view": 2552,
                    "repost": 2,
                    "comment": 21,
                    "like": 8,
                    "is_liked": 0,
                    "dynamic_id": 397377349254099260,
                    ...
            "card": "{ \"user\": { \"uid\": 21696250, \"uname\": \"Alan婪\", \"face\": \"https:\\/\\/i1.hdslb.com\\/bfs\\/face\\/d03761481bd
            ...
```

接着获取转发抽奖动态所有人的信息

> which 不太可能, 一共有18.7万人转发, 借口最多给640人, 但我看了下数据, 动态是2020-05-27 18:02:12发送的, 第一个非`@毕导THU`转发是2020-05-27 18:07:38, 最后一个转发 2020-06-06 13:36:59, 所以可以当作sample使用(么?)
> 
>但是，我发现评论接口没有限制, 大概有1.6万人的评论, 因为这是个抽奖动态，可以认定评论的人都参与了转发(即抽奖), 于是可以爬取评论, 然后进行对比



## 数据清理

然后需要进行数据清理, 通过获奖的api, 可以提取出所有用户的uid, 

```python
list(
    map(
        lambda x: x["uid"], 
        reduce(
            lambda x, y: x + y, 
            data["data"]["lottery_result"].values()
        )
    )
)
```

以及将用户信息和发送、转发的动态提取出来
> 在这里, 我只提取了我认为可能会有影响的因素
>
> class `Card` 是转发和回复的内容

```python
class User(object):

    def __init__(self,
                 uid: int,
                 name: str,
                 level: int,
                 vip_type: int,
                 vip_status: int):
        self.uid = uid
        self.name = name
        self.level = level
        self.vip_type = vip_type
        self.vip_status = vip_status

class Card(object):

    def __init__(self,
                 content: str,
                 timestamp: int,
                 origin: Card = None,
                 id: int = None,
                 user: User = None):
        self.content = content
        self.timestamp = timestamp
        self.origin = origin
        self.id = id
        self.user = user

```

随后, 将信息写入csv文件, 以便之后使用pandas进行分析


# 分析

```python
import pandas as pd
prize_users = pd.read_csv("./prize_users.csv")
all_users = pd.read_csv("./all_other_users.csv")
```

 > `prize_users`是中奖用户, `all_users`是~~所有~~参与的用户, (因为接口问题, 无法获取所有用户, 同时评论有1.6万人, 但因为~~懒~~怕被ban ip只先获取了1.2万人)
 >
 > `all_users` 不包括发布抽奖的up主本人

## 数据定义
```python
prize_users.head()
```
| uid      | level | vip_type | vip_status | time_after |
|----------|-------|----------|------------|------------|
| 21696250 | 5     | 0        | 0          | 1767       |
| 29332457 | 5     | 1        | 0          | 580531     |
| 94078703 | 5     | 2        | 1          | 5347       |
| 17660688 | 5     | 0        | 0          | 63184      |
| 22417036 | 5     | 1        | 1          | 15013      |

 - uid, 用户的b站id
 - level, 用户等级
 - time_after, 在抽奖动态发送多少秒后转发的。

对于vip_type和vip_status, 通过`openbilibili`的`/app/admin/main/vip/model/vip.go`中可以看到这项数值的定义.

```golang
// const vip enum value
const (
    ...

	NotVip    = 0 //非大会员
	Vip       = 1 //月度大会员
	AnnualVip = 2 //年度会员

	VipStatusOverTime    = 0 //过期
	VipStatusNotOverTime = 1 //未过期
	VipStatusFrozen      = 2 //冻结
	VipStatusBan         = 3 //封禁
    ...
)
```

### prize_users 数据详情


|       | uid          | level     | vip_type  | vip_status | time_after    |
|-------|--------------|-----------|-----------|------------|---------------|
| count | 3.300000e+01 | 33.000000 | 33.000000 | 33.000000  | 33.000000     |
| mean  | 1.632111e+08 | 4.484848  | 0.909091  | 0.484848   | 313364.333333 |
| std   | 1.724799e+08 | 0.870388  | 0.842750  | 0.507519   | 320576.262967 |
| min   | 4.134700e+06 | 2.000000  | 0.000000  | 0.000000   | 281.000000    |
| 25%   | 2.525509e+07 | 4.000000  | 0.000000  | 0.000000   | 3331.000000   |
| 50%   | 4.962842e+07 | 5.000000  | 1.000000  | 0.000000   | 199071.000000 |
| 75%   | 2.941396e+08 | 5.000000  | 2.000000  | 1.000000   | 597423.000000 |
| max   | 5.098047e+08 | 6.000000  | 2.000000  | 1.000000   | 812534.000000 |


### all_users 数据详情

|       | uid          | level        | vip_type     | vip_status   | time_after    |
|-------|--------------|--------------|--------------|--------------|---------------|
| count | 1.263000e+04 | 12630.000000 | 12630.000000 | 12630.000000 | 12630.000000  |
| mean  | 2.243269e+08 | 3.829295     | 0.542280     | 0.236817     | 250477.374822 |
| std   | 1.743140e+08 | 0.999087     | 0.726571     | 0.425146     | 289099.261204 |
| min   | 1.300000e+01 | 1.000000     | 0.000000     | 0.000000     | 212.000000    |
| 25%   | 3.654792e+07 | 3.000000     | 0.000000     | 0.000000     | 10677.750000  |
| 50%   | 2.382717e+08 | 4.000000     | 0.000000     | 0.000000     | 59733.500000  |
| 75%   | 3.855455e+08 | 5.000000     | 1.000000     | 0.000000     | 583955.000000 |
| max   | 6.022326e+08 | 6.000000     | 2.000000     | 1.000000     | 852418.000000 |


## 数据分析

### 用户等级分布

![用户等级分布](https://i.loli.net/2020/06/06/krzh17HwDS3Zonv.png)

左边的是得奖用户的等级划分, 右边则是所有用户的等级.

得奖用户中的5级账号是明显多余其他等级的, 占总共的50%以上。 

并且中奖用户的平均等级是4.48, 而所有用户等级的平均数是在3.82, (std相差 .129), 即中奖用户等级偏高。 

### 大会员占比

![大会员状态](https://i.loli.net/2020/06/06/AJfYIUdthxDqwTX.png)

![大会员类型](https://i.loli.net/2020/06/06/GPU9LYDjoXWplkq.png)

可以看到得奖用户的大会员状态占比基本持平, 但可以看到没有大会员的用户是有大会员用户数的3倍, 即大会员会对抽奖结果造成影响(特殊加成), 但是, 年度大会员和月度大会员似乎没有特殊加成

### 转发时间

![转发时间](https://i.loli.net/2020/06/06/3Go5mUpDOalEjey.png)

X轴是以`小时`为单位的, 即转发时间为抽奖动态发布的x小时

中奖用户转发的时间和大部分用户发帖的时间基本一致,

![前5个小时转发](https://i.loli.net/2020/06/06/9LJQfmAEZ1OrcDo.png)

但在这`前5个小时转发`的图里, 可以看到有11个(33%)中奖用户的转发时间是在前2个小时, 似乎在抽奖动态发出后半个小时和一个小时间转发的中奖率更大.

# 结论
> ~~怀疑~~(不用怀疑)数据有缺失, 导致结果不准

就如我文章前头说的, 都是个人观点, 因为只是学习使用，分析啥的都很渣. 

但通过这些数据，得出的结论是，大会员会有特殊加成, 前2个小时转发也会有加成(大概).
