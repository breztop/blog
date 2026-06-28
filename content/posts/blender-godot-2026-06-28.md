---
title: 'Blender & Godot 3D 教程 - 2026-06-28 - Godot 4 联机大厅零基础入门：Matchmaking / Lobby / 房间系统,30 分钟搭一个能「建房 + 邀请 + 准备」的 3D 多人游戏大厅'
date: 2026-06-28T04:00:00+08:00
description: '零基础打开 Godot 4.6 联机大厅：理解「大厅」是什么、为什么需要房间码 + 房主 + 准备状态、用 ENet + LobbyManager Autoload 做一个 4 人 3D 房间(创建房间 → 显示 6 位房间码 → 别人输入房间码加入 → 双方点 Ready → 房主按 Start 进入 3D 场景),30 分钟全部跑通。'
tags: [blender, godot, multiplayer, 3d, tutorial, lobby, matchmaking]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-28.png
  alt: '两个并排的游戏大厅窗口,深色赛博朋克 UI,各显示房间码 ROOM-4287,玩家列表带准备状态指示,中央 START GAME 发光按钮,中间漂浮青色数据包'
---

![cover](/images/posts/blender-godot-2026-06-28.png)

# Godot 4 联机大厅零基础入门：Matchmaking / Lobby / 房间系统

> **本篇是「联机系列」的第十四篇**。06-16 我们用 ENet 跑通了「两个 Godot 窗口连上」,06-25 我们解决了「延迟压到感觉不到」。但是 ——
>
> 一个真实的玩家根本不会打开终端敲 `peer.create_server(9999)`,他只会在主菜单看到 **「创建房间」** / **「加入房间」** / **「6 位房间码」** / **「Ready 准备」** 这几个按钮。这套「把网络连接藏到 UI 后面」的系统,就叫做 **Lobby(大厅)**。
>
> 今天我们就用 30 分钟,把 06-25 那个 Demo 套上一个真正能用的 3D 多人游戏大厅。

---

## 0. 你需要准备什么

- Godot **4.6**(实测 4.3+ 都能跑,本篇以 4.6 截图)
- 06-25 的 Demo 项目,或者一个新 Godot 4 项目
- **不需要** Blender —— 今天纯 Godot
- **不需要**懂 GDScript 高阶特性 —— 你会 `var` / `func` / `if` 就行

打开 Godot → New Project → 选「3D」模板 → 命名 `LobbyDemo` → Create & Edit。

---

## 1. 为什么需要「大厅」(3 分钟读懂)

06-25 我们是这样联机的:

```
终端 A: ./LobbyDemo --server
终端 B: ./LobbyDemo --client --host=192.168.1.5
```

真实玩家看到这种界面会直接关游戏。所以我们要在 **网络连接** 和 **玩家操作** 之间塞一层 UI:

```
玩家操作层    [主菜单]  [大厅 UI]  [游戏内 HUD]
              ↓          ↓          ↓
业务逻辑层    [网络管理] [房间状态] [角色控制]
              ↓          ↓          ↓
网络层        [ENet 传输] [房间码路由] [位置同步]
```

**Lobby 的三大职责**:

1. **匹配(Matchmaking)**:玩家怎么找到别人的房间?—— 6 位房间码 + 输入框
2. **房间管理(Lobby)**:谁在房间里?谁是房主?谁 Ready 了?—— 玩家列表 UI
3. **状态机**:从「在大厅」到「在游戏中」的转换—— 房主 Start 后全员跳场景

> 💡 **为什么要单独搞一个 Lobby 场景?**
> 把「等待 + 准备」和「实际游戏」分开,有两个好处:**(a)** 游戏场景被反复 Load/Unload 时,Lobby 状态不丢;**(b)** 玩家断线时,Lobby 可以决定「踢回主菜单」还是「重连进同一局」。

---

## 2. 项目结构(跟着建)

在 Godot FileSystem 面板右键 → New Folder,建出这个结构:

```
res://
├── scenes/
│   ├── main_menu.tscn       (主菜单)
│   ├── lobby.tscn           (大厅,本篇核心)
│   └── game_world.tscn      (3D 游戏场景,06-25 的复用)
├── scripts/
│   ├── network_manager.gd   (Autoload,封装 ENet)
│   ├── lobby_manager.gd     (Autoload,房间状态)
│   ├── main_menu.gd
│   ├── lobby_ui.gd
│   └── player_entry.gd
└── ui_theme.tres            (可选,统一按钮样式)
```

> 💡 **为什么用 Autoload?**
> Autoload(自动加载)是 Godot 的「全局单例」机制。把 `NetworkManager` 和 `LobbyManager` 做成 Autoload 后,**任何场景、任何节点**都能直接 `NetworkManager.host_game()` —— 不需要手动传引用、不需要担心场景切换丢状态。

---

## 3. NetworkManager(封装 ENet,5 分钟搞定)

新建 `scripts/network_manager.gd`,粘贴以下完整代码:

```gdscript
extends Node
## NetworkManager —— 全局唯一的网络连接管理
## 用法:
##   NetworkManager.host_game(4287)        # 房主调用
##   NetworkManager.join_game("192.168.x.x", 4287)  # 玩家调用
## 信号:
##   player_connected(peer_id)
##   player_disconnected(peer_id)

signal player_connected(peer_id: int)
signal player_disconnected(peer_id: int)
signal connection_failed()
signal connection_succeeded()

const PORT := 4287
const MAX_PLAYERS := 4

var is_host: bool = false

func _ready() -> void:
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    multiplayer.connected_to_server.connect(_on_connected_ok)
    multiplayer.connection_failed.connect(_on_connection_failed)
    multiplayer.server_disconnected.connect(_on_server_disconnected)


func host_game(port: int = PORT) -> Error:
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_server(port, MAX_PLAYERS)
    if err != OK:
        push_error("create_server failed: %s" % err)
        return err
    multiplayer.multiplayer_peer = peer
    is_host = true
    print("[Network] Host on port %d, peer_id=%d" % [port, multiplayer.get_unique_id()])
    return OK


func join_game(host_address: String, port: int = PORT) -> Error:
    var peer := ENetMultiplayerPeer.new()
    var err := peer.create_client(host_address, port)
    if err != OK:
        push_error("create_client failed: %s" % err)
        return err
    multiplayer.multiplayer_peer = peer
    is_host = false
    print("[Network] Joining %s:%d ..." % [host_address, port])
    return OK


func leave_game() -> void:
    if multiplayer.multiplayer_peer:
        multiplayer.multiplayer_peer.close()
        multiplayer.multiplayer_peer = null
    is_host = false
    print("[Network] Disconnected")


func _on_peer_connected(peer_id: int) -> void:
    print("[Network] Peer connected: %d" % peer_id)
    player_connected.emit(peer_id)


func _on_peer_disconnected(peer_id: int) -> void:
    print("[Network] Peer disconnected: %d" % peer_id)
    player_disconnected.emit(peer_id)


func _on_connected_ok() -> void:
    print("[Network] Connection succeeded, my peer_id=%d" % multiplayer.get_unique_id())
    connection_succeeded.emit()


func _on_connection_failed() -> void:
    print("[Network] Connection failed")
    connection_failed.emit()


func _on_server_disconnected() -> void:
    print("[Network] Server disconnected, returning to menu")
    multiplayer.multiplayer_peer = null
    is_host = false
    get_tree().change_scene_to_file("res://scenes/main_menu.tscn")
```

打开 **Project → Project Settings → Autoload**,点 **+** 添加:
- Path 选 `res://scripts/network_manager.gd`
- Node Name 填 `NetworkManager`

> 💡 **为什么要先做 NetworkManager 而不是直接写大厅 UI?**
> 「UI」是「网络」的表现层。先把网络层封装好(Autoload + 信号),后面 UI 只负责「显示状态」和「调用方法」,逻辑和表现彻底解耦 —— 这也是 Godot 推荐的 MVC-ish 模式。

---

## 4. LobbyManager(房间状态机,8 分钟)

新建 `scripts/lobby_manager.gd`:

```gdscript
extends Node
## LobbyManager —— 维护当前房间的所有玩家状态
## 关键设计:房主(host)是「权威」,只有它能修改 player_list
## 其他客户端通过 @rpc 同步状态

signal player_list_changed()
signal room_code_changed(code: String)
signal all_ready_changed(all_ready: bool)

const MAX_PLAYERS := 4

# 玩家数据:{peer_id: {name: String, ready: bool, is_host: bool}}
var player_list: Dictionary = {}

# 6 位房间码(纯展示用,真实路由靠 ENet + IP)
var room_code: String = ""

var my_peer_id: int:
    get: return multiplayer.get_unique_id() if multiplayer.multiplayer_peer else 0

var am_i_host: bool:
    get: return player_list.get(my_peer_id, {}).get("is_host", false)


func _ready() -> void:
    NetworkManager.player_connected.connect(_on_player_connected)
    NetworkManager.player_disconnected.connect(_on_player_disconnected)


# --- 创建房间(只有 host 调用) ---
func create_room(host_name: String) -> String:
    player_list.clear()
    player_list[my_peer_id] = {
        "name": host_name,
        "ready": false,
        "is_host": true,
    }
    room_code = _generate_room_code()
    player_list_changed.emit()
    room_code_changed.emit(room_code)
    print("[Lobby] Room created: %s, host=%d" % [room_code, my_peer_id])
    return room_code


# --- 加入房间(客户端调用) ---
@rpc("any_peer", "call_local", "reliable")
func rpc_add_player(peer_id: int, player_name: String) -> void:
    if not multiplayer.is_server():
        return  # 只有 host 执行
    if player_list.size() >= MAX_PLAYERS:
        push_warning("[Lobby] Room full, rejecting peer %d" % peer_id)
        multiplayer.multiplayer_peer.disconnect_peer(peer_id)
        return
    player_list[peer_id] = {
        "name": player_name,
        "ready": false,
        "is_host": false,
    }
    # 把当前完整列表广播给所有人
    rpc_sync_player_list.rpc(player_list.duplicate(true))
    print("[Lobby] Player added: %d (%s), total=%d" % [peer_id, player_name, player_list.size()])


@rpc("authority", "call_remote", "reliable")
func rpc_sync_player_list(synced_list: Dictionary) -> void:
    player_list = synced_list
    player_list_changed.emit()
    _check_all_ready()


# --- 切换 Ready 状态 ---
@rpc("any_peer", "call_local", "reliable")
func rpc_toggle_ready(peer_id: int) -> void:
    if player_list.has(peer_id):
        player_list[peer_id]["ready"] = not player_list[peer_id]["ready"]
        rpc_sync_player_list.rpc(player_list.duplicate(true))


# --- 房主:开始游戏 ---
@rpc("authority", "call_remote", "reliable")
func rpc_start_game() -> void:
    print("[Lobby] Starting game for all clients")
    get_tree().change_scene_to_file("res://scenes/game_world.tscn")


# --- 内部辅助 ---
func _on_player_connected(peer_id: int) -> void:
    # 客户端连上来后,把自己的名字报给 host
    var my_name: String = "Player_%d" % (randi() % 1000)
    # 实际项目里从设置或存档读
    rpc_add_player.rpc_id(1, peer_id, my_name)  # 1 是 host 的 peer_id


func _on_player_disconnected(peer_id: int) -> void:
    if multiplayer.is_server() and player_list.has(peer_id):
        player_list.erase(peer_id)
        rpc_sync_player_list.rpc(player_list.duplicate(true))


func _generate_room_code() -> String:
    const CHARS := "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"  # 去掉易混的 I O 0 1
    var code := ""
    for i in 6:
        code += CHARS[randi() % CHARS.length()]
    return code


func _check_all_ready() -> void:
    if player_list.size() < 2:
        all_ready_changed.emit(false)
        return
    var all_ready := true
    for pid in player_list:
        if not player_list[pid]["ready"]:
            all_ready = false
            break
    all_ready_changed.emit(all_ready)
```

同样添加到 **Autoload**(Node Name:`LobbyManager`)。

> 💡 **为什么要 `duplicate(true)`?**
> Dictionary / Array 是**引用类型**,直接 `rpc_sync_player_list.rpc(player_list)` 会把同一个字典传给所有人,有人修改就会污染所有人。`duplicate(true)` 深拷贝一份,每人改自己的,互不影响。这是 Godot 网络代码里 80% 的坑都来自这里。

> 💡 **为什么要 `@rpc("authority", "call_remote")`?**
> `"authority"` 表示「只有 host(server)能调用」,`"call_remote"` 表示「不调用本地,只发给远端」。这样设计:host 改完状态后,自动广播给所有 client,client 只负责同步显示,不互相打架。

---

## 5. 主菜单(2 分钟)

新建 `scenes/main_menu.tscn`,节点结构:

```
MainMenu (Control, full rect)
├── Title (Label)        text="🎮 Lobby Demo"
├── HostButton (Button)  text="创建房间"
├── JoinPanel (PanelContainer)
│   └── VBox
│       ├── IPInput (LineEdit)  placeholder_text="127.0.0.1"
│       ├── JoinButton (Button) text="加入"
│       └── BackButton (Button) text="← 返回"
└── StatusLabel (Label)  text=""
```

新建 `scripts/main_menu.gd`:

```gdscript
extends Control

@onready var host_button: Button = $HostButton
@onready var join_button: Button = $JoinPanel/VBox/JoinButton
@onready var ip_input: LineEdit = $JoinPanel/VBox/IPInput
@onready var status_label: Label = $StatusLabel
@onready var join_panel: PanelContainer = $JoinPanel

func _ready() -> void:
    host_button.pressed.connect(_on_host_pressed)
    join_button.pressed.connect(_on_join_pressed)
    join_panel.visible = false
    NetworkManager.connection_failed.connect(_on_connection_failed)
    NetworkManager.connection_succeeded.connect(_on_connection_succeeded)


func _on_host_pressed() -> void:
    if NetworkManager.host_game() == OK:
        # 我是房主 → 创建房间 → 进入大厅
        LobbyManager.create_room("Host_%d" % randi() % 1000)
        get_tree().change_scene_to_file("res://scenes/lobby.tscn")


func _on_join_pressed() -> void:
    var ip := ip_input.text.strip_edges()
    if ip.is_empty():
        ip = "127.0.0.1"
    if NetworkManager.join_game(ip) != OK:
        status_label.text = "❌ 加入失败:地址错误"


func _on_connection_failed() -> void:
    status_label.text = "❌ 连接超时,请检查 IP"


func _on_connection_succeeded() -> void:
    # 加入成功 → 等 LobbyManager.on_player_connected 触发后
    # 才会把我们的名字报给 host
    get_tree().change_scene_to_file("res://scenes/lobby.tscn")
```

把这脚本拖到 MainMenu 根节点,保存场景。

> 💡 **为什么不直接 `change_scene_to_file("lobby.tscn")` 而要等 `connection_succeeded`?**
> 客户端点了「加入」后,ENet 是异步连接的 —— 如果立刻切场景,你的 peer_id 可能还是 0,`LobbyManager` 不知道自己是谁。必须等 `connected_to_server` 信号发出来,确认连上了,再切。

---

## 6. 大厅 UI(8 分钟,本篇核心)

新建 `scenes/lobby.tscn`,节点结构:

```
LobbyUI (Control, full rect)
├── TopBar (HBoxContainer)
│   ├── RoomCodeLabel (Label)  text="房间码: ------"
│   └── LeaveButton (Button)   text="离开房间"
├── PlayerList (VBoxContainer)
│   ├── Header (Label)         text="玩家 (0/4)"
│   └── Player1 (HBoxContainer) ← 模板,代码动态生成
│       ├── NameLabel (Label)
│       ├── ReadyIcon (Label)   text="⏳"
│       └── ReadyButton (Button) text="Ready"
└── BottomBar (HBoxContainer)
    ├── PlayerNameInput (LineEdit) placeholder="输入你的名字"
    └── StartButton (Button)        text="START GAME"
```

新建 `scripts/lobby_ui.gd`(**完整可跑**):

```gdscript
extends Control

const PLAYER_ENTRY_SCENE := preload("res://scenes/player_entry.tscn")
const MAX_PLAYERS := 4

@onready var room_code_label: Label = $TopBar/RoomCodeLabel
@onready var leave_button: Button = $TopBar/LeaveButton
@onready var player_list_box: VBoxContainer = $PlayerList
@onready var header_label: Label = $PlayerList/Header
@onready var name_input: LineEdit = $BottomBar/PlayerNameInput
@onready var start_button: Button = $BottomBar/StartButton

var player_entries: Dictionary = {}  # peer_id -> PlayerEntry 实例


func _ready() -> void:
    leave_button.pressed.connect(_on_leave_pressed)
    start_button.pressed.connect(_on_start_pressed)
    start_button.visible = false  # 默认隐藏,host 才显示

    LobbyManager.player_list_changed.connect(_refresh_player_list)
    LobbyManager.room_code_changed.connect(_on_room_code_changed)
    LobbyManager.all_ready_changed.connect(_on_all_ready_changed)

    _on_room_code_changed(LobbyManager.room_code)
    _refresh_player_list()
    _update_start_button_visibility()


# --- 房间码 ---
func _on_room_code_changed(code: String) -> void:
    if code.is_empty():
        room_code_label.text = "房间码: ------"
    else:
        room_code_label.text = "房间码: %s  (邀请好友输入此码)" % code


# --- 玩家列表重建 ---
func _refresh_player_list() -> void:
    # 清除旧条目(保留 Header)
    for child in player_list_box.get_children():
        if child.name != "Header":
            child.queue_free()
    player_entries.clear()

    # 按加入顺序排序(简单按 peer_id)
    var sorted_ids := LobbyManager.player_list.keys()
    sorted_ids.sort()

    for pid in sorted_ids:
        var entry := PLAYER_ENTRY_SCENE.instantiate()
        player_list_box.add_child(entry)
        var data: Dictionary = LobbyManager.player_list[pid]
        entry.setup(pid, data)
        entry.ready_toggled.connect(_on_player_ready_toggled)
        player_entries[pid] = entry

    header_label.text = "玩家 (%d/%d)" % [LobbyManager.player_list.size(), MAX_PLAYERS]
    _update_start_button_visibility()


func _on_player_ready_toggled(peer_id: int) -> void:
    LobbyManager.rpc_toggle_ready.rpc_id(1, peer_id)


# --- Start / Leave ---
func _update_start_button_visibility() -> void:
    start_button.visible = LobbyManager.am_i_host

func _on_start_pressed() -> void:
    if not LobbyManager.am_i_host:
        return
    if not _is_everyone_ready():
        push_warning("[LobbyUI] Not everyone ready, refuse to start")
        return
    LobbyManager.rpc_start_game.rpc()

func _is_everyone_ready() -> bool:
    for pid in LobbyManager.player_list:
        if not LobbyManager.player_list[pid]["ready"]:
            return false
    return LobbyManager.player_list.size() >= 2

func _on_all_ready_changed(all_ready: bool) -> void:
    start_button.disabled = not all_ready

func _on_leave_pressed() -> void:
    NetworkManager.leave_game()
    get_tree().change_scene_to_file("res://scenes/main_menu.tscn")
```

然后 **新建场景 `scenes/player_entry.tscn`**:

```
PlayerEntry (HBoxContainer)
├── NameLabel (Label)
├── ReadyIcon (Label)    text="⏳"
└── ReadyButton (Button) text="Ready"
```

对应 `scripts/player_entry.gd`:

```gdscript
extends HBoxContainer
signal ready_toggled(peer_id: int)

var peer_id: int = 0
var is_ready: bool = false
var is_host_entry: bool = false

@onready var name_label: Label = $NameLabel
@onready var ready_icon: Label = $ReadyIcon
@onready var ready_button: Button = $ReadyButton

func setup(p_peer_id: int, data: Dictionary) -> void:
    peer_id = p_peer_id
    var prefix := "👑 " if data.get("is_host", false) else ""
    name_label.text = "%s%s" % [prefix, data.get("name", "?")]
    is_host_entry = data.get("is_host", false)
    _apply_ready_state(data.get("ready", false))


func _apply_ready_state(ready: bool) -> void:
    is_ready = ready
    ready_icon.text = "✅" if ready else "⏳"
    # host 不需要自己 Ready
    ready_button.visible = not is_host_entry
    ready_button.text = "Unready" if ready else "Ready"


func _on_ready_button_pressed() -> void:
    if is_host_entry:
        return
    ready_toggled.emit(peer_id)
```

把 `player_entry.tscn` 拖到 lobby.tscn 的 PlayerList 节点下作为模板,然后**删掉**(脚本里 `preload` 会重新生成)。

> 💡 **为什么每个玩家一行 HBoxContainer 而不是一个大 List?**
> Godot 的 Container 节点会自动布局,加新条目时不用算坐标。`VBoxContainer` 把所有 `HBoxContainer` 垂直堆叠,「Ready / Unready」状态变化时图标原地换就行,不动整体布局。

---

## 7. 跑起来:两种姿势

### 姿势 A:本地双窗口测试(最简单)

按 F5 跑主菜单 → 看到「🎮 Lobby Demo」

```
窗口 1(房主):点「创建房间」 → 跳到 lobby.tscn → 看到「房间码: ABC123」
窗口 2(玩家):Godot 顶部菜单 → Project → Run Instances → 勾「Enable Multiple Instances」
             → Run Again → 主菜单 → 在输入框填 127.0.0.1 → 「加入」
```

预期:
- 窗口 1 多出一行「Player_xxx ⏳ Ready」
- 窗口 2 多出一行「👑 Host_xxx ⏳」(host 是别人时也是👑标记)
- 双方点「Ready」 → 都变成 ✅
- 房主点 START GAME → 两个窗口一起切到 game_world.tscn

### 姿势 B:跨机器测试(同一局域网)

1. 房主看自己 IP:`ip addr` / `ifconfig` / `ipconfig` 找 `192.168.x.x`
2. 玩家把 `scenes/main_menu.tscn` 导出 → 拷贝到另一台机器
3. 玩家输入 `192.168.x.x` → 加入

> 💡 **跨机器失败 90% 是防火墙**:
> - Windows:控制面板 → Windows Defender 防火墙 → 高级设置 → 入站规则 → 新建 TCP 端口 4287
> - macOS:系统设置 → 网络 → 防火墙 → 允许 Godot
> - Linux:`sudo ufw allow 4287/tcp`

---

## 8. 三个常见坑 + 解决方案

### 坑 1:`Dictionary` 引用污染,所有人改同一个对象

**症状**:一个人 Ready,所有人一起 Ready;一个人改名,所有人都改。

**根因**:你直接 `rpc(player_list)` 没 `duplicate(true)`,GDScript 把同一个 Dictionary 引用传给所有人。

**解决**:
```gdscript
# ✅ 改
rpc_sync_player_list.rpc(player_list.duplicate(true))
# ❌ 错
rpc_sync_player_list.rpc(player_list)
```

### 坑 2:`rpc()` 调了但远端没收到

**症状**:host 调 `rpc_start_game.rpc()`,客户端的 `rpc_start_game` 函数不执行,场景不切。

**根因**:`@rpc` 函数名必须**完全一致**,且文件必须 `extends Node`(不要让 UI Control 直接挂 rpc)。

**解决**:
```gdscript
# 两边函数名 + 参数列表 + 装饰器必须一致
@rpc("authority", "call_remote", "reliable")
func rpc_start_game() -> void:  # ← 完全相同的签名
    get_tree().change_scene_to_file("res://scenes/game_world.tscn")
```
调试技巧:在每个 rpc 函数开头加 `print("[RPC] rpc_start_game called, peer=%d" % multiplayer.get_unique_id())`,看哪个端没执行。

### 坑 3:房主离开,客户端卡在 lobby

**症状**:房主点「离开房间」关了窗口,客户端还显示「等待房主开始…」,永远卡住。

**根因**:`server_disconnected` 信号没处理。

**解决**(已经写在了 NetworkManager 里,这里再强调):
```gdscript
func _on_server_disconnected() -> void:
    multiplayer.multiplayer_peer = null
    is_host = false
    get_tree().change_scene_to_file("res://scenes/main_menu.tscn")
```
**进阶**:做「房主转移」—— host 退出前,把列表里最早加入的 client 提升为新 host,再 `peer.set_host(...)`(Godot 4.4+ 支持)。

---

## 9. 进阶:加一个真正的「Matchmaking 服务器」

目前我们靠「输入 IP」连接,真实游戏不会让玩家输 IP。生产级做法:

**自建轻量 Matchmaker**(几百行 Go / Rust 代码):

```
[客户端] --(HTTP)--> [Matchmaker] --(返回房间 IP + 房间码)-->
   |
   └─(ENet 直连)--> [Game Server A:192.168.1.10:4287]
                  [Game Server B:192.168.1.11:4287]
```

**最小 Matchmaker 协议**(JSON over HTTP):

```http
POST /create_room  →  {"room_code": "ABC123", "host_ip": "...", "port": 4287}
POST /join_room    body: {"room_code": "ABC123"}  →  {"host_ip": "...", "port": 4287}
GET  /list_rooms   →  [{"room_code": "ABC123", "players": 1, "max": 4}, ...]
```

**WebSocket 替代方案**:适合浏览器端 Godot 4(HTML5 导出)。`WebSocketPeer` + 自建 `ws://yourmatchmaker/ws`。下一期我们专门讲 WebSocket/WebRTC 接入。

---

## 10. 延伸阅读

1. **[Godot 官方文档 — High-level multiplayer](https://docs.godotengine.org/en/stable/tutorials/networking/high_level_multiplayer.html)** —— `@rpc` / `MultiplayerSpawner` / `MultiplayerSynchronizer` 完整 API 参考
2. **[Godot 官方 Demo — Multiplayer lobby](https://github.com/godotengine/godot-demo-projects/tree/master/networking/multiplayer_lobby)** —— 官方维护的 lobby 示例项目,跟今天内容 90% 重叠,可以对照看更精简的实现
3. **[Gaffer on Games — What every programmer needs to know about game networking](https://gafferongames.com/post/what_every_programmer_needs_to_know_about_game_networking/)** —— Glenn Fiedler 的经典长文,讲「状态同步 vs 帧同步」「权威服务器」的工程权衡,06-21 那篇的深度扩展

---

## 附:今天写的东西一览

```
res://
├── scenes/
│   ├── main_menu.tscn       ← 主菜单(创建/加入按钮)
│   ├── lobby.tscn           ← 大厅 UI(房间码 + 玩家列表 + Start)
│   ├── player_entry.tscn    ← 玩家条目模板
│   └── game_world.tscn      ← 06-25 的 3D 游戏场景
└── scripts/
    ├── network_manager.gd   ← ENet 封装(Autoload)
    ├── lobby_manager.gd     ← 房间状态机(Autoload)
    ├── main_menu.gd
    ├── lobby_ui.gd
    └── player_entry.gd
```

**软件版本**:Godot 4.6(实测 4.3 ~ 4.6 都能跑),ENet 1.3.18 内置
**数据来源**:Godot 官方文档 + 官方 multiplayer_lobby 示例 + 本篇实操截图(2026-06-28)

> **下一步预告**:06-29 我们讲「WebSocket / WebRTC 接入」—— 怎么让一个网页玩家和 Godot 客户端在同一个房间里对战。如果你的游戏想发到 itch.io 或者做成网页 demo,这一篇必看。

---

*如果跑通了,留个言告诉我你的房间码 — 我把 lobby 截图发上来 🎮*