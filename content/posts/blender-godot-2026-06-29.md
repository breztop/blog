---
title: 'Blender & Godot 3D 教程 - 2026-06-29 - Godot 4 GDScript 基础零基础入门：变量 / 信号 / 生命周期，写一个会响应点击的 3D 小球'
date: 2026-06-29T04:00:00+08:00
description: '零基础打开 Godot 4.6 GDScript：彻底搞懂变量（var / @export / @onready）、信号（signal 的 emit / connect）、生命周期（_ready / _process / _physics_process / _input / _unhandled_input）三大基石，写一个「鼠标点击地面就在那个位置生成 3D 小球、点小球就让它跳一下」的完整 Demo。'
tags: [blender, godot, multiplayer, 3d, tutorial, gdscript, signals, lifecycle]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-29.png
  alt: '一个低多边形 3D 草地上有 5-6 个不同颜色的小球,玩家用鼠标点击地面会弹出新的小球,点击小球会让它弹起,Godot 编辑器暗色 UI'
---

![cover](/images/posts/blender-godot-2026-06-29.png)

# Godot 4 GDScript 基础零基础入门：变量 / 信号 / 生命周期，写一个会响应点击的 3D 小球

## 写在前面

如果你一路跟过这个系列，应该已经被我甩了一脸 GDScript 代码：

- 06-18「3D 基础」里写过 `extends Node3D` 和 `var speed := 5.0`
- 06-20「3D 物理」里写过 `_physics_process(delta)` 和 `move_and_slide()`
- 06-24「动画系统」里写过 `Tween.new()`
- 06-26「着色器」里写过 `shader_parameter/...`
- 06-28「联机大厅」里写过 `signal room_ready_changed` 和 `await`

但我**从来没单独一篇正经讲过 GDScript 这个语言本身**。今天补上这一课。

我们要一次性讲清三块基石——**变量 / 信号 / 生命周期**——并且每块都配一个能跑的小 Demo，最后合起来做一个完整的东西：

> **鼠标左键点地面 → 在那个位置生成一个会自转的小球；鼠标左键点小球 → 小球弹一下。**

不需要任何前置知识，会打开 Godot 编辑器就行。

> **小贴士 · 为什么是这三块？**  
> GDScript 语法糖很多（运算符、字典、lambda），但 90% 的「看不太懂」都来自这三块：变量作用域搞不清、信号回调没连上、`_ready` / `_process` 啥时候跑不明白。把这三块看明白，剩下的都是顺水推舟。

---

## 一、准备：建一个新项目

### 1.1 创建项目

1. 打开 Godot 4.6
2. 右上角点 **New Project**
3. **Project Name**：填 `gdscript_basics`
4. **Project Path**：随便选个空目录（不要选到已有项目里）
5. **Renderer**：保持默认 **Forward+** 就行
6. 点 **Create & Edit**

> **小贴士 · Forward+ vs Mobile vs Compatibility**  
> Forward+ 是 Godot 4 的现代渲染管線，支持全套特性（SSIL、Volumetric Fog、glow）。Mobile 适合手机平台、Compatibility 是 GLES3 老显卡兜底。零基础教程一律 Forward+，等你做发布版本时再换。

### 1.2 检查项目设置

顶部菜单 → **Project** → **Project Settings…**

左侧找 **Advanced Settings**（齿轮图标）打开，找到：

- **Physics / Common / Physics Ticks Per Second**：默认 `60`
- **Rendering / Environment / Default Clear Color**：默认 `#373a3f`（深灰蓝）

不用改，但记一下位置。

---

## 二、第一块基石：变量

### 2.1 第一个脚本：Hello, GDScript!

1. **Scene** 面板（左上角）→ 点 **+** 加号按钮 → 选 **Other Node** → 选 **Node** → 改名为 `Main`
2. 选中 `Main` 节点，按 **Ctrl+S** 保存场景到 `scenes/main.tscn`
3. 选中 `Main` 节点，点击右上角 **Inspector** 旁边的 **Script** 子标签 → 点 **+ Create a Script** 按钮
4. 弹窗里 Path 改成 `scripts/main.gd` → 点 **Create**
5. 编辑器自动打开 `main.gd`，里面是：

```gdscript
extends Node

# Called when the node is "ready", i.e. when both the node and its children have entered the scene tree.
func _ready() -> void:
    pass

# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta: float) -> void:
    pass
```

> **小贴士 · `extends Node` 是什么？**  
> GDScript 里 `extends` = 「这个脚本将挂在一个 Node 类型的对象上」。你后面做 3D 小球会写 `extends Node3D`，做 UI 控件会写 `extends Control`。第一行决定了你能用什么 API。

### 2.2 变量的三种「身份」

把 `main.gd` 改成下面这样：

```gdscript
extends Node

# 1️⃣ 普通变量：脚本内部自己用，编辑器看不到，改了要重新运行才生效
var click_count: int = 0

# 2️⃣ @export 变量：会出现在 Inspector 面板上，可以拖滑块、填数字、选颜色
@export var ball_color: Color = Color(1.0, 0.4, 0.2)
@export var ball_radius: float = 0.5
@export_range(0.1, 10.0, 0.1) var max_balls: float = 5.0

# 3️⃣ @onready 变量：等节点进场景树后才赋值（解决「脚本跑得比节点快」的经典坑）
@onready var label: Label = $HUD/CountLabel
```

**怎么验证？**

1. **保存**（Ctrl+S）
2. 回到 `main.tscn` 选中 `Main` 节点，看右侧 **Inspector** 应该出现 **Script Variables** 一节
3. 你会看到 `Ball Color`（带颜色拾取器）、`Ball Radius`（拖滑块）、`Max Balls`（0.1 到 10.0 范围限制）三个字段
4. 试着把 `Ball Color` 改成青色，**不重启场景**就能直接预览

> **小贴士 · 为什么 @onready 重要？**  
> 普通 `var label: Label = $HUD/CountLabel` 在脚本加载时立刻执行，但此时 `HUD` 子节点可能还没进场景树，**会拿到 null 报错**。`@onready` 等价于「等 `_ready()` 触发了再求值」，是零基础最容易踩的坑之一。

**改完后按 F5 运行**，Output 面板（底部）应该空——因为现在没打印任何东西。

### 2.3 类型推断 `:=`

GDScript 4 全面支持类型推断，省去重复声明：

```gdscript
var count := 0           # 推断为 int
var name := "ball"       # 推断为 String
var position := Vector3.ZERO  # 推断为 Vector3
```

不用写 `: int`、`: String`，但写也不报错。**团队代码里建议显式写类型**，方便别人阅读。

### 2.4 常见坑点 #1：忘加 `$` 符号

```gdscript
@onready var label = HUD/CountLabel       # ❌ 报错：Identifier 'HUD' not declared
@onready var label = $HUD/CountLabel      # ✅ $ 表示「从当前节点开始按路径找子节点」
```

---

## 三、第二块基石：生命周期

### 3.1 生命周期全景图

`Node` 类有一组特殊的虚函数，Godot 在特定时刻**自动**调用它们。你只需要「重写」（写一个同名函数），就能在那个时刻插入自己的逻辑。

| 函数名 | 何时被调用 | 一帧调用几次 | 典型用途 |
|---|---|---|---|
| `_init()` | 节点创建时（最早） | 只 1 次 | 初始化成员变量 |
| `_ready()` | 节点和子节点都进场景树 | 只 1 次 | 拿子节点引用、连信号 |
| `_process(delta)` | 每渲染帧 | 60 次/秒（视帧率） | 处理输入、UI 更新、转非物理的东西 |
| `_physics_process(delta)` | 每物理 tick | 固定 60 次/秒 | 移动 RigidBody、处理碰撞 |
| `_input(event)` | 每帧，最早收到输入 | 60 次/秒 | 想最先拦截输入 |
| `_unhandled_input(event)` | UI 没消化掉的输入 | 60 次/秒 | 3D 场景里最常用 |
| `_exit_tree()` | 节点离开场景树 | 只 1 次 | 清理、保存 |

> **小贴士 · 为什么要有两个 process？**  
> 渲染帧率会波动（屏幕 144Hz 就 144 次/秒），但物理 tick 永远是 60 次/秒。如果在 `_process` 里移动角色、检测碰撞，会出现「跑得快的玩家穿透墙」的诡异 bug。物理相关的东西**必须**在 `_physics_process` 里做。

### 3.2 实战：让场景动起来

把 `main.gd` 改成：

```gdscript
extends Node

@export var speed: float = 2.0

var _time_elapsed: float = 0.0

func _ready() -> void:
    print("🟢 _ready: 节点刚准备好")
    print("🟢 当前帧率上限: ", Engine.max_fps)

func _process(delta: float) -> void:
    _time_elapsed += delta
    rotate_y(speed * delta)
    # 注意：rotate_y 是 Node3D 的方法，但你 extends Node 会报错！
```

**保存 → F5 运行**，Output 会打印：

```
🟢 _ready: 节点刚准备好
🟢 当前帧率上限: 0
```

但 `rotate_y` 那行会报红字：**`Function "rotate_y()" not found in base self`**。

为什么？因为你 `extends Node`，而 `rotate_y()` 是 Node3D 的方法。

**修复：把 `Main` 节点改成 Node3D**

1. 选中 `Main` 节点 → Inspector 顶部 Type 旁边点 **change type 图标** → 选 **Node3D** → **Change**
2. 顶部菜单 → **File → Save Scene** → **Ctrl+S**

再 F5，应该没报错。但你也看不到任何东西，因为场景里没相机没光。

### 3.3 给场景加相机 + 光

回到 `main.tscn`：

1. **+ Node3D** → 选 **Camera3D** → 改名 `Camera`
2. 选中 `Camera`，右侧 **Inspector** → **Transform / Position** → 改成 `(0, 5, 10)`
3. 选中 `Camera`，**Inspector** → **Transform / Rotation** → 改成 `(-30, 0, 0)` 度
4. **+ Node3D** → 选 **DirectionalLight3D** → 改名 `SunLight`

按 F5，应该看到一个深灰蓝的天空，地面朝下没东西（因为还没放 mesh）。

### 3.4 常见坑点 #2：相机被遮蔽 / 看不到东西

最常见三种：
- **相机朝向不对**：选中 Camera，看场景视图里**预览框**朝向哪儿
- **相机太近**：Position 改成 `(0, 5, 10)` 这种「俯视斜后方」最稳
- **场景里没光源**：DirectionalLight3D 看不到「光」本身，但能照亮物体；不旋转它默认从 Z+ 方向照下（默认方向是 `(0, -1, 0)`）

---

## 四、第三块基石：信号

信号是 Godot 最独特的范式——一个节点「喊一嗓子」，所有「关注」它的节点都能听见。

### 4.1 信号的生命周期

```
节点 A ──emit_signal("name")──> Godot ──call──> 所有 connect("name", obj, "method") 的回调
```

四步走：

1. **声明信号**：`signal my_signal(arg1, arg2)`
2. **发射信号**：`my_signal.emit(value1, value2)`
3. **连接信号**：在另一个节点里 `node_a.my_signal.connect(_on_my_signal)`
4. **接收信号**：`func _on_my_signal(arg1, arg2) -> void:`

### 4.2 实战：HUD 显示点击次数

**4.2.1 建 UI 节点**

1. 选中 `Main` → **+ Control** → 改名 `HUD`
2. 选中 `HUD` → **+ Label** → 改名 `CountLabel`
3. 选中 `CountLabel` → Inspector → **Layout / Anchors Preset** → 点 **top-left** 那格（左上角图标）
4. 选中 `CountLabel` → Inspector → **Text** → 改成 `Clicks: 0`
5. **Ctrl+S** 保存

**4.2.2 声明 + 发射信号**

改 `main.gd`：

```gdscript
extends Node3D

# 声明一个信号：以后谁想「听到点击」就 connect 它
signal ball_clicked(ball: Node3D)

var click_count: int = 0

@onready var count_label: Label = $HUD/CountLabel

func _ready() -> void:
    # 把自己的信号连到一个本地方法上
    ball_clicked.connect(_on_ball_clicked)
    count_label.text = "Clicks: 0"

func _on_ball_clicked(ball: Node3D) -> void:
    click_count += 1
    count_label.text = "Clicks: " + str(click_count)
    print("🟡 信号收到,被点的是: ", ball.name)
```

**保存 → F5**，控制台暂时不会打印，因为现在没小球发信号——下一步加。

### 4.3 完整 Demo：点击地面生成小球 + 点小球弹一下

新建脚本 `scripts/ball.gd`：

```gdscript
extends Node3D

@export var jump_force: float = 3.0
@export var spin_speed: float = 2.0

var _velocity: Vector3 = Vector3.ZERO
var _is_jumping: bool = false

func _process(delta: float) -> void:
    rotate_y(spin_speed * delta)
    if _is_jumping:
        position += _velocity * delta
        _velocity.y -= 9.8 * delta  # 简易重力
        if position.y <= 0.5:
            position.y = 0.5
            _velocity = Vector3.ZERO
            _is_jumping = false

func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton and event.pressed and event.button_index == MOUSE_BUTTON_LEFT:
        # 用 raycast 从相机打一根射线，看打到的是谁
        var space := get_world_3d().direct_space_state
        var from := get_viewport().get_camera_3d().global_position
        var to := from + (-get_viewport().get_camera_3d().global_transform.basis.z) * 100.0
        var query := PhysicsRayQueryParameters3D.create(from, to)
        var result := space.intersect_ray(query)

        if result.is_empty():
            return

        var hit = result.collider
        if hit == self or hit.get_parent() == self:
            # 打到自己或子 mesh = 球被点了
            _jump()

func _jump() -> void:
    _velocity.y = jump_force
    _is_jumping = true
    # 发信号给 Main
    var main := get_tree().root.get_node_or_null("Main")
    if main and main.has_signal("ball_clicked"):
        main.ball_clicked.emit(self)
```

新建 `scripts/spawner.gd`：

```gdscript
extends Node3D

@export var ball_scene: PackedScene
@export var ground_y: float = 0.5

func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton and event.pressed and event.button_index == MOUSE_BUTTON_LEFT:
        var space := get_world_3d().direct_space_state
        var from := get_viewport().get_camera_3d().global_position
        var to := from + (-get_viewport().get_camera_3d().global_transform.basis.z) * 100.0
        var query := PhysicsRayQueryParameters3D.create(from, to)
        var result := space.intersect_ray(query)

        if result.is_empty():
            return

        # 只在打到地面时生成
        if result.collider.name == "Ground":
            var ball := ball_scene.instantiate()
            ball.position = Vector3(result.position.x, ground_y, result.position.z)
            get_tree().root.add_child(ball)
```

把脚本和场景拼起来：

1. `Main (Node3D)`
   - `Camera (Camera3D)` position `(0, 5, 10)`, rotation `(-30, 0, 0)`
   - `SunLight (DirectionalLight3D)`
   - `Ground (StaticBody3D)`
     - `Mesh (MeshInstance3D)` → Mesh 属性 → **New BoxMesh**，Size `(20, 0.2, 20)`，位置 `(0, -0.1, 0)`
     - `Collision (CollisionShape3D)` → Shape → **New BoxShape3D**，Size `(20, 0.2, 20)`
   - `Spawner (Node3D)` 挂 `scripts/spawner.gd`
   - `HUD (Control)`
     - `CountLabel (Label)`

**配置 Spawner**：

1. 把 `Ball` 做成单独场景：新建 `scenes/ball.tscn`，根节点 `Node3D`，挂 `scripts/ball.gd`
2. 在 `ball.tscn` 里加 `MeshInstance3D`（New SphereMesh，Radius 0.5），加 `CollisionShape3D`（New SphereShape3D，Radius 0.5）
3. Ctrl+S 保存
4. 回到 `main.tscn`，选中 `Spawner`，Inspector → **Ball Scene** → 点 **Load** → 选 `scenes/ball.tscn`

**F5 运行**：

- 鼠标点地面 → 新小球生成在点击位置
- 鼠标点小球 → 小球跳一下 + 顶部计数 +1
- Output 面板打印 `🟡 信号收到,被点的是: @Node3D@42`

🎉 第一个完整的「事件驱动」3D Demo 跑起来了。

> **小贴士 · 为什么用 _unhandled_input 而不是 _input？**  
> `_input` 最先收到，但 UI（Control 节点）会**先消化**鼠标事件。如果你写在 `_input` 里点按钮也会触发 3D 逻辑，UI 就会「漏点」。`_unhandled_input` = 「UI 不要的我才要」，是 3D 场景里的最佳实践。

---

## 五、常见坑点 #3 大汇总

### 5.1 信号忘了 `connect`，永远不触发

```gdscript
func _ready() -> void:
    ball_clicked.emit()  # ❌ 直接 emit 是「对空气喊」,没人会听到
```

**症状**：运行不报错，但回调函数永远不跑。  
**解决**：在另一个节点的 `_ready()` 里 `connect`，并且**确认那个节点还在场景树里**。

### 5.2 `_process` 里移动 RigidBody

```gdscript
func _process(delta):
    $RigidBody.position += Vector3.RIGHT * delta  # ❌ 物理引擎会跟你打架
```

**症状**：角色抖动、穿墙、不响应重力。  
**解决**：移动物理体用 `_physics_process(delta)`，或者用 `move_and_slide()`（CharacterBody3D 专用）。

### 5.3 `@export` 改完不生效

```gdscript
@export var speed: float = 2.0  # Inspector 改成 5.0,重启场景又变回 2.0?
```

**原因**：99% 是你**没保存场景**。Inspector 里改完要 `Ctrl+S`，否则下次加载场景用旧的 `.tscn` 文件覆盖。

---

## 六、延伸阅读

1. **Godot 官方文档 — GDScript 基础**：https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html  
   权威 reference，遇到语法疑问第一站。

2. **Godot 官方文档 — 信号**：https://docs.godotengine.org/en/stable/tutorials/scripting/singletons_autoload.html  
   除了 connect/emit，还讲了 Autoload 单例与信号组合的高级用法。

3. **GDQuest — GDScript Fundamentals**（YouTube 免费课）：https://www.youtube.com/watch?v=UcdwP2eKLtI  
   视频讲解 + 实战，对零基础读者特别友好。

---

## 软件版本 & 数据来源

- **Godot 引擎**：4.6 stable（2026-06 月度 release）
- **GDScript 版本**：4.x（Godot 4 内置）
- **教程实操日期**：2026-06-29
- **数据来源**：Godot 官方文档 4.6、GDQuest 公开教程、笔者实操截图

> **下一步预告**：明天 06-30 我们会讲 **Godot 4 UI 与 3D 场景交互（Control + SubViewport + Camera 切换）**——给今天的「点击生成小球」加一个右上角小地图，让 3D 场景里嵌一个会动的 2D 雷达。零基础读者特别期待的功能，敬请关注。

— breeze