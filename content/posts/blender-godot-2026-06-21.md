---
title: "Blender & Godot 3D 教程 - 2026-06-21 - Godot 4 联机架构选型:状态同步 vs 帧同步,选错了重做要一个月"
date: 2026-06-21T04:00:00+08:00
description: "把 6-15 提到的状态同步 vs 帧同步拆开讲透:Godot 4 怎么落地状态同步(MultiplayerSynchronizer + Replication Config)、帧同步的确定性物理门槛、两个方案的代码对比、选型决策树,以及 Godot 4 官方推荐方向。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-21.png
  alt: "两台电脑通过发光网络线相连,各显示同一 3D 联机游戏场景"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-21.png)

> 系列第六篇。昨天的 3D 物理教程文末说"为联机物理同步铺路",今天我们就来挑明:**做联机游戏,你必须先在「状态同步」和「帧同步」里二选一**。选错了,**后期重写的代价以"月"为单位计**。
>
> 6-15 那篇我们快速过了一遍这两个概念,今天单独拎出来,**配合 Godot 4.6 真实代码 + 坑点 + 决策树**,讲透怎么选、怎么落地。

## 一、30 秒回顾:昨天那几个关键词

> 没看过 6-15 的同学,先扫一眼这段,有概念就好;细节后面展开。

**状态同步**:每帧把对象的位置、旋转、血量当作一坨数字打包发给对方。
- 典型:FPS、TPS、MMO、绝大多数现代多人游戏
- Godot 4 原生支持:`MultiplayerSynchronizer`

**帧同步(Lockstep)**:**不传状态,只传输入**。所有客户端收到同样的输入序列,自己跑同样的确定性物理,得到同样的输出。
- 典型:格斗(街霸)、RTS(星际争霸 2)、策略战棋
- Godot 4 没有原生支持,要自己写确定性物理 + 录输入

**怎么选?一句话**:操作密集型(格斗、RTS)用帧同步;显示密集型(FPS、RPG)用状态同步。**本系列 100% 用状态同步**,因为 Godot 没原生帧同步,自己撸的门槛极高。

## 二、状态同步:Godot 4 怎么落地?

昨天我们写过一个简单 Demo:两个窗口各控制一个红色立方体移动,对方能看到。用的就是状态同步。完整拆开看:

### 2.1 三件套

| 节点/类 | 作用 |
|---|---|
| `ENetMultiplayerPeer` | 底层 UDP 传输 |
| `MultiplayerSpawner` | 节点生死的同步(玩家加入时自动创建 Player 节点) |
| `MultiplayerSynchronizer` | 节点属性的同步(position/rotation 等每帧广播) |

### 2.2 关键代码:客户端怎么"告诉"服务器自己的位置?

```gdscript
# player.gd - 每个玩家角色挂这个脚本
extends Node3D

@export var speed: float = 5.0
var input_direction: Vector3 = Vector3.ZERO

func _process(delta: float) -> void:
    # 1. 客户端读自己的输入
    input_direction.x = Input.get_axis("move_left", "move_right")
    input_direction.z = Input.get_axis("move_forward", "move_back")
    
    # 2. 客户端本地移动(本地预测,看起来"丝滑")
    position += input_direction * speed * delta
    
    # 3. MultiplayerSynchronizer 自动把 position 广播给其他人
    #    (Replication Config 里配好的)
```

看起来简单,但背后**全靠 `MultiplayerSynchronizer` 的 Replication Config** —— 这才是状态同步的"灵魂":

### 2.3 配 Replication Editor(最容易出错的一步)

选中场景里的 `Player` 节点 → 添加子节点 `MultiplayerSynchronizer` → Inspector 点 **Replication Editor** 按钮 → 加一条同步路径:

```
Player            : Node3D    (复制整个节点的 transform)
  └─ Mesh         : MeshInstance3D  (同步颜色变化,可选)
```

**关键设置**:
- **Replication Mode** = `Always`(每帧都同步)或 `On Change`(属性变了才同步)
- **Spawn** = `true`(服务器 spawn 时,所有客户端都自动 instantiate)
- **Sync** = `true`(这个属性进同步列表)

> 💡 **小贴士**:Replication Editor 的"+ Add Replication"按钮藏得很深,第一次用一定找不到。**它是 Inspector 顶部那个 Replication → Replication Editor 子面板里的按钮,不是节点右键菜单**。

### 2.4 服务器权威 vs 客户端权威

上面的代码,客户端**自己算自己的位置**,服务器只负责转发 — 这叫 **"客户端权威 + 服务器转发"**。

```
[客户端 A] 自己算 position = (1.0, 0, 2.0) → 同步器广播
[客户端 B] 收到 A 的 position = (1.0, 0, 2.0) → 强制把 A 拉到这位置显示
```

**问题**:客户端可以撒谎。改成 position = (99999, 0, 99999),全服玩家就看到 A 瞬移到地图外 — 这就是作弊。

**正经做法:服务器权威**:

```gdscript
# server_player.gd - 只在服务器跑的物理
extends Node3D

func _physics_process(delta: float) -> void:
    # 服务器才是物理的"唯一真相"
    # 1. 收所有客户端的 input(通过 RPC)
    # 2. 服务器自己跑物理,算每个玩家的 position
    # 3. MultiplayerSynchronizer 把服务器算好的 position 广播给所有客户端
    # 4. 客户端的 _physics_process 实际上什么都不算,只接收结果
    pass
```

这个模型(权威服务器 + 客户端预测 + 服务器调和)就是 FPS 大作的标配 — 命运、Apex、Valorant 都是这套。**代价是代码复杂度翻 3-5 倍**,本系列暂时不展开。

## 三、帧同步:Godot 里为什么难?

### 3.1 帧同步的核心要求:确定性物理

帧同步的前提是 — **任何客户端跑同样的输入序列,必须得到完全一样的结果**。

```
输入序列: [W, W, idle, A, A, jump]
客户端 A 跑物理 → position = (3.2, 1.0, 0.0)
客户端 B 跑物理 → position = (3.2, 1.0, 0.0)   ← 必须完全一致
```

### 3.2 Godot 物理是不是确定性的?

**半确定性**。这是为什么 Godot 不适合做帧同步的关键。

| 因素 | 状态 | 影响 |
|---|---|---|
| `RigidBody3D` 物理引擎 | ❌ 不严格确定 | 浮点误差会累积,不同 CPU 算出来不同 |
| `CharacterBody3D` + 自写物理 | ✅ 自己写,可确定 | 你可以保证数学一致 |
| 浮点运算 | ❌ x86 / ARM / GPU 算出来可能差 1e-7 | 1e-7 累积 1 万帧 = 0.001,玩家能看到"飘" |
| 随机数 | ❌ Godot 默认 `randf()` 用 Mersenne Twister,但不同平台初始化可能不同 | 需要自己 seed |
| 时间步长 | ❌ 默认 `delta` 是浮点秒,不同硬件 delta 不一样 | 必须用固定步长 + 整数帧号 |

### 3.3 如果你坚持在 Godot 4 做帧同步,怎么搞?

**自写一套确定性物理**,全部用整数或固定精度小数:

```gdscript
# deterministic_player.gd - 帧同步的物理(示意)
extends Node3D

var fixed_pos: Vector2i = Vector2i.ZERO  # 用整数,不用浮点
var fixed_vel: Vector2i = Vector2i.ZERO

func step(input_dir: Vector2i) -> void:
    # 固定步长物理,完全确定
    fixed_vel += input_dir * 16  # 每帧加 16 单位速度
    fixed_pos += fixed_vel
    
    # 转回浮点给渲染用
    position = Vector3(fixed_pos.x * 0.001, 0, fixed_pos.y * 0.001)
```

**网络部分**:

```gdscript
# 服务器只发输入序列
@rpc("any_peer", "call_remote", "reliable")
func submit_input(frame: int, input: Vector2i) -> void:
    if not multiplayer.is_server():
        return
    # 服务器累计所有客户端输入,每 tick 广播一帧
    pass
```

**代价**:
- 物理完全自己写(不能用 Godot 自带 RigidBody3D / CharacterBody3D)
- 浮点全部换整数或定点小数
- 调试地狱(两个客户端结果不一致时,要用帧号 + 输入序列逐步对比)
- 一个客户端卡了,全员等他(锁步 lockstep),否则后面帧会错位

> 💡 **小贴士**:**这就是为什么格斗/RTS 厂商都自己造轮子** — Riot 的 LOL 自己撸物理、暴雪的星际争霸 2 自己撸、Valve 的 Dota 2 自己撸。Godot 没提供这套,是因为它定位是"全能 3D 游戏引擎"而不是"竞技专用引擎"。

## 四、决策树:我的游戏该用哪个?

```
你的游戏类型?
│
├─ FPS / TPS 射击 ────────→ 状态同步 ✅(命中判定要在服务器)
├─ MMO / 大世界 RPG ─────→ 状态同步 ✅(玩家上千,服务器算不过来也得算)
├─ 合作沙盒(我的世界类) ─→ 状态同步 ✅(区块按需同步,玩家少)
├─ MOBA(LOL、王者荣耀) ──→ 状态同步 ✅(虽然感觉是帧同步,但实际是状态同步+回放)
├─ 格斗(街霸、罪恶装备) ─→ 帧同步 ✅(帧操作要 1:1 精确,玩家少)
├─ RTS 即时战略 ─────────→ 帧同步 ✅(单位多,带宽要省)
├─ 策略战棋(火纹、圣女) ─→ 帧同步 ✅(格子制 + 回合制)
└─ 卡牌 / 棋牌 / 桌游 ──→ 状态同步(其实不用同步物理,只同步牌局状态)
```

**经验法则**:
- 玩家多(>10) → 状态同步(服务器算不过来)
- 玩家少(<8) + 操作密集 → 帧同步
- 不确定 → 状态同步(实现简单,后期再升级)

## 五、坑点速查表(踩过的都标红)

| 坑 | 症状 | 解决 |
|---|---|---|
| Replication Editor 找不到"+ Add" | 同步列表是空的,多人模式下别人看不到你动 | Inspector → Replication → 点 "Replication Editor" 子面板里的 "+ Add Property" |
| `MultiplayerSynchronizer` 加了但不同步 | Client 看到对方"瞬移"或不显示 | 检查 Sync = true,且属性在 Spawn = true 列表里 |
| 服务器和客户端物理结果不一致 | 客户端看到 A 在 (1,0,2),服务器算 A 在 (1.0001, 0, 2) | 服务器权威 — 服务器算,客户端只显示 |
| 帧同步中两个客户端结果慢慢漂移 | 跑 10 秒后两个客户端位置差 0.5 米 | 全用整数 / 定点小数,固定步长物理,自己 seed 随机数 |
| 一个客户端卡了,所有人等他 | 帧同步的 lockstep 代价 | 用**乐观锁步**:延迟的玩家最多 3-5 帧,超过就断开 |
| 状态同步 + RigidBody3D,服务器和客户端物理漂移 | 客户端看到箱子在地上,服务器算箱子在悬浮 | 服务器权威物理,客户端不跑 `RigidBody3D`,只接收 transform |

## 六、延伸阅读

1. **Gaffer On Games - What Every Programmer Needs To Know About Game Networking**:<https://gafferongames.com/post/what_every_programmer_needs_to_know_about_game_networking/> — Glenn Fiedler(光环、命运网络架构师)写的联机入门圣经,全行业公认最好的科普,**强烈建议从这篇开始**。
2. **Glenn Fiedler - Networked Physics in Unity (Deterministic Lockstep)**:<https://gafferongames.com/post/networked-physics-in-unity/> — 同作者,讲帧同步的确定性物理怎么搞,虽然写的是 Unity,**确定性数学部分完全通用**。
3. **Godot 官方文档 - MultiplayerSynchronizer**:<https://docs.godotengine.org/en/stable/classes/class_multiplayersynchronizer.html> — Godot 4 状态同步的官方文档,所有可配置项 + 信号,做复杂项目必查。

---

> **软件版本**:本文基于 Godot 4.6-stable(2026-01-26 发布)撰写。  
> **数据来源时间**:2026-06-21(参考 Godot 官方文档 4.6 分支、Gaffer On Games 系列、Quora / Reddit r/gamedev 关于"frame vs state sync"的高赞讨论交叉验证)。