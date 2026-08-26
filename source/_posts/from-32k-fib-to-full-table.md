---
title: Hash 之后，路由表还慢在哪里？——从 32K FIB 到百万级 Full Table
date: 2026-08-23 13:40:27
tags: FM10K烹饪指南
mermaid: true
---

> 这篇依旧是奇思妙想所做的产物。在我完成VPP+ASIC的架构后我开始思考，我能不能让这台"路由器"吃一个full table呢？
>
> 依旧感谢GPT老师

## Hash之后，问题才刚刚开始

在糊netlab-ng这个项目时，由于整个系统的设定从CRS转变成了CRR。所以我也将他的FIB从4K提升到了32K，这个时候问题就出现了。

这个项目的RIB最早采用了相对直接的架构：

路由条目存放在线性数组中，查找时根据 `prefix + protocol` 顺序扫描整个 Route Table。

但随着数据量从 `4K` 增长到了 `32K`  问题开始变得明显。 如果每收到一条来自 FRR/FPM 的路由更新，都需要在线性数组中扫描一次： 

 **route_find() = O(N)**，那么连续导入 N 条路由时，整体成本就可能逐渐接近：**N × O(N) ≈ O(N²)**。

在几千条路由时，它可能只是表现为“慢了一点”。

到了数万条路由，这种设计就开始成为整个 FIB Convergence Pipeline 中无法忽略的一部分。

于是，NetLab-ng 的 RIB 做了第一轮很直接的优化。

### Route Key Hash Index 与 Free Slot Hint

我为 `(prefix, protocol)` 建立了 Route Key Hash Index，让一条路由不再需要通过扫描整个 Route Array 才能被找到；同时加入 `free_hint`，记录下一次最可能存在空闲 Route Slot 的位置，避免每一次 Route Allocation 都重新从数组头部开始搜索。

前者解决：

> **这条路由在哪里？**

后者解决：

> **下一条路由应该放在哪里？**

通过 `(prefix, protocol)` Hash Index，原本需要线性扫描的 Exact Route Lookup 可以在平均情况下下降到接近 `O(1)`；而 `free_hint` 则避免了每次新增路由时都从 Route Array 的第 0 个位置重新寻找空闲 Slot。至少从 Route CRUD 的角度来看，这个问题似乎已经解决了。

但这里出现了一个更有意思的问题。

如果现在把：NL_L3_SOFTWARE_MAX_ROUTES 从 32K 修改成：1,000,000

NetLab-ng 就真的能够直接吃下一张 Internet Full Table 了吗？答案显然是否定的。

甚至可以说：

> **当 Hash Index 把最明显的 O(N²) 问题消灭以后，真正决定百万级路由表性能的那些问题，才终于暴露出来。**

因为一条路由从 FRR 进入 NetLab-ng，最终被下发到 FM10840 或 VPP，并不仅仅经历一次 `route_find()`：

```
FRR / FPM
    │
    ▼
Route Lookup
    │
    ▼
RIB Update
    │
    ▼
Best Route Selection
    │
    ▼
Next-hop Resolution
    │
    ▼
Hardware / Slowpath Placement
    │
    ▼
FIB Generation
    │
    ├──────────────► FM10840
    │
    └──────────────► VPP
```

在这条路径里，Route Hash 只解决了最前面的一小部分。

继续扩大路由规模以后，新的瓶颈会逐渐转移到：

- RIB Snapshot Copy
- Best Route 重建
- Next-hop / Neighbor Resolution
- ASIC 与 VPP 的 FIB Placement
- Full FIB Serialization
- Control Plane 与 Slowpath 之间的批量同步

换句话说，这已经不再是一个简单的：**怎样把 Hash Table 写得更快？**而变成了：**怎样设计一套能够让百万级路由长期驻留，同时又让绝大多数 Route Update 都不需要重新遍历整个 RIB 的数据面控制架构？**

这也是这篇文章真正想讨论的问题。

从一个简单的 Hash Index 开始，我们最终会一路走到一个更本质的目标：

**百万级路由表优化的关键，并不是让“扫描 100 万条路由”变得更快，而是让绝大多数更新根本不需要扫描这 100 万条路由。**



## 埋个坑吧，先折腾别的了。等有空再让VPP吃全表
