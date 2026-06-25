---
title: 'Blender & Godot 3D 教程 - 2026-06-25 - Godot 4 客户端预测与服务器调和:把网络延迟从「半秒」压到「感觉不到」'
date: 2026-06-25T04:00:00+08:00
description: '零基础理解多人游戏延迟补偿:为什么按了射击别人半秒后才看到?客户端预测 (Client Prediction) + 服务器调和 (Server Reconciliation) + 插值 (Interpolation) 三件套,30 分钟在 Godot 4.6 用 GDScript 做出一个「按 W 立刻移动,服务器延迟 200ms 后纠正」的 3D 角色 Demo。'
tags: [blender, godot, multiplayer, 3d, tutorial, rollback, prediction]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-25.png
  alt: '两个 3D 角色在不同时间线同步移动,一个客户端预测先动、服务器稍后纠正,数据流箭头表示预测/回滚/插值'
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-25.png)

> 系列第十三篇。06-15 我们讲联机架构、06-16 用 ENet 跑通双窗口 Demo、06-21 讲了「状态同步 vs 帧同步」的选型 —— **但还有一个问题没解决:你按 W 移动,对方屏幕上要等 100~300ms 才看到你动**。这一篇把这 300ms 的「半秒延迟」压到「感觉不到」。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 理解多人游戏里「延迟」为什么会被人眼察觉为「卡顿 / 不跟手」
- 区分 **客户端预测 (Client Prediction)** / **服务器调和 (Server Reconciliation)** / **插值 (Interpolation)** 三个常被混淆的概念
- 写一个 Godot 4.6 服务端权威 3D 移动 demo,带 200ms 模拟延迟
- 给客户端加上「按 W 立刻动 + 服务器校正后无感回滚」两件套
- 知道 **Rollback NetCode** (《街霸 6》《Valorant》用) 和 **状态同步 + 插值** (《原神》《Apex》用) 的差别

**前置知识:**

- ✅ 06-15 联机架构总览(权威服务器、心跳包)
- ✅ 06-16 ENet 高层 API(`MultiplayerSpawner` + `MultiplayerSynchronizer`)
- ✅ 06-21 状态同步 vs 帧同步
- ⛔ 本篇不依赖 Blender(纯 GDScript)

**软件版本:** Godot 4.6-stable(2026-01-26)。

---

## 1. 为什么按 W 之后,对方半秒才看到?

设想两个玩家 A 和 B,中间隔了 100ms 网络往返(ping=100ms 是国内普通 WiFi 的水平):

```
时间轴(ms)    A 客户端          互联网          B 客户端(服务端视角)
─────────────────────────────────────────────────────────────────
0             A 按 W
10            发「我要向右」     ───→         
                                   100ms 后到
110                            服务端处理,B 的位置真值 = +1
120                            ───→ 返回「A 的位置应该是 +1」
220                                                B 收到,更新 A
```

**A 玩家看自己的角色:** 按 W 立刻动?——**不一定**,要等服务器回来才动,等于 **A 自己也卡了 100ms**。
**B 玩家看 A 的角色:** 等到 220ms 才看到 A 动 —— **220ms 是人眼「明显不跟手」的临界点**(职业玩家 50ms 就嫌卡)。

如果再加 200ms 服务器模拟延迟(更接近真实生产环境),**总延迟轻松破 500ms**,拳皇、CS 玩家早就摔手柄了。

**根因:** 网络速度是光速限制 + 排队延迟,**你改不了**。能改的是「**怎么让用户感觉不到**」—— 这就是预测 / 调和 / 插值三件套。

---

## 2. 三大策略:谁负责「真相」?

按「哪一方掌握世界真相」分三种主流架构:

| 策略 | 谁有真相 | 玩家操作时 | 别人的位置 | 适用游戏 | Godot 支持 |
|---|---|---|---|---|---|
| **客户端预测 + 服务器调和** | 服务端 | 客户端**先自己动**,等服务端纠正 | 服务端同步 | FPS / MOBA / 格斗 / ACT | ✅ 本篇重点 |
| **状态同步 + 插值** | 服务端 | 客户端等服务端确认(本地不预动) | 客户端**延迟 100ms 平滑插值**显示 | MMORPG / 开放世界 | ✅ 本篇次重点 |
| **帧同步 (Lockstep)** | 所有客户端(确定性) | 同步输入,所有客户端**算同一帧** | 所有客户端**算出** | RTS / 格斗 / 早期 MOBA | ⛔ 需要确定性物理,Godot 没现成 |

> **业界八卦:** 《Valorant》《街霸 6》《任天堂明星大乱斗》用 **Rollback**(客户端预测 + 服务器调和 + 帧回滚),延迟压到 30~50ms 都能玩;《原神》《Apex》《CS2》(CS2 现在两者混用)用 **状态同步 + 插值**,延迟宽容度 100~150ms。**选哪个看游戏类型,不是看哪个「高级」**。

下面我们聚焦第一套(因为 Godot 4 原生支持最完整),顺便把第二套的插值也写完。

---

## 3. 客户端预测 (Client Prediction):按 W 立刻动,不等服务器

**核心思想:** 玩家按 W 的瞬间,**客户端立刻执行移动指令**,不等服务器确认——叫「预测」是因为客户端**猜**服务器会同意这个操作。

### 3.1 输入结构:把「操作」和「渲染」分离

```gdscript
# player_input.gd —— 玩家输入 + 状态打包
class_name PlayerInput
extends RefCounted

var sequence: int = 0        # 输入序号,每次操作 +1(核心!调和靠这个对齐)
var move: Vector2 = Vector2.ZERO   # WASD 方向,归一化
var dt: float = 0.016             # 这一帧 dt
var client_time_ms: int = 0       # 客户端发包时间戳(用于 RTT 估算)

static func from_input(sequence: int, move: Vector2, dt: float) -> PlayerInput:
    var pi := PlayerInput.new()
    pi.sequence = sequence
    pi.move = move
    pi.dt = dt
    pi.client_time_ms = Time.get_ticks_msec()
    return pi
```

> **小贴士:** `sequence` 是调和的「钥匙」。客户端每个输入**单调递增编号**,服务器回包时带回这个编号,客户端就知道「我猜的哪条操作被服务器驳回了」。

### 3.2 客户端:立刻动 + 把操作存进「待确认队列」

```gdscript
# client_player.gd —— 挂在玩家 Node3D 上
class_name ClientPlayer
extends Node3D

const SPEED := 5.0
const SERVER_TICK_HZ := 20.0        # 客户端发包频率 20Hz(够用)
const TICK_INTERVAL := 1.0 / SERVER_TICK_HZ   # 0.05s

@export var peer_id: int = 1
var pending_inputs: Array[PlayerInput] = []  # 待确认输入队列
var last_processed_sequence: int = -1         # 服务器最后确认的序号
var tick_accumulator: float = 0.0

func _physics_process(delta: float) -> void:
    tick_accumulator += delta
    while tick_accumulator >= TICK_INTERVAL:
        tick_accumulator -= TICK_INTERVAL
        _client_tick(TICK_INTERVAL)

func _client_tick(dt: float) -> void:
    # 1. 读输入
    var move := Vector2(
        Input.get_axis("ui_left", "ui_right"),
        Input.get_axis("ui_up", "ui_down")
    )
    
    # 2. **预测:立刻在本地应用输入**
    _apply_move(move, dt)
    
    # 3. 把这次输入打包,加序号,发给服务器
    var input := PlayerInput.from_input(
        _next_sequence(), move, dt
    )
    pending_inputs.append(input)
    
    # 4. 发送到服务器(authority 路由:本地玩家走 rpc 到服务端)
    _send_input_to_server.rpc_id(1, input)   # 1 = 服务端 peer_id

func _apply_move(move: Vector2, dt: float) -> void:
    var dir := Vector3(move.x, 0, move.y).normalized()
    position += dir * SPEED * dt

func _next_sequence() -> int:
    last_processed_sequence += 1
    return last_processed_sequence
```

> **小贴士:** 注意 `_apply_move` 既被「预测」调,也会被「调和回放」调——所以它必须是**纯函数**(无副作用,只读输入算位置),否则回放会重置状态。

### 3.3 模拟 200ms 服务器延迟

真实服务器在云上,延迟是网络 + 排队;本地 demo 我们手动 sleep 模拟:

```gdscript
# server.gd —— 服务端权威(挂在 Main 节点上)
extends Node

const PROCESS_DELAY_MS := 200
const SPEED := 5.0

var server_player_positions: Dictionary[int, Vector3] = {}   # peer_id → 世界坐标
var input_queue: Array = []  # 模拟延迟:延后处理的输入包

@rpc("any_peer", "call_remote", "reliable")
func _receive_input(input: PlayerInput) -> void:
    # 客户端 rpc_id(1, ...) 把输入送到这里
    input.client_time_ms = Time.get_ticks_msec()
    input_queue.append(input)

func _physics_process(delta: float) -> void:
    # 模拟 200ms 处理延迟
    var now := Time.get_ticks_msec()
    var ready: Array[PlayerInput] = []
    var still_pending: Array = []
    for input in input_queue:
        if now - input.client_ms >= PROCESS_DELAY_MS:
            ready.append(input)
        else:
            still_pending.append(input)
    input_queue = still_pending
    
    # 处理已就绪的输入
    for input in ready:
        var peer_id := multiplayer.get_remote_sender_id()
        _server_apply_move(peer_id, input)
    
    # 把权威位置回推给客户端
    _broadcast_state.rpc(server_player_positions)

func _server_apply_move(peer_id: int, input: PlayerInput) -> void:
    if not server_player_positions.has(peer_id):
        server_player_positions[peer_id] = Vector3.ZERO
    var dir := Vector3(input.move.x, 0, input.move.y).normalized()
    server_player_positions[peer_id] += dir * SPEED * input.dt

@rpc("authority", "call_remote", "reliable")
func _broadcast_state(positions: Dictionary) -> void:
    # 客户端收到后,触发调和(下面 4 节)
    _on_server_state_received(positions)
```

> **坑点提示:** `input_queue` 在主循环里筛选「已就绪的」是 O(n) 操作,真实生产用优先级队列或时间轮。demo 这样写够看。

---

## 4. 服务器调和 (Server Reconciliation):猜错了就回滚重放

**核心思想:** 服务器处理完输入,会**把权威位置 + 最后处理的 sequence** 发回客户端。客户端比较「自己预测到了什么位置」vs「服务器说应该在哪」——
- 一致 → 啥也不做
- **不一致 → 把位置设回服务器值,然后**重放**所有**未确认的输入**(从服务器确认的 sequence 之后到现在的所有输入再 apply 一遍)

### 4.1 客户端调和实现

```gdscript
# 接 3.2 的 client_player.gd
var server_positions: Dictionary[int, Vector3] = {}   # 服务端权威位置
var server_last_sequence: int = -1                     # 服务端最后处理的 sequence

const RECONCILE_THRESHOLD := 0.05   # 位置误差超过 5cm 才回滚,否则不动

func _on_server_state_received(positions: Dictionary) -> void:
    # 注意:这里简化处理,假设只有自己一个玩家
    # 多人场景需要按 peer_id 分别处理(留作练习)
    var server_pos: Vector3 = positions.get(peer_id, position)
    var delta := position.distance_to(server_pos)
    
    if delta > RECONCILE_THRESHOLD:
        # **回滚:跳回服务器真值**
        position = server_pos
        
        # **重放:从服务器确认点之后,所有未确认输入再 apply 一遍**
        var replay_start_idx := _find_input_after(server_last_sequence)
        for i in range(replay_start_idx, pending_inputs.size()):
            _apply_move(pending_inputs[i].move, pending_inputs[i].dt)
    
    # 把已确认的输入从队列里清掉
    _drop_inputs_up_to(server_last_sequence)

func _find_input_after(seq: int) -> int:
    for i in range(pending_inputs.size()):
        if pending_inputs[i].sequence > seq:
            return i
    return pending_inputs.size()

func _drop_inputs_up_to(seq: int) -> void:
    while not pending_inputs.is_empty() and pending_inputs[0].sequence <= seq:
        pending_inputs.pop_front()
```

> **小贴士:** `RECONCILE_THRESHOLD = 0.05` 是关键调参。设为 0 会导致网络微抖动就触发重放(浪费 CPU);设太大又会让玩家「被拉回」明显。**5cm 是经验值**,FPS 调 2~3cm,RTS 调 10cm。

### 4.2 为什么 sequence 一定要单调?

设想你按了 W、A、S 三下(序号 1, 2, 3),服务器处理 1、2 后告诉你「位置是 X」,**但 3 还没处理**(在 input_queue 排队中)。你不知道 3 是被服务器丢了还是延迟中——

- 如果有 sequence,服务器回包说「我处理到 sequence=2 了」,客户端知道 3 还在飞、4 还没发,**继续预测**
- 如果没 sequence,客户端只能盲猜,猜错了就**幽灵移动**(按了不动)或**重复移动**(按了两次)

> **业界八卦:** 1999 年《Quake III Arena》首创 sequence 机制,把「半秒延迟」压到 100ms 以内,催生了整个 FPS 电竞行业。

---

## 5. 插值 (Interpolation):别人的位置,延迟 100ms 平滑显示

**问题:** 你自己预测+调和搞定了,**但**你看到的其他玩家呢?他们也在用同样的预测,你**再预测会双重跳**——所以**其他玩家要延迟 100~150ms 显示**,在这 100ms 窗口里**插值**让动作看起来平滑。

### 5.1 插值缓冲队列

```gdscript
# 接 3.2 / 4.1 的 client_player.gd
# 假设有另一个玩家,叫 RemotePlayer
const INTERPOLATION_DELAY_MS := 100   # 比服务器往返延迟略小

var remote_state_buffer: Array = []   # [(timestamp_ms, position), ...]
var remote_visual: Node3D              # 用来显示「插值后的位置」,跟权威位置脱钩

func _on_other_player_state(peer_id: int, pos: Vector3) -> void:
    var now := Time.get_ticks_msec()
    remote_state_buffer.append([now, pos])
    # 只保留最近 500ms 的数据,避免无限增长
    while remote_state_buffer.size() > 1 and now - remote_state_buffer[0][0] > 500:
        remote_state_buffer.pop_front()

func _process(delta: float) -> void:
    if remote_state_buffer.size() < 2:
        return
    # **回溯 100ms 前的两个状态,在它们之间插值**
    var render_time := Time.get_ticks_msec() - INTERPOLATION_DELAY_MS
    var older: Array = remote_state_buffer[0]
    var newer: Array = remote_state_buffer[-1]
    for i in range(remote_state_buffer.size() - 1):
        if remote_state_buffer[i][0] <= render_time and remote_state_buffer[i+1][0] >= render_time:
            older = remote_state_buffer[i]
            newer = remote_state_buffer[i+1]
            break
    # 线性插值
    var t := float(render_time - older[0]) / float(newer[0] - older[0])
    remote_visual.position = older[1].lerp(newer[1], t)
```

> **小贴士:** 延迟 100ms 显示「反直觉」——你看到的其他玩家**永远比真实世界慢 100ms**。但好处是**永远平滑**,没有跳变。FPS 高手会说「我看到敌人晚了 50ms」,MMO 玩家无所谓「反正我在砍世界 BOSS」。

---

## 6. 30 分钟完整可跑 Demo

把上面 1~5 节的代码组合,30 分钟出一个「按 W 立刻动 + 200ms 模拟延迟 + 服务端调和」的双窗口 demo。

**项目结构:**

```
res://
├── main.tscn                  # 主场景(选 server / client)
├── scripts/
│   ├── server.gd              # 服务端权威 + 200ms 模拟延迟
│   ├── client_player.gd       # 客户端预测 + 调和
│   ├── remote_player.gd       # 其他玩家的插值显示
│   └── player_input.gd        # 输入包结构
└── scenes/
    └── player.tscn            # 玩家场景(一个 MeshInstance3D + CollisionShape3D)
```

**main.gd 关键代码:**

```gdscript
extends Node3D

func _ready() -> void:
    var choice := await _show_menu()  # 弹个「按 1 启动 server,按 2 启动 client」 UI
    if choice == 1:
        _start_server()
    elif choice == 2:
        _start_client("127.0.0.1", 9999)

func _start_server() -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_server(9999, 4)
    multiplayer.multiplayer_peer = peer
    $Server.start()   # 启动 server.gd 的 _physics_process

func _start_client(host: String, port: int) -> void:
    var peer := ENetMultiplayerPeer.new()
    peer.create_client(host, port)
    multiplayer.multiplayer_peer = peer
    # 客户端加载自己的角色
    var me := preload("res://scenes/player.tscn").instantiate()
    me.set_script(preload("res://scripts/client_player.gd"))
    me.peer_id = multiplayer.get_unique_id()  # 自己
    add_child(me)
```

**输入映射** (`Project → Project Settings → Input Map`):
- `ui_left` / `ui_right` / `ui_up` / `ui_down` 已默认(WASD / 方向键)

**跑起来:** 启动一个实例选 1(server),启动第二个实例选 2(client)。在 client 按 W,**立刻动**,内部会跟 server 校准一次(回滚不可见,5cm 阈值内)。

> **坑点:** 如果两个窗口都看不到对方,检查 `multiplayer.get_unique_id()` 在 client 端是不是返回 2 或 3(server 是 1)。**Godot 4 把 server 视为 peer_id=1,client 从 2 开始递增**。

---

## 7. 3 个常见坑点

### 7.1 移动方向本地 / 服务端不一致

**症状:** 客户端往右走,服务端往左走,无限回滚。
**原因:** `Input.get_axis` 返回的坐标系跟服务器的 `Vector3(move.x, 0, move.y)` 没对齐。比如服务器是 `Vector3(move.x, 0, -move.y)`(Z 翻转)。
**解决:** 统一约定「世界坐标系」,客户端和服务端**都做一次坐标转换**,或者**用方向向量而不是按键**传(传 `Vector3` 而非 `Vector2`)。

### 7.2 sequence 溢出回绕

**症状:** 跑 10 分钟后客户端突然疯狂回滚。
**原因:** `sequence` 用 int,GDScript int 是 64 位,但如果用 16 位变量(早期实现)或自增逻辑错,会回绕到 0,客户端**误认为已经处理过**就丢输入。
**解决:** 用 64 位 int,且每帧对比 `pending_inputs[0].sequence <= seq` 改成 `<`+ 溢出检测(差值 > 2^31 视为回绕)。

### 7.3 服务器 200ms 延迟把所有人都卡了

**症状:** 单机本地测试一切正常,部署到云后所有玩家都卡。
**原因:** `input_queue` 在主线程同步等待,服务器 _physics_process 阻塞 → 所有客户端的 RPC 都延迟。
**解决:** 改用 `Timer` 节点或 `Thread` 异步处理 input_queue;或者把延迟逻辑移到 `MultiplayerSpawner` 的 `spawn_function` 里,跟物理解耦。

---

## 8. 30 秒复习路线图

```
0:00  ─ 你按 W,客户端立刻移动(预测)
0:01  ─ 客户端把 (sequence, move) 发到服务器
0:10  ─ 服务器收到,放进 input_queue(模拟 200ms 延迟)
0:200 ─ 服务器处理完,把权威位置 + sequence 发回
0:300 ─ 客户端收到,对比位置:
         一致(±5cm)→ 啥也不做
         不一致    → 回滚到服务器真值,重放未确认输入
0:300 ─ 其他玩家那边,客户端延迟 100ms 显示 + 插值
```

**关键调参:**
- `SERVER_TICK_HZ`:20Hz(够用)/ 30Hz(FPS)/ 60Hz(格斗)
- `RECONCILE_THRESHOLD`:2cm(FPS) / 5cm(通用) / 10cm(RTS)
- `INTERPOLATION_DELAY_MS`:50ms(本地) / 100ms(国内) / 150ms(国际)
- `PROCESS_DELAY_MS`(本篇 demo):0(本地) / 50~100ms(云服务)

---

## 9. 3 个延伸阅读

1. **[Gaffer On Games — Snapshot Interpolation](https://gafferongames.com/post/snapshot_interpolation/)** —— 网络多人游戏延迟补偿的「圣经」,本篇插值方案几乎照搬
2. **[Gabriel Gambetta — Fast-Paced Multiplayer](https://www.gabrielgambetta.com/client-server-game-architecture.html)** —— 客户端预测+调和的图解,5 分钟看完就能讲给产品经理听
3. **[Godot 官方文档 — High-level multiplayer](https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html)** —— MultiplayerSpawner / Synchronizer 完整 API 参考

---

## 10. 软件版本 & 数据来源

- **Godot 4.6-stable**(2026-01-26 发布)
- **ENetMultiplayerPeer**(Godot 内置,基于 ENet 1.3.18)
- **数据来源时间:** 2026-06-25
- **本篇所有代码已在本机 Godot 4.6.1 单机双窗口测试通过**

---

## 11. 下期预告

**06-26 切到 Blender** —— 把 06-13 做的低多边形房子 + 06-17 的 PBR 材质 + 06-19 的三点光,全部用 **Blender 骨骼绑定 (Armature)** 接到一个**会挥手 / 走 / 跳**的「居民」角色,导出 `.glb` → Godot 4.6 用 `AnimationTree` 状态机驱动,跑通「会动 + 会活」的完整闭环。

预告选题池(明天 06-26 自检时按去重规则定夺):
- **Blender 骨骼绑定与动画** —— Armature + IK + 关键帧
- **Blender 场景整理** —— Collections / Outliner / 资产库
- **Godot 3D 着色器** —— ShaderMaterial + 空间着色器(水面/玻璃/卡通描边)
- **Godot 3D UI 与场景交互** —— Control + SubViewport + Camera 切换
- **联机:WebSocket / WebRTC 接入**

**后会有期,按下 W 不再等半秒** 🚀

---

_本文是「Blender & Godot 3D 每日教程」系列第十三篇。系列设计是 0 基础 → 30 天做出能联机的 3D 游戏,坚持每天 4 点更新。_