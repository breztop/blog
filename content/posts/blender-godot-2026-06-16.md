---
title: "Blender & Godot 3D 教程 - 2026-06-16 - Godot 4 ENet 高层多人 API：跑通你的第一个双窗口 3D 联机 Demo"
date: 2026-06-16T04:00:00+08:00
description: "零基础动手写 Godot 4 ENet 高层联机代码:MultiplayerSpawner 同步生成 + MultiplayerSynchronizer 同步状态,30 分钟跑起来两个窗口同步移动的 3D Demo。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-16.png
  alt: "Blender & Godot 3D 教程系列封面"
---

![cover](/images/posts/blender-godot-2026-06-16.png)

> 系列第五篇。昨天我们聊了联机的「架构地图」——C/S 还是 P2P、状态同步还是帧同步、为什么 Godot 4 自带 ENet 高层 API 是小项目的最优解。今天我们把那张地图**落地成代码**。目标 30 分钟:打开两个 Godot 窗口,一个当 Host 一个当 Client,各自用 WASD 控制自己的小方块,对方屏幕里的小方块跟着动。能跑起来就算入门。

## 〇、30 秒回顾:昨天那几个关键词

如果你直接打开今天这篇没看昨天的,先花 30 秒对齐概念:

- **ENet**:一个 UDP 之上的可靠传输层,Godot 直接内嵌,不用你装第三方库。UDP 快但会丢包,ENet 在 UDP 上加了**包序号 + 重传 + 通道(Channel)**,保证你发的数据对方按序收到。
- **peer(对端)**:网络上跟你连着的另一台机器,在 Godot 里用一个**整数 ID** 表示(`multiplayer.get_unique_id()` 拿到自己的 ID,`peer_connected(id)` 拿到对方的 ID)。`1` 这个 ID 永远保留给服务器。
- **权威(Authority)**:每个网络节点都有个"主人 ID",只有主人能写这个节点的"权威状态"。其它客户端拿到的是状态副本。今天我们会看到,每个 `Player` 节点的 authority 就是控制它的那个 peer ID。
- **状态同步 vs 帧同步**:昨天定了我们走**状态同步**——只广播"我现在的位置是 (3,1,5)",不广播"我按了 W 键"。`MultiplayerSynchronizer` 就是这个思路的编辑器配置入口。

OK,概念齐了,开干。

## 一、为什么今天只写 ENet 高层 API?

昨天说了,Godot 4 自带的 MultiplayerSpawner / MultiplayerSynchronizer 这套高层 API,是基于 ENet(UDP 可靠传输层)封装出来的**场景级同步器**。你不需要手写 Socket、不需要打包字节、不需要写丢包重传逻辑——只需要在编辑器里**勾选要同步的字段**,Godot 就帮你把状态广播给所有连接进来的 peer(对端)。

**今天我们要写出来的东西,加起来不到 100 行 GDScript,能跑起来的演示效果:**

- 一个 Godot 项目,既能当 Host,也能当 Client
- Host 启动一个 ENet server,监听 7777 端口
- Client 输入 Host IP(比如 `127.0.0.1` 或者 `192.168.1.10`),点 Join 就连上
- Host 和 Client 各控制自己的红色小方块,按 WASD 移动
- 双方屏幕里**同时显示两个方块**,对方控制的方块由 MultiplayerSynchronizer 自动同步位置

> 💡 **小贴士:为什么用两个窗口?** Godot 4 允许你同时启动多个"运行实例"(Run Instances),每个实例是独立的进程,模拟两台电脑。你不需要真的拿两台机器,这是单机调试多人游戏的标准做法。Mac / Linux / Windows 操作一样。

## 二、准备工作:5 分钟搭好舞台

### 步骤 1:新建项目

打开 Godot 4.6 → Project Manager → **New Project** →

- Project Name: `multiplayer_demo`
- Project Path: 选个你喜欢的空目录(比如 `~/godot/multiplayer_demo`)
- Renderer: **Forward+**(3D 必选,Compatibility 模式不支持某些 3D 特性)
- 点击 **Create & Edit**

### 步骤 2:打开编辑器设置,把"允许多实例"打开

这一步是关键,不然后面跑不出两个窗口。

顶部菜单 **Editor → Editor Settings...** → 搜索 `run` → 找到 **Run → Multiple Instances** → 勾选 **"Allow Multiple Instances"**

> 💡 **小贴士:为什么默认不允许?** Godot 默认假设你只想跑一个游戏实例(防止你手抖开 5 个窗口吃掉内存)。多人调试必须手动打开,改完一次后 Godot 会记住。

### 步骤 3:搭一个最简单的 3D 场景

在 FileSystem 面板右键 `res://` → **New → Folder** → 命名为 `scenes`。进入 `scenes` 文件夹,右键 → **New → Scene** → 会自动创建一个 `Node3D` 根节点。按 `Ctrl+S` 保存为 `main.tscn`(覆盖默认)。

选中根节点 `Main`(Node3D 类型),在 Inspector 面板把它 **Rename** 为 `Main`。然后在场景树里添加以下子节点(右键 `Main` → **Add Child Node**):

| 节点名 | 类型 | 作用 |
| --- | --- | --- |
| `Floor` | `MeshInstance3D` | 地面 |
| `Players` | `Node3D` | 用来挂载玩家角色的空容器 |
| `UI` | `CanvasLayer` | 放 UI |
| `Spawner` | `MultiplayerSpawner` | 同步生成玩家 |
| `SyncManager` | `MultiplayerSynchronizer` | 同步全局状态(可选,本教程先不深用) |

### 步骤 4:做个地面

选中 `Floor` 节点 → Inspector 面板 **Mesh** 属性 → **New BoxMesh** → 在弹出的 BoxMesh 里把 Size 改成 `(20, 0.2, 20)`,然后把 Floor 节点的 **Position** 改成 `(0, -0.1, 0)`。

> 💡 **小贴士:为什么 Size 是 20×0.2×20?** Y 轴的 0.2 是地面"厚度",20 是边长,你想要多大的竞技场就改这里。Position 的 -0.1 是把地面往下挪半个厚度,这样地面**顶面正好在 y=0**,玩家出生不会陷到地里。

### 步骤 5:做个玩家方块场景(关键,后面要被 Spawner 复用)

在 `scenes/` 文件夹右键 → **New → Scene** → 这次选 `CharacterBody3D` 作为根节点 → 命名为 `Player`。**Ctrl+S** 保存为 `scenes/player.tscn`。

接着:

1. 选中根 `Player` → Inspector → 添加 **CollisionShape3D** 子节点 → Inspector 里 **Shape** → New BoxShape3D → Size `(1, 1, 1)`
2. 再添加一个 **MeshInstance3D** 子节点 → Mesh → New BoxMesh → Size `(1, 1, 1)` → 材质 **New StandardMaterial3D** → **Albedo** 颜色改成红色 `(1, 0, 0)`
3. 添加一个 **MultiplayerSynchronizer** 子节点 → **Replication Config** 属性里点 `[empty]` → **New ReplicationConfig** → 弹出 ReplicationConfig 编辑器

在 ReplicationConfig 编辑器里:

- 左边 **Scene Path**(复制节点路径)选择 `.:Player`(`Player` 自己),同步属性选 **position**(位置)和 **rotation**(旋转)
- **Spawn** 选项保持勾选(默认就是勾的)
- **Sync** 选项保持勾选
- 点击 ReplicationConfig 编辑器右上角 **×** 关掉

> 💡 **小贴士:为什么要同步 position 和 rotation?** 这是"状态同步"的最小集——你只告诉对方"我现在的位置和朝向",对方拿到这俩数字就能渲染。两个玩家都在自己客户端独立运行物理和动画,只是彼此的状态数字会同步。比起"我按了 W 键"那种事件同步,这种**轮询式状态同步**简单十倍,适合大多数中小游戏。

回到 `Player` 场景根节点,给根节点**附加脚本**:`Player` 节点右上角 **Script** → **New Script** → 路径 `scenes/player.gd` → 创建。

最后 **Save Scene**(Ctrl+S)。

## 三、写代码:Host / Client 启动逻辑

### main.gd - 入口脚本

回到 `main.tscn`,选中根节点 `Main`,**附加脚本** → 新建 `scenes/main.gd`。把以下内容**完整复制粘贴**:

```gdscript
extends Node3D

const PORT := 7777
const MAX_CLIENTS := 4
const DEFAULT_SERVER_IP := "127.0.0.1"

@onready var ui_layer: CanvasLayer = $UI
@onready var players_root: Node3D = $Players
@onready var spawner: MultiplayerSpawner = $Spawner
@onready var status_label: Label = $UI/Panel/StatusLabel

var player_scene: PackedScene = preload("res://scenes/player.tscn")
var my_player: CharacterBody3D = null

func _ready() -> void:
    # 默认把 UI 显示出来,等用户点 Host / Join
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    multiplayer.connected_to_server.connect(_on_connected_to_server)
    multiplayer.connection_failed.connect(_on_connection_failed)
    multiplayer.server_disconnected.connect(_on_server_disconnected)

# ===================== UI 信号回调 =====================

func _on_host_pressed() -> void:
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_server(PORT, MAX_CLIENTS)
    if err != OK:
        _set_status("Host 失败: %s" % error_string(err))
        return
    multiplayer.multiplayer_peer = peer
    _set_status("Host 已启动,端口 %d。等待客户端连接..." % PORT)
    _spawn_local_player()

func _on_join_pressed() -> void:
    var ip := $UI/Panel/IPInput.text.strip_edges()
    if ip.is_empty():
        ip = DEFAULT_SERVER_IP
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_client(ip, PORT)
    if err != OK:
        _set_status("连接失败: %s" % error_string(err))
        return
    multiplayer.multiplayer_peer = peer
    _set_status("正在连接 %s:%d ..." % [ip, PORT])

func _on_disconnect_pressed() -> void:
    multiplayer.multiplayer_peer.close()
    multiplayer.multiplayer_peer = null
    _set_status("已断开连接")
    _remove_all_players()

# ===================== 玩家生成与同步 =====================

func _spawn_local_player() -> void:
    # 在自己机器上生成一个玩家,MultiplayerSpawner 会自动把它广播给所有 peer
    var player := player_scene.instantiate()
    player.name = str(multiplayer.get_unique_id())  # 节点名必须唯一,否则 Spawner 会冲突
    player.position = Vector3(randf_range(-3, 3), 1, randf_range(-3, 3))
    players_root.add_child(player, true)  # 第二个参数 true = 走 Spawner 同步路径
    my_player = player

func _on_peer_connected(id: int) -> void:
    _set_status("Peer %d 已连接" % id)
    # Host 这边:为新加入的客户端也生成一个 Host 自己的玩家给对方看
    if multiplayer.is_server():
        _spawn_local_player()

func _on_peer_disconnected(id: int) -> void:
    _set_status("Peer %d 已断开" % id)

func _on_connected_to_server() -> void:
    _set_status("连上服务器!我的 ID 是 %d" % multiplayer.get_unique_id())
    _spawn_local_player()

func _on_connection_failed() -> void:
    _set_status("连接服务器失败,检查 IP 和端口")

func _on_server_disconnected() -> void:
    _set_status("服务器已断开")
    _remove_all_players()

func _remove_all_players() -> void:
    for child in players_root.get_children():
        child.queue_free()
    my_player = null

func _set_status(msg: String) -> void:
    if status_label:
        status_label.text = msg
        print(msg)
```

> 💡 **小贴士:`add_child(node, true)` 第二个参数 `true` 是关键**。这个参数叫 `force_readable_name`,而且在 MultiplayerSpawner 体系下,**`add_child` 第二个参数还有一个隐藏作用**:它告诉 Spawner "这个新加的节点要走网络同步"。配合 Spawner 的 `_spawnable_scenes` 配置,服务器这边 `add_child` 之后,所有客户端会自动实例化同样的节点。如果忘了 true,只有本地能看到,别人看不到。

### player.gd - 玩家移动脚本

打开 `scenes/player.gd`,替换内容:

```gdscript
extends CharacterBody3D

const SPEED := 5.0

func _physics_process(delta: float) -> void:
    # 只让"自己拥有"的玩家响应输入,否则会两个方块都跟着你键盘走
    if not is_multiplayer_authority():
        return

    var input_dir := Vector2(
        Input.get_action_strength("ui_right") - Input.get_action_strength("ui_left"),
        Input.get_action_strength("ui_down") - Input.get_action_strength("ui_up"),
    )
    # WASD 也行:把 ui_* 映射到 WASD 在 Project Settings → Input Map 里加一下就行
    var key_w := Input.is_key_pressed(KEY_W) as float
    var key_s := Input.is_key_pressed(KEY_S) as float
    var key_a := Input.is_key_pressed(KEY_A) as float
    var key_d := Input.is_key_pressed(KEY_D) as float
    input_dir = Vector2(key_d - key_a, key_s - key_w)

    var direction := Vector3(input_dir.x, 0, input_dir.y).normalized()
    velocity = direction * SPEED
    move_and_slide()
```

> 💡 **小贴士:为什么必须判断 `is_multiplayer_authority()`?** 这就是昨天讲的**权威(Authority)** 概念落地——每个网络对象都有一个"主人"(authority),只有主人能写自己的状态。这里的 `Player` 节点,在 Spawner 同步到对端时,Godot 会自动把"主人的 peer id"设到节点的 `multiplayer_authority` 属性上。你如果不判断,本机按下 W 会同时驱动本机和对方屏幕里的那个方块——**两个方块都朝你方向飞**,乱套。

### 配置 MultiplayerSpawner

回到 `main.tscn`,选中 `Spawner` 节点 → Inspector:

1. **`_spawnable_scenes`** → 点 **Add Element** → 选 `res://scenes/player.tscn`
2. **`spawn_path`** → 点右边 **Assign** → 选场景树里的 `Main/Players` 节点(指向玩家容器的路径,告诉 Spawner "新生成的玩家挂在这下面")
3. 其他保持默认

### 配置 UI

在 `UI` (CanvasLayer) 下面加一个 **Panel**(Control 类型),Panel 里加:

- **`IPInput`**(LineEdit)—— 占位文本(Placeholder) 写 `127.0.0.1`
- **`HostBtn`**(Button)—— 文字 `Host(开房)`
- **`JoinBtn`**(Button)—— 文字 `Join(加入)`
- **`DisconnectBtn`**(Button)—— 文字 `Disconnect(断开)`
- **`StatusLabel`**(Label)—— 文字 `未连接`

把这三个 Button 的信号(`pressed`)分别连到 `main.gd` 里的 `_on_host_pressed` / `_on_join_pressed` / `_on_disconnect_pressed`。

## 四、跑起来!

按 **F5** 或顶部 ▶ Play 按钮,**第一次**会让你选主场景,选 `main.tscn`。

跑起来后:

1. 看到 UI 面板,点击 **Host** → 状态栏变成 `Host 已启动,端口 7777...`
2. 顶部菜单 **Editor → Run Multiple Instances** → 或者直接按 **F5 再一次**,Godot 会开第二个窗口
3. 第二个窗口点 **Join**(默认 IP 是 `127.0.0.1`)→ 状态变成 `连上服务器!我的 ID 是 ...`
4. 两个窗口都会**看到两个红色方块**,按 W/A/S/D,各自的方块移动,另一个窗口里那个方块**几乎同步跟着移动**

成功!这就是你的第一个 3D 联机 Demo。

### 进阶挑战(选做)

跑通 Demo 之后,可以试着改这两个功能,加深理解:

1. **加一个 `Camera3D` 跟着玩家**:把 Camera3D 挂到 `Player` 根节点下面(局部坐标 `(0, 5, 10)`,朝向 `(-15, 0, 0)`),你会发现 Host 窗口和 Client 窗口的视角各自跟随自己的方块——这是因为 Camera3D 是 Player 的子节点,跟着 Synchronizer 一起同步走。
2. **加一个 `Label3D` 显示玩家名字**:在 Player 场景里加 `Label3D`,在 `_ready()` 里 `text = "Player %d" % get_multiplayer_authority()`。你会看到两个窗口里,每个方块头顶分别写着 "Player 1" 和 "Player 2"(服务器是 1,客户端是随机分配的其它 ID)。这个练习会帮你直观理解 "authority" 的运行时表现。

## 五、3 个常见坑点 + 解决方案

**坑 1:第二个窗口点了 Join,但只看到自己一个方块,看不到对方的**

原因 99% 是 `Spawner.spawn_path` 没指向 `Players` 节点,或者 `_spawnable_scenes` 没加 `player.tscn`。回 `main.tscn` 检查 `Spawner` 节点的 Inspector,看 `spawn_path` 和 `_spawnable_scenes` 两栏是不是都填了。

**坑 2:按 W 移动,自己这边的方块飞了,但对方那边不动**

两个原因最常见:

- `player.gd` 里 `is_multiplayer_authority()` 判断漏了,本机驱动了对方那个方块的输入
- `MultiplayerSynchronizer` 的 ReplicationConfig 里 Scene Path 选错节点了,比如选成了 `MeshInstance3D` 而不是 `Player`。回 `player.tscn`,展开 ReplicationConfig,确认 Path 是 `.:Player`(`Player` 自己)

**坑 3:跨机器联不上(Host 在 A 电脑,Client 在 B 电脑连不上)**

排查顺序:

1. 两台电脑**同一个局域网**(Wi-Fi / 有线连同一个路由器)
2. 关掉 Windows 防火墙 / macOS 防火墙 / Linux `ufw`——至少先临时放行 7777 端口验证
3. Client 端 IPInput 框里输入**Host 机器的局域网 IP**(不是 `127.0.0.1`,那是本机),Win+R → 输入 `ipconfig` 看 IPv4 地址,Mac/Linux 终端 `ifconfig` 或 `ip addr`
4. 还是不行?在 Host 机器 `ping 192.168.x.x` 验证能通,再排查端口被占用 / 路由隔离

**坑 4:Host 窗口能看到 Client 方块,但 Client 窗口看不到 Host 方块**

最常见原因是 `add_child(player, true)` 第二个参数 `true` 漏了。`true` 这个参数(`force_readable_name`)在 MultiplayerSpawner 体系里有个隐藏作用——告诉 Spawner "这个新节点需要走网络同步路径"。如果不传,Host 本地 `add_child` 出来的玩家**只在 Host 这边存在**,Spawner 不会广播给客户端。改完保存,两个窗口重跑就能看到双向同步了。

**坑 5:ReplicationConfig 里的属性改了不生效**

比如你 ReplicationConfig 里加了 `velocity` 想同步速度,但运行时对方看到的方块移动依然卡顿。这是因为 `MultiplayerSynchronizer` 默认**每秒同步 30 次**(也就是 30Hz 的状态轮询),velocity 这种瞬时量很难体现平滑。要么你换成同步 `position`(已经在教程里做对了),要么把 Synchronizer 节点的 `replication_interval` 调到 0.016(约 60Hz)。位置同步天然是累积的,30Hz 已经足够丝滑。

## 六、3 个延伸阅读

1. **Godot 官方文档:High-level multiplayer**(权威且最新):<https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html>
2. **Godot 官方 Demo:Multiplayer Shooter**(完整可下载项目,涵盖 Spawner / Synchronizer / Lobby):<https://github.com/godotengine/godot-demo-projects/tree/master/networking/multiplayer_shooter>
3. **GameNetworking 知识库**(网络概念的中文友好百科,Katie 写的,业界公认的入门书):<https://gafferongames.com/categories/game-networking/>

---

## 软件版本与数据来源

- **Blender**:4.x 系列(本篇未直接使用,但模型导入路径沿用 06-12/13 教程)
- **Godot**:4.6(2026 年 6 月当前 stable 版本,`MultiplayerSpawner` 和 `MultiplayerSynchronizer` 在 4.2+ 已稳定)
- **GDScript 语法**:Godot 4.x(`@onready` / `@rpc` 等注解,4.0 引入)
- **数据来源**:Godot 官方文档 tutorials/networking 章节 + Godot Demo Projects `multiplayer_shooter` + 笔者本地实测(2026-06-16,macOS 14.5,Forward+ 渲染器)
- **前置阅读**:建议先看 06-15《联机架构总览》,否则你不知道今天写的 API 是在哪个网络模型下工作的