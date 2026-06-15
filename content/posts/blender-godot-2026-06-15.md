---
title: "Blender & Godot 3D 教程 - 2026-06-15 - Godot 4 联机架构总览:从单机到多人,先选对你的网络模型"
date: 2026-06-15T04:00:00+08:00
description: "零基础搞懂联机游戏怎么做的:四大网络模型(C/S / P2P / Listen / Dedicated)、状态同步 vs 帧同步、权威服务器、Godot 4 高层联机 API,30 分钟做出第一个能联机的 3D Demo。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-15.png
  alt: "两台电脑通过发光网络线相连,各显示同一 3D 联机游戏场景"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-15.png)

> 系列第四篇。前三篇我们搞定了「Blender 建模 → Godot 4 加载 3D 场景」。今天切到本系列真正的终点——**联机游戏**。但和大多数教程一上来就教 API 不同,**今天我们先聊架构**。因为没有架构概念,你后面写的 `MultiplayerSpawner.spawn()`、`MultiplayerSynchronizer` 配置就是一堆魔法咒语;有了架构,你会发现每行代码都是为了解决某个具体的网络问题。

## 一、为什么从架构开始,不直接上 API?

在 Blender 教程里,我们从「界面」开始;在 Godot 教程里,我们从「Node / Scene」开始;**联机这个主题里,「网络模型」就是它的「界面」**。如果你跳过这步,直接抄一段:

```gdscript
var peer := ENetMultiplayerPeer.new()
peer.create_server(7777, 4)
multiplayer.multiplayer_peer = peer
```

跑起来——咦,真的能连两个窗口?但你完全不知道:
- **谁是服务器?** 怎么判定?
- **谁说了算?** 玩家 A 说他现在位置是 (100, 0, 100),其他玩家信不信?
- **延迟 100ms 怎么办?** 玩家 B 看到 A 永远落后 6 帧?
- **作弊怎么办?** 玩家 A 直接改包说自己无敌?

这些问题不来自 Godot,它们来自**所有联机游戏都必须做的取舍**。今天的任务:把这套取舍讲清楚,让你看后面任何联机教程(包括明天的 API 实战)时,**每行代码都能映射到具体的设计决策上**。

## 二、四个最常见的网络模型

联机游戏发展 30 年,业界沉淀出 4 种主流网络拓扑,先看图后看字:

### 2.1 C/S 客户端-服务器模型

```
     [玩家 A]            [玩家 B]
        |                   |
        |   UDP/TCP 包      |
        +-------> [服务器] <+
                   |
                   | 60Hz 推送世界状态
                   v
        [玩家 A]            [玩家 B]
```

**谁说了算:服务器**。两个玩家之间**不能直接通信**,所有消息必须经过服务器中转。

- **优点**:反作弊简单(所有计算都在服务器做)、状态唯一(不会出现"你看到我在山头,我看到我在树后"的鬼故事)
- **缺点**:服务器带宽和 CPU 是瓶颈,玩家越多越贵;延迟 = 客户端 ↔ 服务器 × 2
- **典型游戏**:**几乎所有大型多人游戏**——MMO、LOL、王者荣耀、CS、Valorant、PUBG

### 2.2 P2P 点对点模型

```
    [玩家 A] <-----> [玩家 B]
       ^                 ^
        \                /
         \              /
          \            /
           [玩家 C]
       A-B、A-C、B-C 全互联
```

**谁说了算:大家都是服务器**。每个客户端同时是服务器,直接把状态广播给其他所有客户端。

- **优点**:延迟最低(玩家 B 离 A 最近时,直接走,不需要绕远路);无服务器成本
- **缺点**:玩家越多带宽按 N² 爆炸(8 人 P2P = 28 条链路);**反作弊几乎不可能**(任何客户端都能作弊)
- **典型游戏**:本地多人合作、老游戏局域网、《星际争霸 1》、**主机平台本地对战**

### 2.3 Listen Server(房主模式)

```
    [玩家 A = 房主 = 服务器]
       /                \
      / 局域网 / 公网    \
     v                   v
[玩家 B]            [玩家 C]
```

**谁说了算:玩家 A**(他既是玩家又是服务器)。A 退出 = 整个游戏崩。

- **优点**:搭伙开黑零成本,适合朋友间临时玩
- **缺点**:A 的带宽/CPU 要扛所有客户端;A 一断网全员掉线;公网下 A 必须做端口转发
- **典型游戏**:Minecraft 基岩版局域网、CSGO 休闲模式、Stardew Valley 多人、**绝大多数独立游戏**

### 2.4 Dedicated Server(专用服务器) + Relay(中继)

```
    [玩家 A] [玩家 B] [玩家 C] [玩家 D]
        \      |       |      /
         \     |       |     /
          \    |       |    /
           [专用服务器机房]
                OR
           [中继服务(Steam Relay / Epic / Photon)]
```

**Dedicated Server**:服务器跑在机房 24 小时,玩家随便来随便走。
**Relay(中继)**:玩家之间不直接互联,所有包都过一台云服务器(类似 C/S,但云服务器只做转发不做计算)。

- **优点**:Dedicated = 最稳定;Relay = 玩家不需要公网 IP、不需要做端口转发
- **缺点**:Dedicated = 你要付月费租机子;Relay = 厂商抽成 + 依赖第三方
- **典型游戏**:Dedicated = CSGO 竞技服、Minecraft 服务器;Relay = **绝大多数主机多人游戏**(PS/Xbox/Switch 联机)、Photon SDK 接入的游戏

> **小贴士**:我们这个系列是教程级,接下来 5-6 篇**全部用 Listen Server 模式**——一个玩家开服,其他玩家加入。原因:零成本(不用租服务器)、零配置(同一局域网即可)、Godot 一行 `create_server()` 搞定。等你做完 Demo 想去公网玩,**最便宜的过渡就是换 Relay**(如 Hamachi、Radmin VPN,或者商用服务如 Photon)。

## 三、状态同步 vs 帧同步(联机游戏最核心的选型)

确定了"用 C/S 还是 P2P",下一步是"**同步什么**"。这是联机游戏最关键的架构决策之一。

### 3.1 状态同步(State Synchronization)

**思路**:每帧把每个对象的位置、旋转、血量、状态**当作一坨数字**打包发给对方。

```
服务器 tick @ 60Hz:
  玩家 A: position=(1.0, 0.5, 2.0), hp=100, state="walking"
  玩家 B: position=(-2.0, 0.5, 1.5), hp=80,  state="attacking"
  怪物  : position=(0.0, 0.0, 5.0), hp=50, state="idle"
→ 一次性序列化,广播给所有客户端
```

- **优点**:实现简单、断线重连容易(把当前世界状态重发一遍就行)、逻辑可以复杂(服务器跑得动就行)
- **缺点**:带宽大(每个对象每帧都要发)、服务器 CPU 重(60Hz × N 个对象 × M 个玩家)
- **典型游戏**:几乎所有 FPS、RPG、MMO、绝大多数现代多人游戏

### 3.2 帧同步(Frame Synchronization / Lockstep)

**思路**:**不传当前状态,只传操作**。所有客户端收到相同的输入序列,**自己跑相同的物理模拟**,得到相同的输出。

```
服务器 tick @ 30Hz:
  frame 1234: player_A_input = "W", player_B_input = "A"
  frame 1235: player_A_input = "W+W", player_B_input = "idle"
  ...
→ 每个客户端自己跑物理:
    A 按 W → 加速 → 移动
    B 按 A → 左转
→ 所有客户端结果完全相同(因为物理是确定性的)
```

- **优点**:带宽极小(只传输入)、看视频回放几乎零成本(录输入序列即可)
- **缺点**:**物理必须严格确定**(浮点误差、随机数、碰撞顺序都必须可重现——这个门槛很高)、一个客户端卡了全员等他(锁步 lockstep)、断线重连要回溯 N 帧
- **典型游戏**:**格斗游戏(街霸、罪恶装备)**、**RTS(星际争霸 2、魔兽争霸 3)**、**MOBA(LOL、王者荣耀其实是状态同步)**

### 3.3 怎么选?

| 游戏类型 | 推荐方案 | 原因 |
|---|---|---|
| FPS / TPS 射击 | **状态同步** | 子弹命中是即时事件,客户端预测 + 服务器调和 |
| MMO RPG | **状态同步** | 玩家上千,服务器必须能"丢弃"不重要的状态 |
| 格斗 / 横版动作 | **帧同步** | 玩家少(2-8 人)、帧操作要 1:1 精确 |
| RTS 即时战略 | **帧同步** | 几百单位同屏,带宽带不起 |
| MOBA | 状态同步为主 + 部分帧同步 | LOL 其实是用状态同步做的(回放系统自己录世界状态) |
| 合作沙盒(我的世界) | **状态同步 + 区块按需同步** | 玩家少、地图巨大 |

> **小贴士**:**本系列用状态同步**。明天我们要写的 `MultiplayerSynchronizer` 就是状态同步的官方实现。Godot 的帧同步支持较弱,真要做帧同步你得自己写确定性物理,这是另一个量级的工作。

## 四、权威服务器(Authoritative Server)——为什么不能信客户端?

假设你做了状态同步,**但是允许客户端随便改自己的位置**——

```
[玩家 A 客户端] → 服务器: 我现在 (999999, 0, 999999)
服务器: 好的,我把 A 放到那里
→ 全服玩家看到 A 瞬移到了地图外
```

这就是**无权威模式**——`Move and Sync` 模式,在 P2P 老游戏里很常见,现在的游戏几乎不用了,因为**作弊泛滥**。

**权威服务器模式**:服务器是唯一"真相"——
1. 客户端告诉服务器:**"我要往右走"**(意图,不是结果)
2. 服务器跑物理模拟,**算出 A 应该在哪里**
3. 服务器把"算出来的位置"广播给所有客户端
4. 客户端老老实实显示服务器算的位置(就算本地物理算出来不一样)

```
[客户端 A] → 服务器: 我按了 W (input=W)
[服务器]   → 所有客户端: A 现在的位置是 (1.05, 0.5, 2.1)
[客户端 B] → 收到后,把 A 的位置强设为 (1.05, 0.5, 2.1) 然后显示
```

- **优点**:反作弊,所有计算都在服务器,**客户端只管显示**
- **缺点**:延迟感强(从按 W 到看到自己动 = 网络往返时间),所以大型游戏做"**客户端预测**":客户端先按自己的物理动起来,服务器算完再"调和"(如果算的不一样就拉回去)。

> **小贴士**:本系列的 Demo 走**"客户端权威 + 服务器转发"** 模式——简单说,**每个客户端控制自己的角色,服务器把位置广播给其他人**。严格说不算真正的"权威服务器",但**对教程级足够**,真要做竞技再升级。

## 五、Godot 4 联机技术栈速览

现在概念有了,**Godot 4 怎么落地的**?

### 5.1 三层架构图

```
┌─────────────────────────────────────────────────┐
│ 你的 GDScript 代码(游戏逻辑)                       │
│   ↑ / ↓ 调 spawn()、监听信号、读写属性                │
├─────────────────────────────────────────────────┤
│ MultiplayerSpawner / MultiplayerSynchronizer      │
│   "高层节点": 帮你自动同步节点的诞生/死亡和属性       │
│   ↑ / ↓ 用 RPC 调底层 API                          │
├─────────────────────────────────────────────────┤
│ MultiplayerAPI(挂在 SceneTree 上的全局对象)          │
│   ↑ / ↓ 用 ENet / WebSocket / WebRTC 收发包        │
├─────────────────────────────────────────────────┤
│ 操作系统 UDP/TCP 套接字                              │
└─────────────────────────────────────────────────┘
```

新手阶段你只需要关心**前两层**:写 GDScript + 摆节点。第三、四层引擎已经搞定。

### 5.2 三个底层 MultiplayerPeer(选一个用就行)

| Peer | 协议 | 默认端口 | 浏览器支持 | 适用场景 |
|---|---|---|---|---|
| **`ENetMultiplayerPeer`** | **UDP**(可靠 + 不可靠通道) | 任意 | ❌ | **90% 游戏的默认选择**,局域网 + 跨网都行 |
| `WebSocketMultiplayerPeer` | TCP over HTTP | 80/443 | ✅ | **网页游戏**、跨平台、移动端 |
| `WebRTCMultiplayerPeer` | P2P over UDP | 任意 | ✅ | 浏览器 P2P、低延迟,需要 STUN/TURN |

> **小贴士**:**我们这个系列 100% 用 ENetMultiplayerPeer**。它在 Godot 4 是默认 peer,API 最简单,文档最全。WebSocket/WebRTC 留到你需要做网页版本或低延迟 P2P 时再学。

### 5.3 两个高层节点(本系列的 MVP)

**`MultiplayerSpawner`**:节点的"生/死"在所有客户端自动同步。
- 你在服务器 `spawner.spawn(数据)`,**所有客户端自动 instantiate 同样的节点**
- 你在服务器 `node.queue_free()`,**所有客户端自动 delete**

**`MultiplayerSynchronizer`**:节点的属性在所有客户端自动同步。
- 服务器改了 `position`,**所有客户端的 position 跟着变**
- 客户端改了 `hp`,**服务器和其他客户端都能看到**

> **小贴士**:这两个节点是 Godot 4.x 引入的,3.x 没有。3.x 时代要么手写 RPC,要么用第三方插件。**这是我们选 Godot 4 的核心理由之一**。

## 六、实战:30 分钟做出第一个联机 3D Demo

**目标**:两个 Godot 4.6 窗口(一个 Host、一个 Client)同时打开,各看到一个红色立方体,**按 WASD 控制自己的立方体移动,对方能实时看到你动**。预计 30 分钟。

### 6.1 新建项目

1. 启动 Godot 4.6 → New Project → 路径 `~/godot-projects/04-multiplayer-intro`,Renderer 保持 Forward+
2. 项目打开后,自动进 3D 视口
3. 顶部菜单 `Scene → New Scene` → 选 `Node3D` → Create
4. 选中根节点,Inspector 顶部 Name 改成 `Main`,`Ctrl + S` 保存为 `main.tscn`

### 6.2 搭建 3D 场景(地面 + 阳光 + 相机)

1. Scene 面板选中 `Main` → 加 `WorldEnvironment` 节点 → Inspector → Environment → New Environment → Background 选 Sky → Sky → New Sky(蓝天搞定)
2. 加 `DirectionalLight3D` 节点,改名 `Sun`,Inspector 勾上 Shadow Enabled
3. 加 `Camera3D` 节点,Position `(0, 8, 12)`,Rotation X `-40°`(斜俯视)
4. 加 `MeshInstance3D` 节点,改名 `Ground`,Mesh 选 New PlaneMesh,Scale `(3, 1, 3)`(30×30 米地板)

### 6.3 搭玩家场景(player.tscn)

1. `Scene → New Scene` → 选 `CharacterBody3D` → Create → 改根名 `Player` → `Ctrl+S` 保存为 `player.tscn`
2. 加 `MeshInstance3D` 子节点,改名 `Mesh`,Mesh 选 New BoxMesh,Material 选 New StandardMaterial3D,**Albedo Color 改红色**
3. 加 `CollisionShape3D` 子节点,Inspector → Shape → New BoxShape3D,Size `(1, 1, 1)`
4. 加 `MultiplayerSynchronizer` 子节点(关键!等下配置)
5. 加脚本:FileSystem 面板右键 → New Resource → GDScript → 文件名 `player.gd` → 双击打开,把下面代码**整段粘贴**:

```gdscript
extends CharacterBody3D

@export var speed: float = 6.0

func _physics_process(_delta: float) -> void:
    # 关键:只有"我自己的玩家"才接受输入,别人的玩家节点跳过
    if not is_multiplayer_authority():
        return
    var input_vec: Vector2 = Input.get_vector(&"move_left", &"move_right", &"move_up", &"move_down")
    var direction: Vector3 = Vector3(input_vec.x, 0, input_vec.y)
    velocity = direction * speed
    move_and_slide()
```

`Ctrl+S` 保存,**回到 player.tscn**,选中根节点 `Player` → Inspector 底部 Script 字段 → 选 `player.gd`。

**配置 MultiplayerSynchronizer**:
1. 选中 `MultiplayerSynchronizer` 节点
2. Inspector 找到 **Replication Editor**(在 MultiplayerSynchronizer 分类下)
3. 点 `Replication Editor` → 弹出窗口
4. 左边 `Sync` 面板 → 点 `+` → 选 `Player:Mesh:position` → 添加
5. 左边 `Sync` 面板 → 再点 `+` → 选 `Player:Mesh:rotation` → 添加
6. 关掉窗口,`Ctrl+S` 保存 player.tscn

> **为什么要这样配**:`MultiplayerSynchronizer` 默认**什么都不同步**——你得显式告诉它同步哪些属性。`Player:Mesh:position` 意思是"玩家根节点下的 Mesh 子节点的 position 属性"。

### 6.4 输入映射(Project Settings)

1. 顶部菜单 `Project → Project Settings` → 切到 `Input Map` 标签
2. 添加四个动作,每个 `+ Add` 后输名字 + 点 `+` 绑定按键:
   - `move_up` → 键盘 `W`
   - `move_down` → 键盘 `S`
   - `move_left` → 键盘 `A`
   - `move_right` → 键盘 `D`
3. 关掉设置窗口

### 6.5 搭主场景的网络层(回到 main.tscn)

1. 选中 `Main` 节点 → 加子节点 `MultiplayerSpawner`(就是这名字)
2. 选中 `MultiplayerSpawner`:
   - Inspector → `Spawn Path` → 点 "Assign" → 选 `Main`(后面所有 spawn 的节点都加在 Main 下)
   - Inspector → `Spawnable Scenes` → 点 `+` → 选 `player.tscn`
3. 加一个 `CanvasLayer` 子节点(用来放 UI),改名 `UI`
4. `UI` 下加 `Control` 子节点(全屏铺满,布局会自动适配)
5. `Control` 下加三个子节点:
   - `LineEdit`(改名 `AddressInput`,placeholder 写 "127.0.0.1")
   - `Button`(改名 `HostButton`,text 写 "Host (开服)")
   - `Button`(改名 `JoinButton`,text 写 "Join (加入)")
   - `Label`(改名 `StatusLabel`,text 写 "Disconnected")
6. 选中 `Main` 节点,挂脚本 `main.gd`:

```gdscript
extends Node3D

const PORT: int = 7777
const MAX_PLAYERS: int = 4
const PLAYER_SCENE: PackedScene = preload("res://player.tscn")

@onready var address_input: LineEdit = $UI/Control/AddressInput
@onready var status_label: Label = $UI/Control/StatusLabel
@onready var spawner: MultiplayerSpawner = $MultiplayerSpawner


func _ready() -> void:
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    multiplayer.connected_to_server.connect(_on_connected_to_server)
    multiplayer.connection_failed.connect(_on_connection_failed)
    multiplayer.server_disconnected.connect(func(): status_label.text = "Server disconnected")
    $UI/Control/HostButton.pressed.connect(_on_host_pressed)
    $UI/Control/JoinButton.pressed.connect(_on_join_pressed)


func _on_host_pressed() -> void:
    var peer: ENetMultiplayerPeer = ENetMultiplayerPeer.new()
    var err: int = peer.create_server(PORT, MAX_PLAYERS)
    if err != OK:
        status_label.text = "无法创建服务器,err=%d" % err
        return
    multiplayer.multiplayer_peer = peer
    status_label.text = "Hosting on :%d (我是 peer 1)" % PORT
    # 服务器自己(peer_id = 1)也要 spawn 一个 player
    _spawn_player(1)


func _on_join_pressed() -> void:
    var peer: ENetMultiplayerPeer = ENetMultiplayerPeer.new()
    var addr: String = address_input.text.strip_edges()
    if addr.is_empty():
        addr = "127.0.0.1"
    var err: int = peer.create_client(addr, PORT)
    if err != OK:
        status_label.text = "无法连接,err=%d" % err
        return
    multiplayer.multiplayer_peer = peer
    status_label.text = "Connecting to %s:%d ..." % [addr, PORT]


func _on_connected_to_server() -> void:
    var my_id: int = multiplayer.get_unique_id()
    status_label.text = "Connected! (我是 peer %d)" % my_id
    # 服务器会通过 spawner 自动给我们发一个 player,不用自己 spawn


func _on_connection_failed() -> void:
    status_label.text = "Connection failed"


func _on_peer_connected(id: int) -> void:
    print("Peer %d connected" % id)
    if multiplayer.is_server():
        _spawn_player(id)


func _on_peer_disconnected(id: int) -> void:
    print("Peer %d disconnected" % id)


func _spawn_player(id: int) -> void:
    var spawn_pos := Vector3(randf_range(-3.0, 3.0), 0.5, randf_range(-3.0, 3.0))
    # 用 spawn() 自定义数据,MultiplayerSpawner 会把数据传到所有客户端
    spawner.spawn({
        "peer_id": id,
        "spawn_pos": spawn_pos,
    })


# 这个函数名是约定:MultiplayerSpawner 调用时每个客户端都会执行
func _init_player_from_spawn_data(node: Node, data: Variant) -> void:
    node.name = str(data.peer_id)
    node.position = data.spawn_pos
    # 关键:把 player 节点的"权威"设成对应的 peer_id
    # 这样只有那个玩家能控制自己的角色,其他玩家控制不了
    node.set_multiplayer_authority(data.peer_id, true)
```

**挂 spawn_function**:选中 `MultiplayerSpawner` 节点 → Inspector 找到 `Spawn Function` → 选 `Main` 节点上的 `_init_player_from_spawn_data` 方法(Godot 会弹出一个选择器,选 `Main` → `_init_player_from_spawn_data`)。

`Ctrl+S` 保存,`F5` 跑起来,你应该看到地面 + 蓝天 + UI 按钮。

### 6.6 联机测试

**最关键的步骤**——开两个 Godot 实例:

1. **当前窗口先按 Host 按钮** → 看到状态 "Hosting on :7777",并且场景里出现一个红色立方体(自己的 player)
2. **再开一个 Godot 窗口**:`File → Open Project` 选同一个 `project.godot` 路径,Godot 允许同一项目多开
3. **第二个窗口按 Join 按钮** → 状态变 "Connecting..." → 几秒后变 "Connected!"
4. **第二个窗口的场景里也应该出现一个红色立方体**
5. **两个窗口各按 WASD** → **对方窗口里那个立方体跟着动**

**如果跨电脑玩**:把第二个窗口的 `AddressInput` 改成第一台机器的局域网 IP(Windows 在 cmd 跑 `ipconfig` 看 IPv4,macOS/Linux 跑 `ifconfig`/`ip addr`),并确保两台机器**在同一局域网**、防火墙**放行 UDP 7777 端口**。

> **坑点 1:第二个窗口按 Join 没反应?**
> 99% 是**防火墙**。Linux 跑 `sudo ufw allow 7777/udp`;macOS 第一次会弹窗问允不允许;Windows Defender 会问要不要放行,**选"允许"**。

> **坑点 2:两个窗口都有立方体,但移动不同步?**
> 检查 player.tscn 里的 `MultiplayerSynchronizer` 是否正确添加了 `Player:Mesh:position` 和 `Player:Mesh:rotation` 到 Sync 列表。**默认同步列表是空的**,必须手动加。

> **坑点 3:按 WASD 只在自己窗口能动,对方看到不动?**
> 大概率是 `is_multiplayer_authority()` 没生效。回到 player.gd,确保 `if not is_multiplayer_authority(): return` 这行在,**且 main.gd 里 `set_multiplayer_authority(id, true)` 传了正确的 peer_id**(可用 `print(multiplayer.get_unique_id())` 调试)。

## 七、常见坑点速查表

| 坑 | 症状 | 解决 |
|---|---|---|
| 第二个窗口连不上 | Join 按钮按下后没反应 / "Connection failed" | 防火墙放行 UDP 7777;两台机器同局域网;服务器 IP 输对 |
| 自己看不到 player,对方能看到 | spawn 函数没正确设置 spawn_pos | 检查 `node.position = data.spawn_pos` 是否在 `_init_player_from_spawn_data` 里执行 |
| 移动有延迟 + 卡顿 | 同步频率太低 / 包太大 | MultiplayerSynchronizer Inspector → Replication Interval 改 `0.05`(20Hz → 20Hz 默认就好,先排查其他问题) |
| 多人同时按 W,主机端看到乱跳 | **没设 authority** | `set_multiplayer_authority(peer_id, true)` 必须调用,且 authority 才有权改 position |
| `spawn()` 调用后服务器没收到 | spawn_path 没指对 | Inspector → `Spawn Path` 必须填**已经存在的节点路径**,通常填 `Main` 或 `../Players` 这种 |
| 跨网连不上(公网) | 服务器在内网,客户端在外网 | 需要**端口转发**(路由器)或**用 Hamachi / Radmin VPN**模拟局域网,或**用 Photon Relay**(零配置的公网方案) |
| 客户端退出后主机内存泄漏 | peer_disconnected 没清理 spawn 的节点 | 在 `_on_peer_disconnected` 里 `get_node("Players/%d" % id).queue_free()`,或用 `MultiplayerSpawner` 自动管理 |
| 服务器自己崩了,客户端也崩 | `server_disconnected` 没接 | `multiplayer.server_disconnected.connect(...)`,里面弹 UI 让用户回主菜单 |
| 浮点不同步(0.1 vs 0.10000001) | position 一直在跳 | MultiplayerSynchronizer 的属性 → 勾 `Delta` 模式而非 Always,引擎会自动过滤无变化的更新 |

## 八、今天的小结

- **网络模型 4 选 1**:C/S(默认)、P2P(老)、Listen(朋友开黑)、Dedicated(正式运营)
- **同步方式 2 选 1**:状态同步(主流,带宽大但易做) vs 帧同步(MOBA/格斗,带宽小但难做)
- **权威模式**:服务器是唯一真相,客户端别信(防作弊)
- **Godot 4 技术栈**:`MultiplayerAPI`(全局) → `MultiplayerPeer`(ENet/WebSocket/WebRTC) → `MultiplayerSpawner`/`MultiplayerSynchronizer`(高层节点)
- **本系列选择**:**Listen Server + 状态同步 + 客户端权威**(教程友好)
- **今天能跑的成果**:两个窗口同步移动红色立方体,代码 ~50 行

## 九、明天预告

下一篇进入 **Godot 4.6 - ENet 高层多人 API 实战**:把今天的 Demo 升级成"多个玩家 + 一个会动的 NPC",用 `MultiplayerSynchronizer` 同步**旋转和颜色**,用 RPC 同步**开火事件**(每个玩家按左键发射一颗小球,所有人看到)。同时**第一次聊延迟补偿**:客户端预测 + 服务器调和的最基础写法。

## 十、延伸阅读

1. **Gaffer On Games - What Every Programmer Needs To Know About Game Networking**:<https://gafferongames.com/post/what_every_programmer_needs_to_know_about_game_networking/> — Glenn Fiedler(光环、命运的网络架构师)写的联机入门圣经,**全行业公认最好的科普**,强烈建议从这篇开始。
2. **Godot 官方文档 - High-level multiplayer**:<https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html> — Godot 4 高层联机 API 官方教程,ENet 的所有细节(MTU、通道、丢包)都在这。
3. **Godot 官方文档 - MultiplayerSpawner / MultiplayerSynchronizer**:<https://docs.godotengine.org/en/stable/classes/class_multiplayerspawner.html> 和 <https://docs.godotengine.org/en/stable/classes/class_multiplayersynchronizer.html> — 两个高层节点的完整 API 参考,所有可配置项 + 信号,做复杂项目时必查。

---

> **软件版本**:本文基于 Godot 4.6-stable(2026-01-26 发布)撰写。  
> **数据来源时间**:2026-06-15(参考 Godot 官方文档 4.6 分支、Gaffer On Games 经典网络系列、Godot Forums 与 Reddit r/godot 实战案例交叉验证)。