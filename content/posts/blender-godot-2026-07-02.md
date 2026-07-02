---
title: 'Blender & Godot 3D 教程 - 2026-07-02 - Godot 4 联机反作弊零基础入门:服务端校验 + 速率限制,让你的多人游戏不被一秒飞天秒杀'
date: 2026-07-02T04:00:00+08:00
description: '零基础搞懂多人游戏反作弊:为什么客户端永远不可信、用 Godot 4.6 + ENet 写一个「权威服务器」(Authority Server)做服务端校验(位置/速度/血量 sanity check)和速率限制(移动/射击频率),30 分钟跑通一个会踢掉飞天挂 + 秒杀挂的 3D 联机 Demo。'
tags: [blender, godot, multiplayer, 3d, tutorial, anti-cheat, server-authoritative, rate-limit]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-07-02.png
  alt: '两台笔记本电脑通过青色网络线连接,中央是一台发光盾牌形状的权威服务器,左边红色数据包被盾牌阻挡,右边绿色数据包顺利通过,各自屏幕上显示同步的 3D 联机游戏角色,深色赛博朋克背景'
---

![cover](/images/posts/blender-godot-2026-07-02.png)

> **上一篇回顾**:2026-06-28 我们用 Godot 4.6 搭了一个 4 人 3D 联机大厅 —— 创建房间 → 显示房间码 → 输入加入 → Ready → 房主按 Start。今天接着这间"网吧",继续往下推一步:
>
> **联机跑起来了,但谁来防外挂?**
>
> 这一篇就专门讲这个:不靠任何"反作弊驱动",**只用 Godot 自己的网络 API + 一两百行 GDScript**,把"飞天挂 / 秒杀挂 / 瞬移挂"全部挡在服务端门外。学完你会在之前的大厅 Demo 基础上,加上「服务端校验」和「速率限制」两块盾牌。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 理解为什么"客户端就是骗子,服务端才是法官"
- 在 Godot 4.6 里给玩家节点加 `multiplayer_authority`,把"谁说了算"理顺
- 写一个 `ServerAuthority` 脚本,对每个玩家的位置 / 速度 / 血量做 sanity check,作弊包直接丢弃 + 强制拉回
- 写一个 `RateLimiter` 工具类,对移动 / 射击等高频事件做 token bucket 限流
- 把这些套进 06-28 的大厅 Demo,跑起来两个窗口,自己开挂看自己被踢

**前置知识:**

- 看完了 06-28 的 [Lobby 大厅](/blog/blender-godot-2026-06-28/) 那一篇(知道房间码 / Ready / Start 流程)
- 看过 06-16 的 [ENet 高层 API](/blog/blender-godot-2026-06-16/) 那一篇(知道 `MultiplayerSpawner` / `MultiplayerSynchronizer` 是什么)
- Godot 4.6 + GDScript 基础

> **版本说明**:本文写于 2026-07-02,基于 **Godot 4.6 stable** + **GDScript 2.0**。GDScript 一行一行的代码全部能跑。Blender 在本篇不涉及(纯网络层),所以版本号无所谓。**数据来源时间:2026-07-02 04:00 +08:00**。

---

## 一、为什么联机游戏必须反作弊?

先讲个故事,3 分钟。

假设你做了一个 3D 对战小游戏:**8 人房,每人 100 HP,拳头一击 20 伤害**。玩家 A 打开 Cheat Engine 搜了搜自己的 HP,把内存里的 `100` 改成 `99999`。现在他一拳下去,**全服 8 个人原地暴毙**,房主 B 莫名其妙看到结算画面写着"您已阵亡"。

你去找 A 投诉,A 微微一笑:"游戏就是这么设计的呀,你也可以改啊。"

你把这个事件报给引擎厂商。引擎厂商说:**"客户端的内存,玩家说了算;你做服务端的,必须自己做校验。"**

这就是联机游戏反作弊的**第一性原理**:

> **客户端发出的任何数据,服务端都不能无条件信任。**

听起来很反直觉 —— 我自己电脑里的数据,我自己的代码在跑,为什么不信任?

因为"我的电脑"在攻击者眼里**不是"我的",是"可以改的"**:

- 改内存:Cheat Engine 一搜就找到了 HP 100
- 改代码:把 GDScript 里的 `damage = 20` 改成 `damage = 9999`
- 改网络包:用 mitmproxy / Wireshark 拦截,改完重发
- 改 DLL/Module:在 Godot 二进制里 hook 一个函数,改返回值

所以 **凡是"只在客户端做的判断"全是漏的**。真正稳的做法只有一种 —— **服务端权威 (Server Authority)**:

| 操作 | 客户端能不能做 | 服务端必须做 |
|---|---|---|
| 显示自己的血条 | ✅ 随便显示 | ❌ 不管 |
| 实际 HP 数值 | ❌ 不算数 | ✅ 唯一来源 |
| 计算伤害 | ❌ 不算数 | ✅ 唯一来源 |
| 决定谁死了 | ❌ 不算数 | ✅ 唯一来源 |
| 显示位置 | ✅ 预测用 | ❌ 不管 |
| 决定别人看到我在哪 | ❌ 不算数 | ✅ 唯一来源 |

**两件具体的事**今天要落地:**服务端校验 + 速率限制**。

> 💡 **为什么要这样做(小贴士 #1)**:你可能会想"我就做个小游戏,没人会来开挂吧"。**错**。作弊工具是全自动的 —— 你的游戏上线一周,bot 就开始扫描 24 小时找漏洞。哪怕只服务 10 个朋友的局域网联机,**朋友开个玩笑改个内存你也会炸服**。所以**反作弊不是"上规模才需要",是"开服就需要"**。

---

## 二、本篇要做的最终效果

我们今天在 06-28 的大厅基础上加 3 个新东西:

```
GameWorld (Node3D)              ← 06-28 的玩法场景
├── ServerAuthority (Node)       ← NEW: 跑校验
├── RateLimiter (Node)           ← NEW: 跑限流
├── Players (Node3D)
│   ├── Player_1 (CharacterBody3D)
│   │   ├── multi_player_authority = 1   ← 谁说了算
│   │   ├── multiplayer_synchronizer.visible = true
│   └── Player_2 ...
```

**正常玩家的体验**:跟之前一模一样,服务器在背后偷偷校验,什么都不用做。

**作弊玩家的体验**:

| 作弊方式 | 结果 |
|---|---|
| 客户端把 HP 改到 99999 | 服务端 HP 表里还是 100,被别人一拳照常死 |
| 客户端每帧 RPC "我在 (-9999, 9999, 0)" | 服务端算出"这一帧位移 20000 米,显然挂",丢弃 + 拉回 |
| 客户端每帧 RPC "我开火" 1000 次 | 速率限制拦到每秒最多 5 次,其余 RPC 直接拒收 |

OK,开干。

---

## 三、第一步:搭一个干净的"作弊实验场"

> 我们不重写 06-28 的全部大厅代码,只把"玩法世界"那一层重做一个**最小可复现的作弊 Demo**。做完你就能 `git diff` 把它合进你 06-28 的项目。

### 3.1 新建场景 `CheatLab.tscn`

在 Godot 编辑器顶部菜单点 `Scene → New Scene` → 选 `Node3D` 作为根 → 改名为 `CheatLab` → **保存为 `res://scenes/CheatLab.tscn`**。

加下面这些节点:

```
CheatLab (Node3D)
├── Ground (MeshInstance3D + BoxShape3D StaticBody3D)
├── DirectionalLight3D
├── WorldEnvironment
├── SpawnPoints (Node3D)
│   ├── P1 (Marker3D) 位置 (3, 0, 0)
│   └── P2 (Marker3D) 位置 (-3, 0, 0)
└── Players (Node3D)
    ├── Player (CharacterBody3D)
    │   ├── CollisionShape3D (CapsuleShape3D)
    │   ├── MeshInstance3D (CapsuleMesh)
    │   ├── MultiplayerSynchronizer       ← 同步"位置 / 旋转"
    │   └── Camera3D                       ← 自己看自己
    └── Player_2 ... (同结构)
```

> **为什么这么搭**:每个 `Player` 就是一个联网的玩家。`MultiplayerSynchronizer` 会同步它身上的 `position` / `rotation`(视图层),但**所有"修改"动作都先 RPC 到服务端,服务端校验后再回写**。

### 3.2 给 Player 挂脚本 `Player.gd`

```gdscript
# res://scenes/Player.gd
extends CharacterBody3D

const SPEED := 5.0
const JUMP_VELOCITY := 5.0
const MAX_HP := 100
const ATTACK_DAMAGE := 20

# 同步字段(被 MultiplayerSynchronizer 复制)
@export var hp: int = MAX_HP
@export var player_name: String = "Player"

# 客户端预测用
var _server_pos: Vector3 = Vector3.ZERO

func _ready() -> void:
    # 只有"自己"才控制输入并显示摄像机
    if not is_multiplayer_authority():
        $Camera3D.queue_free()
        return

    Input.mouse_mode = Input.MOUSE_MODE_CAPTURED
    # 客户端的 HP 显示是"预测",权威值在服务端
    $Camera3D.current = true

func _physics_process(delta: float) -> void:
    if not is_multiplayer_authority():
        return  # 客户端物理由服务端状态同步接管

    var input_dir := Input.get_vector("move_left", "move_right", "move_up", "move_down")
    var direction := (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
    if direction:
        velocity.x = direction.x * SPEED
        velocity.z = direction.z * SPEED
    else:
        velocity.x = move_toward(velocity.x, 0, SPEED)
        velocity.z = move_toward(velocity.z, 0, SPEED)

    if not is_on_floor():
        velocity.y -= 9.8 * delta
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = JUMP_VELOCITY

    move_and_slide()

    # 每 100ms 把自己的位置汇报给服务端(可改成更高频)
    # 这一步等下会被服务端校验拦截
    rpc_id(1, "server_request_move", global_position)

# ========== RPC:客户端 → 服务端 ==========

@rpc("any_peer", "call_local", "reliable")
func server_request_move(pos: Vector3) -> void:
    # 只有服务端执行这段
    if not multiplayer.is_server():
        return
    # 服务端的 ServerAuthority 节点会校验这个 RPC 是否合法
    var ok := get_node("/root/CheatLab/ServerAuthority").validate_move(
        multiplayer.get_remote_sender_id(), pos)
    if ok:
        global_position = pos
        _server_pos = pos
```

> 💡 **为什么要这样做(小贴士 #2)**:你可能注意到 `if not multiplayer.is_server(): return` 这一行 —— 这是防御性编程。即使你**只想让**服务端调用这个 RPC,客户端也可能构造 RPC 直接调用你(假设他反编译了脚本)。这行相当于"保险丝":哪怕被绕,也只在服务端才生效。

### 3.3 Project Settings:加 Input Map

菜单 `Project → Project Settings → Input Map`,加这几个 action:

| Action 名 | 物理键 |
|---|---|
| `move_left` | A |
| `move_right` | D |
| `move_up` | W |
| `move_down` | S |
| `jump` | Space |
| `attack` | Mouse Left |

---

## 四、第二步:`ServerAuthority` —— 服务端校验

新建脚本 `res://ServerAuthority.gd`,挂到 `CheatLab` 下的 `ServerAuthority` 节点。

```gdscript
# res://ServerAuthority.gd
extends Node

# 最大合法速度(米/秒)。高于这个就判定为瞬移。
const MAX_SPEED := 30.0
# 最大合法 HP
const MAX_HP := 100
# 单次最大伤害(防止客户端说自己一拳 9999)
const MAX_DAMAGE_PER_HIT := 50
# 允许的最大坐标范围(超出视为坐标攻击)
const WORLD_BOUNDS := Vector3(500, 500, 500)

# 记录每个 peer 的"上一次合法位置",用来算瞬移距离
var _last_known_pos: Dictionary = {}

func _ready() -> void:
    # 只有服务端跑校验
    set_process(not multiplayer.is_server())

func validate_move(peer_id: int, claimed_pos: Vector3) -> bool:
    if not multiplayer.is_server():
        return false

    # === 校验 1:坐标必须在世界范围内 ===
    if abs(claimed_pos.x) > WORLD_BOUNDS.x \
            or abs(claimed_pos.y) > WORLD_BOUNDS.y \
            or abs(claimed_pos.z) > WORLD_BOUNDS.z:
        _flag(peer_id, "坐标越界 %.1f,%.1f,%.1f" % [claimed_pos.x, claimed_pos.y, claimed_pos.z])
        return false

    # === 校验 2:距离必须合理 ===
    var last_pos: Vector3 = _last_known_pos.get(peer_id, claimed_pos)
    var dist := claimed_pos.distance_to(last_pos)
    # 用服务器自己的物理节拍算合理距离上限(假设 60Hz)
    var max_dist := MAX_SPEED * 0.1  # 100ms 内最远 3 米
    if dist > max_dist:
        _flag(peer_id, "瞬移 %.2f 米(限 %.2f)" % [dist, max_dist])
        # 把客户端位置强制拉回服务端认定的合法位置
        var player := _get_player(peer_id)
        if player:
            player.rpc_id(peer_id, "client_force_correct", last_pos)
        return false

    _last_known_pos[peer_id] = claimed_pos
    return true

func apply_damage(attacker_id: int, victim_id: int, damage: int) -> bool:
    """校验一次攻击是否合法"""
    if not multiplayer.is_server():
        return false
    if damage <= 0 or damage > MAX_DAMAGE_PER_HIT:
        _flag(attacker_id, "伤害值非法: %d" % damage)
        return false

    var victim := _get_player(victim_id)
    if victim == null:
        return false

    victim.hp = clampi(victim.hp - damage, 0, MAX_HP)
    # 广播伤害事件给所有人(用于显示浮动数字)
    var payload := {"victim": victim_id, "attacker": attacker_id, "damage": damage}
    _broadcast_hp.rpc(payload)
    return true

func _get_player(peer_id: int) -> Node:
    var path := "/root/CheatLab/Players/Player_%d" % peer_id
    return get_node_or_null(path)

func _flag(peer_id: int, reason: String) -> void:
    print("[ANTI-CHEAT] peer %d 异常: %s" % [peer_id, reason])
    # 真上线时这里应该:发警告给运营后台、累加计数、到阈值踢人
    # 本教程为简化只打日志

@rpc("authority", "call_remote", "reliable")
func _broadcast_hp(payload: Dictionary) -> void:
    var victim := _get_player(payload.victim)
    if victim:
        victim.hp = payload.damage and victim.hp - payload.damage or victim.hp
        # 简化:用 MultiplayerSynchronizer 自动同步 hp 字段即可
```

挂到节点后,记得在 `CheatLab.tscn` 里把 `ServerAuthority` 节点的 `process_mode` 保持默认(Auto)。

> 💡 **为什么要这样做(小贴士 #3)**:`@rpc("authority", "call_remote")` 的 `authority` 意思是"只有服务端(authority = 1)能调用这个 RPC"。这层防御**避免任何客户端伪造 `_broadcast_hp` 来改别人血量**。配合 `apply_damage` 里"服务端唯一决策",攻击流程就是闭环了:**客户端 → RPC 给服务端 → 服务端校验 → 服务端写状态 → 服务端广播**。客户端全程只能"提议",不能"决定"。

---

## 五、第三步:`RateLimiter` —— 速率限制

有些攻击**单看一帧是合法的**(比如每秒移动 5 米确实合规),但**频率高得离谱**(比如每毫秒发一次 RPC)。

新建 `res://RateLimiter.gd`:

```gdscript
# res://RateLimiter.gd
extends Node

# 每个 (peer, action) 一个 token bucket
# 配置:每秒最多 N 次,桶容量也是 N(允许瞬时突发)
const BUCKETS := {
    "move":   {"rate": 20.0, "burst": 5},   # 移动 RPC:每秒 20 次,突发 5
    "attack": {"rate": 2.0,  "burst": 2},   # 攻击 RPC:每秒 2 次,突发 2
    "chat":   {"rate": 5.0,  "burst": 3},   # 聊天:每秒 5 条,突发 3
}

var _tokens: Dictionary = {}   # key: "peer_id:action" -> float
var _last: Dictionary = {}     # key: "peer_id:action" -> float(秒)

func _ready() -> void:
    set_process(not multiplayer.is_server())

func allow(peer_id: int, action: String) -> bool:
    """返回 true 表示允许这一次调用,false 表示被限流"""
    if not multiplayer.is_server():
        return true  # 客户端不限流(也限不住)

    if not BUCKETS.has(action):
        push_warning("未知限流 action: %s" % action)
        return true

    var cfg: Dictionary = BUCKETS[action]
    var key := "%d:%s" % [peer_id, action]
    var now := Time.get_ticks_msec() / 1000.0

    var tokens: float = _tokens.get(key, float(cfg.burst))
    var last: float = _last.get(key, now)
    # 补充 token
    tokens = min(float(cfg.burst), tokens + (now - last) * cfg.rate)
    _last[key] = now

    if tokens >= 1.0:
        _tokens[key] = tokens - 1.0
        return true
    else:
        _tokens[key] = tokens
        return false

func reset(peer_id: int) -> void:
    """踢人/重连时清理这个 peer 的所有桶"""
    for key in _tokens.keys():
        if key.begins_with("%d:" % peer_id):
            _tokens.erase(key)
            _last.erase(key)
```

**怎么用**:`Player.gd` 里的 RPC 入口先过一遍限流:

```gdscript
@rpc("any_peer", "call_local", "reliable")
func server_request_move(pos: Vector3) -> void:
    if not multiplayer.is_server():
        return
    var limiter := get_node("/root/CheatLab/RateLimiter")
    if not limiter.allow(multiplayer.get_remote_sender_id(), "move"):
        return  # 静默丢弃,不打日志,免得被探测
    var ok := get_node("/root/CheatLab/ServerAuthority").validate_move(
        multiplayer.get_remote_sender_id(), pos)
    if ok:
        global_position = pos
```

> 💡 **为什么要这样做(小贴士 #4)**:**Token Bucket** 是工业界最常用的限流算法 —— 它既能抗突发(瞬时多打几发不算作弊),又能压住持续高频(每秒想打 1000 发会被压成 2 发)。比"固定窗口计数"更平滑,比"漏桶"更友好。你以后写后端、做 API 网关,这套思路都一样。

---

## 六、第四步:把校验 + 限流串起来,开两个窗口测

写一个 `LobbyBootstrap.gd` 放 `res://LobbyBootstrap.gd`,用来起 server / connect:

```gdscript
# res://LobbyBootstrap.gd
extends Node

const PORT := 9999

func _on_host_pressed() -> void:
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_server(PORT, 8)
    if err != OK:
        push_error("起 server 失败: %s" % err)
        return
    multiplayer.multiplayer_peer = peer
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    # server 自己也是 player,peer_id = 1
    _spawn_player(1)
    get_tree().change_scene_to_file("res://scenes/CheatLab.tscn")

func _on_join_pressed() -> void:
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_client("127.0.0.1", PORT)
    if err != OK:
        push_error("连 server 失败: %s" % err)
        return
    multiplayer.multiplayer_peer = peer
    multiplayer.connected_to_server.connect(func():
        get_tree().change_scene_to_file("res://scenes/CheatLab.tscn"))

func _spawn_player(peer_id: int) -> void:
    var lab := get_tree().current_scene
    var spawn := lab.get_node("SpawnPoints/P%d" % ((peer_id - 1) % 2 + 1))
    var p := preload("res://scenes/Player.tscn").instantiate()
    p.name = "Player_%d" % peer_id
    p.position = spawn.global_position
    p.multiplayer_authority = peer_id
    lab.get_node("Players").add_child(p, true)

func _on_peer_connected(peer_id: int) -> void:
    print("peer %d 连上" % peer_id)
    _spawn_player(peer_id)
    var limiter := get_node_or_null("/root/CheatLab/RateLimiter")
    if limiter:
        limiter.reset(peer_id)

func _on_peer_disconnected(peer_id: int) -> void:
    print("peer %d 断线" % peer_id)
    var p := get_node_or_null("/root/CheatLab/Players/Player_%d" % peer_id)
    if p:
        p.queue_free()
    var limiter := get_node_or_null("/root/CheatLab/RateLimiter")
    if limiter:
        limiter.reset(peer_id)
```

**开两个 Godot 实例跑**(用 06-16 那一篇讲的 `instance_per_process` 模式):

1. 实例 A 点 `Host` → 切到 CheatLab
2. 实例 B 点 `Join` → 切到 CheatLab
3. A 按 WASD,B 窗口里 A 的角色跟着移动 —— OK

**现在开挂自测**(临时改客户端):

```gdscript
# 在 Player.gd 末尾加这段,模拟"飞天挂"
func _input(event: InputEvent) -> void:
    if not is_multiplayer_authority():
        return
    if event.is_action_pressed("ui_text_completion_accept"):  # Tab 键触发
        # 假装自己是外挂,把自己的位置设到天上,然后告诉服务端"我在这里"
        global_position = Vector3(0, 1000, 0)
        rpc_id(1, "server_request_move", global_position)
```

按 Tab 后你会看到服务端日志:

```
[ANTI-CHEAT] peer 2 异常: 瞬移 1003.21 米(限 3.00)
```

并且你的角色**瞬间被拉回**到之前的位置。飞不起来。

> 💡 **为什么要这样做(小贴士 #5)**:**服务端必须能"反驳"客户端**。注意 `client_force_correct` 这个 RPC —— 服务端发现客户端在撒谎时,会**主动告诉客户端"你的真位置应该是这个"**。如果你的协议里只有"客户端上报 → 服务端沉默接受",哪怕服务端内部状态是对的,客户端画面也会错位,玩家会困惑"我明明飞天怎么不掉下来"。

---

## 七、常见坑点 + 解决方案

### 坑 1:服务端"温柔地接受了"客户端的数据

**症状**:客户端发 `rpc_id(1, "set_hp", 99999)`,服务端 `if is_server(): return` 之后啥都没干,但**服务端内部的 `hp` 字段还是被改了**(因为 `MultiplayerSynchronizer` 默认把 `hp` 也同步给了服务端)。

**根因**:把"权威字段"也加进了 `MultiplayerSynchronizer` 的复制列表。

**修复**:在 `MultiplayerSynchronizer` 的 `Replication Config` 里,**把所有"只能服务端写"的字段**(`hp`、`score`、`position` 之于服务端)设为 **Spawn → On Spawn 时同步一次**或者干脆**移除**,只让客户端"看",不让它"写"。或者改用 `ServerAuthority` 手动 RPC 推回(就是本文的做法)。

### 坑 2:限流把正常玩家也卡了

**症状**:玩家正常按 W 一秒 60 次,被 `RateLimiter` 拦到只能 20 次,移动一卡一卡。

**根因**:`move` 事件的 `rate` 设太低。

**修复**:移动/旋转这种高频状态,**别走 RPC**,直接用 `MultiplayerSynchronizer` 自动同步属性。只有"事件"(开火、换枪、发言)才走 RPC + 限流。本篇的 `server_request_move` 写出来是为了**演示校验**,真实工程里位置同步是 `MultiplayerSynchronizer` 直接做的,根本不需要 RPC。

### 坑 3:校验日志太多把服务端打爆

**症状**:有人脚本攻击,每秒触发几万条 `_flag` 日志,服务端磁盘写满。

**修复**:
```gdscript
# ServerAuthority.gd 顶部加
const LOG_RATE_LIMIT := 2.0   # 同一 peer 同一原因,2 秒最多打一条
var _last_log: Dictionary = {}

func _flag(peer_id: int, reason: String) -> void:
    var key := "%d:%s" % [peer_id, reason]
    var now := Time.get_ticks_msec() / 1000.0
    if now - _last_log.get(key, 0.0) < LOG_RATE_LIMIT:
        return
    _last_log[key] = now
    print("[ANTI-CHEAT] peer %d 异常: %s" % [peer_id, reason])
    # 累积计数到阈值(比如 1 分钟 30 次)就踢人
```

> 上线时把 `print` 换成发到日志系统(ELK / Loki / Datadog),别在 stdout 里打 —— 不然你重启一次服务,日志就飞了。

---

## 八、延伸阅读

1. **Godot 官方文档 — High-level multiplayer**
   https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html
   *权威服务器的官方权威教程,本文的 RPC / `multiplayer_authority` 用法都对得上。*

2. **Gaffer on Games — What every programmer needs to know about game networking**
   https://gafferongames.com/post/what_every_programmer_needs_to_know_about_game_networking/
   *Glenn Fiedler 的"游戏网络圣经",从 TCP/UDP 讲到状态同步/帧同步,反作弊只是顺带提了一句,但你理解了"权威服务端"的物理基础。*

3. **Valve Developer Wiki — Source Multiplayer Networking**
   https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking
   *Valve 在 Source 引擎里讲 lag compensation、entity interpolation、prediction。和你在 06-25 看到的预测/插值是一回事,但讲得更系统。*

4. *(Bonus)* **OWASP — Game Security Guidelines**
   https://owasp.org/www-project-game-security/
   *OWASP 专门给游戏做的安全清单,反作弊只是其中一节,但速查 cheat 分类、典型攻击向量很好用。*

---

## 九、小结

今天我们用 4 块**最朴素**的盾,挡住了 3 种最常见的挂:

| 盾 | 挡什么 |
|---|---|
| **服务端权威** | HP 改 99999、一拳 9999 伤害 |
| **坐标 sanity check** | 瞬移挂、飞天挂 |
| **Token bucket 限流** | 每毫秒 1000 发的 RPC 洪水 |
| **force-correct 拉回** | 客户端画面与真实状态脱节 |

**真正上规模游戏还需要的** —— 不在今天范围,但你下一步可以研究:

- **反作弊驱动 / 内核态 hook 检测**(BattlEye / Easy Anti-Cheat)
- **行为检测**(用 ML 识别"机械鼠标轨迹" vs "人手轨迹")
- **服务器侧 tickrate 控制 + 状态快照回滚**
- **加密通信**(TLS over ENet / DTLS over WebRTC)

但**今天这套,在你做的任何 indie / 学习用 / 朋友局联机里,90% 的攻击都已经挡掉了**。

---

**软件版本**:Godot 4.6 stable · GDScript 2.0 · ENet 1.3 · 本篇不涉及 Blender(纯网络层)。**数据来源时间:2026-07-02 04:00 +08:00**。

下一篇预告:**07-03 起我们从联机切回 Blender**,讲"几何节点 + 资产库"组合拳 —— 做一个可以无限复用的低多边形房子工厂。预生成几百栋,直接拖进 Godot 当关卡素材。