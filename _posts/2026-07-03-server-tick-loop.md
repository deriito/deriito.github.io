---
layout: post
title: 服务端 tick 循环的两种写法
lang: zh-Hans
---

写 MMO 服务端绕不开 tick 循环。它决定了战斗结算的精度、状态同步的带宽,以及你半夜被叫起来查的那类 bug 长什么样。这些年我见过的实现基本可以归成两类,取舍很不一样。

## 变步长

最直觉的写法:上一帧花了多久,这一帧就按多久推进。

```go
for {
    now := time.Now()
    dt := now.Sub(last)
    last = now
    world.Update(dt)
    time.Sleep(tickInterval - time.Since(now))
}
```

好处是逻辑上永远"跟得上"真实时间,不会积压。坏处是 `dt` 变成了一个外部输入,而**物理和状态机对 `dt` 敏感**。

> 服务器负载一高,`dt` 就变大,穿墙检测的采样间隔跟着变大,于是玩家在卡顿的时候更容易穿过墙。这类 bug 只在压力测试或者线上高峰复现,本地永远抓不到。

更麻烦的是它不可重放。同一批输入,两次跑出来的结果不一样,战斗回放和反作弊校验都没法做。

## 定步长

把逻辑时间和真实时间解耦,累加真实耗时,按固定片长消费:

```go
const step = 50 * time.Millisecond

accumulator += time.Since(last)
last = time.Now()

for accumulator >= step {
    world.Update(step)   // dt 永远是常量
    accumulator -= step
}
```

`world.Update` 收到的永远是同一个数,于是:

- 物理和状态机行为确定,同样的输入必然得到同样的输出
- 可以录输入、重放整场战斗,反作弊和 bug 复现都成立
- 帧率和逻辑速率解耦,同步频率可以单独调

### 代价

代价是**追帧**。服务器卡了 500ms,累加器里就攒了 10 个 step,下一帧要连着跑 10 次 `Update`。如果这 10 次本身又很慢,累加器只会越涨越多,直接进入死亡螺旋。

所以必须封顶:

```go
if accumulator > maxCatchUp {   // 我们设的是 5 个 step
    accumulator = maxCatchUp
}
```

超出的部分直接丢掉。逻辑时间会比真实时间慢一点,但这远好过整个进程卡死。

## 怎么选

| | 变步长 | 定步长 |
| --- | --- | --- |
| 实现复杂度 | 低 | 中 |
| 确定性 | 无 | 有 |
| 可重放 | 否 | 是 |
| 负载敏感 | 高 | 低(有封顶) |
| 适合 | 挂机、放置、回合制 | 实时战斗、竞技 |

只要有实时战斗和 PvP,**定步长几乎是唯一选择**,确定性带来的可重放能力在排查线上问题时的价值,远超那点额外实现成本。

---

关于封顶值怎么定,以及追帧时要不要跳过渲染同步,参考 Gafferon Games 那篇 [Fix Your Timestep](https://gafferongames.com/post/fix_your_timestep/),写得比我清楚。

回到[文章列表]({{ '/writing/' | relative_url }})。
