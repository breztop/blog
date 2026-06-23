---
title: "Blender & Godot 3D 教程 - 2026-06-24 - Godot 4 动画系统零基础入门:AnimationPlayer + Tween + AnimationTree 状态机"
date: 2026-06-24T04:00:00+08:00
description: "零基础打开 Godot 4.6 动画系统:理解 AnimationPlayer(关键帧轨道)、Tween(代码驱动轻量动画)、AnimationTree 状态机(自动按状态切动画)三件套,做出一个会待机/会跑/会跳、UI 按钮按下时平滑缩放的完整 3D 角色 Demo,并把 GDScript 信号连接、C# 与 GDScript 互调用讲清。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-24.png
  alt: "Godot 4 3D 场景中一个低多边形角色在跑动,旁边有跳跃动画的影子残像"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-24.png)

> 系列第十二篇。06-20 我们给 Godot 加上了物理,角色能站能跳能推箱子——**但他站得像根木头,跳起来没有任何"起跳—滞空—落地"的曲线感**;菜单按钮按下去"啪"地一下变灰,毫无反馈。**这两类问题都靠"动画"解决**。
> 
> 今天把 Godot 4.6 的动画三件套一次性讲透:`AnimationPlayer`(关键帧轨道,手动编曲)、`Tween`(代码驱动的轻量动画,一行动画效果)、`AnimationTree`(状态机,自动按状态切动画)。学完这一篇,**你的游戏从"能动"升级到"活过来"**——也为后续 06-29+ 的"综合实战"模式(把 Blender 模型导入 Godot 后做带动画的角色)打地基。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 理解 Godot 里"动画"这件事为什么被拆成三件套(轨道 / 代码 / 状态机)
- 用 `AnimationPlayer` 给一个 3D 立方体做出"位置 + 颜色 + 缩放"三轨道的关键帧动画
- 用 `Tween` 用一行 GDScript 让按钮"按下时缩一下、弹回",做 UI 反馈
- 用 `AnimationTree` 搭一个 Idle / Run / Jump 状态机,根据角色速度自动切动画
- 跑通一个**完整可玩**的 3D Demo:按下 WASD 角色跑动(切到 Run),按空格起跳(切到 Jump),松开落地(回 Idle)
- 搞懂动画播放器里那个"人形资源"是个什么玩意(为后面接入 Blender 骨骼模型铺路)

**前置:**

- Godot 4.6 已装能开
- 看过 06-20(知道 `CharacterBody3D` + `move_and_slide()` 怎么用)
- 知道 `Node3D` 是根节点、GDScript 能看懂 `func` / `if` / `signal`
- **不要求**接触过 Blender 骨骼(今天用 Godot 自带的占位模型就够)

## 1. 为什么"动画"要拆三件套?

打开 Godot 文档搜"animation"会蹦出来一堆节点:`AnimationPlayer` / `AnimationTree` / `Tween` / `AnimatedSprite2D` / `AnimationMixer`……第一次看会懵。

其实你只需要理解一个原则:**"动画"在游戏里有三种完全不同的来源,Godot 给了三种不同的工具**。

| 来源 | 谁在控制 | 工具 | 适用场景 |
|---|---|---|---|
| **美术/设计师手摆的"关键帧"** | 美术在时间线上摆的(像做 Flash) | `AnimationPlayer` | 角色待机动作、攻击动作、过场动画、相机运镜 |
| **程序员代码驱动的"轻动画"** | 一行 GDScript 触发的"插值" | `Tween` | UI 反馈(按钮按下缩一下)、飘字伤害数字、相机震屏 |
| **运行时根据状态自动切** | 代码读"当前是跑还是跳",自动播对应动画 | `AnimationTree` | 角色状态机(待机/跑/跳/攻击自由切换) |

> 💡 **为什么要拆?** 因为**设计意图不同**。一个动作是"策划调出来的"还是"程序写出来的",决定了谁应该改它。Godot 把这两个职责拆得很清——`AnimationPlayer` 里调关键帧的是动画师/关卡设计师,`Tween` 是程序员的工具,`AnimationTree` 是连接两者的胶水。**你今天会看到,这三个东西不是替代关系,是组合关系**。

### 1.1 一句话心法

> **AnimationPlayer = 录音机(把动作"录"下来回放);Tween = 闹钟(到时间自动插值);AnimationTree = DJ 台(根据节奏切歌)**。

---

## 2. AnimationPlayer:把"动作"录下来

### 2.1 第一个动画:让立方体"原地跳一下"

**步骤 1**:打开 Godot 4.6,新建一个 `Node3D` 根节点的场景,根节点改名叫 `World`,Ctrl+S 存为 `world.tscn`。

**步骤 2**:在 `World` 下加一个 `+` → 搜 `CSGBox3D`(用 CSG 节点,不用去找外部模型),`Size` 保持 `(2, 2, 2)`,位置 `Position = (0, 1, 0)` 让它浮在地上一米。

**步骤 3**:加一盏 `DirectionalLight3D`,`Rotation` 改成 `(-50, -30, 0)`,确保 `Shadow Enabled` 勾上。

**步骤 4**:加一个 `Camera3D`,`Position = (0, 3, 8)`,`Rotation = (-15, 0, 0)`,`Current = true`。

**步骤 5**:加一个 `WorldEnvironment`,新建 `Environment` 资源,`Background Mode` 选 `Sky`,这样就有蓝天。

**步骤 6**:**关键一步**。在 `World` 下加一个 `+` → 搜 `AnimationPlayer` → 添加,选中 `AnimationPlayer` 节点。

**步骤 7**:选中 `AnimationPlayer`,看编辑器下方的"动画"面板(Animation dock,默认在编辑器底部)。点"新建动画"(New Animation)按钮,弹窗输入名字 `bounce`,点 OK。

> 💡 **为什么要这样?** `AnimationPlayer` 本身只是个"播放器",它需要一个或多个"动画资源"(`.anim` 文件)才能工作。新建动画 = 新建一个空白的"动画磁带"。

**步骤 8**:现在编辑器右下角出现了一条"时间轴"。把播放头(那条竖线)拖到 **`0.5 秒`** 的位置。然后选中场景里的 `CSGBox3D`,把它的 `Position` 改成 `(0, 3, 0)`——也就是"半秒时它跳到了 3 米高"。

你会看到时间轴上 `0.5s` 处自动出现了一个**黄色菱形**(关键帧 Keyframe)。

**步骤 9**:把播放头拖到 `1.0 秒`,把 `CSGBox3D` 的 `Position` 改回 `(0, 1, 0)`。

**步骤 10**:时间轴上 `0s` 和 `1.0s` 处**右键** → "插入关键帧"(Insert Key),选 `position` 这个属性,这样开始帧和结束帧也有了"插值起点"。

**步骤 11**:把播放头拖回 `0s`,点编辑器顶部的"播放"按钮 ▶ —— 你会看到立方体**从 0.5 秒起跳、1.0 秒落回**。

> 💡 **为什么要这样?** 关键帧的本质是"在某个时间点上**记录**某个值",然后播放器在两个关键帧之间**自动插值**(默认线性 linear,也可以改 Ease In/Out)。你不写代码,Godot 就帮你把"位置"按时间连续变化。

### 2.2 关键代码:用 GDScript 触发这个动画

新建脚本挂在 `World` 节点上,代码如下,**完整可复制**:

```gdscript
# world.gd - 演示 AnimationPlayer 触发
extends Node3D

@onready var anim_player: AnimationPlayer = $AnimationPlayer

func _ready() -> void:
    # 等待 1 秒后开始播放"bounce"动画
    await get_tree().create_timer(1.0).timeout
    anim_player.play("bounce")
    print("播放 bounce 动画,持续 1 秒")
```

> 💡 **为什么要 `await`?** 场景刚启动时 `AnimationPlayer` 还没完全初始化,直接 `play()` 在某些版本会报警告。等 1 秒是教学 demo 的简单延迟,生产代码一般会在按钮信号里调用。

### 2.3 多轨道:同时改"位置 + 颜色 + 缩放"

一个动画**不只是位置**。在动画面板顶部,你看到的"轨道"(Tracks)区域,目前只有 `CSGBox3D:position` 这一条。你可以继续加:

**步骤 1**:在动画面板的"轨道"区域上方点 `+` → `Add Track` → `Property Track` → 选 `CSGBox3D`。

**步骤 2**:弹窗让你选属性,选 `material:albedo_color`(`albedo_color` 是 CSG 的基础颜色属性)。确定后这条轨道就出现了。

**步骤 3**:在 `0.0s` 处右键这条轨道 → `Insert Key` → 选 `material:albedo_color`,在弹出的小颜色盘选**红色**。

**步骤 4**:在 `0.5s` 处插一个关键帧,颜色选**蓝色**。

**步骤 5**:再加一条 `CSGBox3D:scale` 轨道,在 `0.0s` 缩放 `(1, 1, 1)`,`0.5s` 缩放 `(1.5, 1.5, 1.5)`,`1.0s` 回到 `(1, 1, 1)`。

**步骤 6**:播一下 ——立方体一边跳、一边变色、一边放大缩小,**三个属性同时被轨道驱动**。

> 💡 **为什么要多轨道?** 因为游戏的"动作"从来不是单一属性。角色起跳是 `position.y` 在变 + `rotation` 在扭 + 可能 `material` 闪一下白光表示无敌帧。一个 `AnimationPlayer` 用多条轨道驱动**同一个对象或不同对象**的多个属性,这是动画的核心。

### 2.4 循环播放

点动画面板底部那个"循环"(Loop)图标(长得像两个箭头组成的圈),动画就会播完自动重播。生产代码里也会用 `anim_player.play("bounce")` 反复调用。

---

## 3. Tween:一行的"轻动画"

`AnimationPlayer` 适合美术手摆的"复杂长动画"。**但 90% 的 UI 反馈根本不需要手摆关键帧**——按钮按下缩一下、飘字从头顶冒出来、相机被击中震一下……这种"程序触发的简单插值"用 `Tween` 一行解决。

### 3.1 第一个 Tween:让按钮按下时"弹一下"

**步骤 1**:新建一个 `Control` 根节点的场景,根节点改名叫 `MenuUI`,加一个 `Button` 子节点,文字写"开始游戏",位置摆到屏幕中央。

**步骤 2**:给 `Button` 加一个 `pressed` 信号连接到 `MenuUI` 上的脚本:

```gdscript
# menu.gd - 演示 Tween
extends Control

@onready var start_button: Button = $Button

func _ready() -> void:
    start_button.pressed.connect(_on_start_pressed)

func _on_start_pressed() -> void:
    # 一行 Tween:按钮按下时缩到 0.8,0.1 秒后弹回 1.0
    var tween := create_tween()
    tween.tween_property(start_button, "scale", Vector2(0.8, 0.8), 0.1)
    tween.tween_property(start_button, "scale", Vector2.ONE, 0.1)
```

**步骤 3**:跑场景,点按钮 —— 你会看到按钮"duang"地缩一下弹回来,**10 行代码、零关键帧**。

> 💡 **为什么 `Tween` 不是 `AnimationPlayer` 的简化版?** 因为 Tween 是"代码生成的临时动画"——**用完就丢**,不占用 `.anim` 文件、不进资源库。它的生命周期跟调用它的对象一致,GC 友好。AnimationPlayer 是"持久的、可被设计师反复改的资源"。**两者不冲突,可以混用**。

### 3.2 常用 Tween 模式

```gdscript
# 平移
var tween := create_tween()
tween.tween_property(node, "position", Vector2(100, 200), 0.5) \
    .set_trans(Tween.TRANS_CUBIC) \
    .set_ease(Tween.EASE_OUT)

# 循环(震动屏幕 / 无限漂浮)
tween.set_loops()  # 默认无限循环
tween.tween_property(camera, "position:x", 0.1, 0.05)
tween.tween_property(camera, "position:x", -0.1, 0.05)

# 串联多个动画
tween.tween_callback(func(): print("前一个结束")).set_delay(0.2)
tween.tween_property(node, "modulate:a", 0.0, 0.5)  # 渐隐
```

`TRANS_CUBIC` + `EASE_OUT` 是游戏里最常用的组合——**起步快、结尾慢**,看起来"丝滑"。默认的 `TRANS_LINEAR` 是机械感,游戏 UI 几乎不用。

### 3.3 Tween 杀进程坑

```gdscript
# ❌ 这样会内存泄漏
for i in range(100):
    create_tween().tween_property(node, "x", i, 0.1)

# ✅ 用 kill() 或 tween 自动释放
var tween := create_tween()
# ... 触发后如果节点被 free,Godot 会自动 kill
```

---

## 4. AnimationTree 状态机:让角色"自动切动画"

这是动画的**重头戏**。前两节你学的是"录一段动作放"和"代码触发一段动作"。**但实际游戏里角色不会只播一个动画**——他站着是 Idle,跑起来是 Run,跳起来是 Jump,挨打是 Hit……**这些动画之间怎么切换、过渡多久、能不能被新输入打断**,就是状态机的工作。

### 4.1 搭一个"假人" 3D 角色

**步骤 1**:新建 `Node3D` 根节点 `Player`,存为 `player.tscn`。

**步骤 2**:在 `Player` 下加 `CharacterBody3D` (别用 CSGBox,**用 CharacterBody3D 是为后面接 06-20 的角色控制器**)。位置 `(0, 1, 0)`。

**步骤 3**:在 `CharacterBody3D` 下加 `CollisionShape3D`,`Shape` 选 `New CapsuleShape3D`,`Radius = 0.4`,`Height = 1.6`。

**步骤 4**:在 `CharacterBody3D` 下加 `CSGBox3D` 作为"身体"(占位),尺寸 `(0.8, 1.6, 0.8)`,位置 `(0, 0, 0)`。这样你**有体积可以撞、有网格可以看**。

**步骤 5**:在 `Player` 下加 `AnimationPlayer` 节点。我们马上要录三段动画。

### 4.2 录三段动画:Idle / Run / Jump

#### 4.2.1 Idle(待机,微微浮动)

在 `AnimationPlayer` 里点"新建动画",名字 `idle`,长度拉到 `2.0s`。在 `0s` 和 `2.0s` 都给 `CSGBox3D:position.y` 插关键帧,值都是 `0`;在 `1.0s` 插一个关键帧,`position.y = 0.05`(微微浮起来一点)。播一下会看到身体**轻微上下浮动**,这表示"活着"。

#### 4.2.2 Run(跑步,身体前后倾斜)

新建动画 `run`,长度 `0.6s`。轨道:
- `0.0s` → `position.z = 0`,`rotation.x = 0`
- `0.3s` → `position.z = -0.2`(身体前倾),`rotation.x = -0.1`
- `0.6s` → `position.z = 0`,`rotation.x = 0`

循环播放就是"跑步姿势循环"。

#### 4.2.3 Jump(起跳,被"打"上去再回落)

新建动画 `jump`,长度 `0.8s`。轨道:
- `0.0s` → `position.y = 0`,`scale = (1, 1, 1)`
- `0.2s` → `position.y = 1.5`(被弹起),`scale = (0.8, 1.2, 0.8)`(挤扁)
- `0.5s` → `position.y = 0.5`(滞空下落),`scale = (1, 1, 1)`
- `0.8s` → `position.y = 0`,`scale = (1.1, 0.9, 1.1)`(落地压缩)

> 💡 **为什么要改 scale?** 这是动画"重量感"的经典技巧——**起跳挤扁、滞空拉长、落地压扁**。不学这招,跳起来就是"线性上下",看起来像根萝卜。

### 4.3 接 AnimationTree 状态机

**步骤 1**:在 `Player` 下加 `AnimationTree` 节点。

**步骤 2**:选中 `AnimationTree`,看 Inspector:
- `Anim Player` 选 `../AnimationPlayer`(指向我们刚录的播放器)
- `Tree Root` 默认是 `AnimationNodeBlendTree`,**改成 `AnimationNodeStateMachine`**

**步骤 3**:在编辑器下方,会看到一个新的"State Machine"面板。默认有两个节点:`Start`(绿色,起点)和 `End`(红色,空)。**在空白处右键** → `Add State` → 名字 `idle`,弹窗选 `Animation` → `idle` 那个 `.anim` 资源。同理加 `run`(绑 `run` 动画)和 `jump`(绑 `jump` 动画)。

**步骤 4**:从 `Start` 拖一根线到 `idle`(进入游戏默认是待机)。

**步骤 5**:**加转换(Transition)**。在 `idle` 上右键 → `Add Transition` → 拖到 `run`。在弹窗里把 `Switch Mode` 改成 `Immediate`(立即切,适合动效不打断的场景),`Auto Advance` 留空。**但要加条件**——点 `idle → run` 那条线,看 Inspector,展开 `Advance Condition` 区域,点 `+` 加条件,`parameter` 填 `parameters/conditions/is_running`,Type = `bool`,值 `true`。意思是"当 `is_running = true` 时,自动从 idle 切到 run"。

**步骤 6**:同理加:
- `idle → jump`:条件 `is_jumping = true`
- `run → idle`:条件 `is_running = false`
- `run → jump`:条件 `is_jumping = true`
- `jump → idle`:条件 `is_jumping = false`(落回地面后回 idle)

**步骤 7**:**启用状态机**。`AnimationTree` 节点有个 `Active` 属性,**必须勾上**才生效。

### 4.4 写脚本:根据角色状态切换条件

挂在 `Player` 上的脚本:

```gdscript
# player.gd - 3D 角色 + 动画状态机
extends Node3D

@onready var body: CharacterBody3D = $CharacterBody3D
@onready var anim_tree: AnimationTree = $AnimationTree

const SPEED := 5.0
const JUMP_VELOCITY := 6.0
const GRAVITY := 12.0

var velocity: Vector3 = Vector3.ZERO
var is_jumping: bool = false

func _ready() -> void:
    anim_tree.active = true  # 必须显式启用

func _physics_process(delta: float) -> void:
    # 1. 读输入
    var input_dir := Vector3.ZERO
    input_dir.x = Input.get_axis("move_left", "move_right")
    input_dir.z = Input.get_axis("move_forward", "move_back")
    var is_running := input_dir.length() > 0.01

    # 2. 应用重力
    if not body.is_on_floor():
        velocity.y -= GRAVITY * delta
    else:
        if is_jumping:
            is_jumping = false  # 落地了

    # 3. 跳跃
    if Input.is_action_just_pressed("jump") and body.is_on_floor():
        velocity.y = JUMP_VELOCITY
        is_jumping = true

    # 4. 移动
    velocity.x = input_dir.x * SPEED
    velocity.z = input_dir.z * SPEED
    body.move_and_slide()

    # 5. 把状态喂给动画树
    anim_tree.set("parameters/conditions/is_running", is_running)
    anim_tree.set("parameters/conditions/is_jumping", is_jumping)
```

> 💡 **为什么要用 `parameters/conditions/...` 这种字符串路径?** 因为 `AnimationTree` 的所有"状态机参数"都暴露为 `AnimationTree` 节点的属性,**通过 `set("parameters/...", value)` 动态写值**。这是 Godot 状态机的标准做法,也是它和 Unity Animator 的 `SetBool/SetTrigger` 类似的核心 API。

### 4.5 跑起来

**步骤 1**:在 Project Settings → Input Map 里,确认 `move_left`(A 键)、`move_right`(D 键)、`move_forward`(W 键)、`move_back`(S 键)、`jump`(空格键)都有定义(默认模板有,不用改)。

**步骤 2**:实例化 Player 到 world.tscn。在 world.tscn 里把 player.tscn 拖进去,位置 `(0, 0, 0)`。给场景加个地面(CSGBox3D 大薄板 20×1×20)。

**步骤 3**:F5 跑。WASD 移动 → 角色进入 Run 动画;松开 → 回 Idle;按空格 → 切 Jump。

> 💡 **看动画"过渡"**:状态机面板里点 `idle → run` 那条线,有个 `xfade time` 字段(默认 0)。**改成 0.15**,你会看到两个动画之间有 0.15 秒的平滑过渡——这是"游戏感"的关键。

---

## 5. 综合实战:把三件套用起来

把今天学的"串"成一个完整小 Demo。完整场景结构:

```
GameWorld (Node3D)
├── Ground (CSGBox3D, 20×1×20, position.y=-0.5)
├── DirectionalLight3D
├── Camera3D (Current, position=(0, 5, 10))
├── WorldEnvironment
├── Player (instance of player.tscn)
└── UILayer (CanvasLayer)
    └── StartButton (Button, text="开始游戏")
```

```gdscript
# game_world.gd - 场景根脚本
extends Node3D

@onready var player: Node3D = $Player
@onready var start_button: Button = $UILayer/StartButton

func _ready() -> void:
    start_button.pressed.connect(_on_start_pressed)

func _on_start_pressed() -> void:
    # 1. 按钮弹一下(Tween)
    var tween := create_tween()
    tween.tween_property(start_button, "scale", Vector2(0.8, 0.8), 0.1)
    tween.tween_property(start_button, "scale", Vector2.ONE, 0.1) \
        .set_ease(Tween.EASE_OUT)
    tween.tween_callback(_start_game)

func _start_game() -> void:
    # 2. 玩家起跳一次(AnimationPlayer 触发动画)
    player.get_node("AnimationPlayer").play("bounce")
    # 3. 玩家进入 Run 状态(AnimationTree 切)
    await get_tree().create_timer(1.5).timeout
    player.get_node("AnimationTree").set("parameters/conditions/is_running", true)
```

> 💡 **为什么要用 `await`?** 让"按钮弹完 → 角色起跳 → 角色开始跑"形成一个有时间顺序的**剧情链**。这是"叙事感游戏"的基础。

---

## 6. 三个常见坑点 + 解决方案

### 坑 1:动画播不出来 / 节点不变

**症状**:F5 跑,`AnimationPlayer.play("xxx")` 没报错,但场景里节点纹丝不动。

**排查**:
1. 检查动画名是否拼写错误(大小写敏感)
2. 检查轨道上**目标路径**是否对(比如 `CSGBox3D:position` 这个 `CSGBox3D` 是节点名,如果后来改了名,轨道就找不到对象了)
3. 检查 `AnimationTree` 节点**是否启用了 `Active`** —— 状态机的"沉默杀手"

**解决**:在动画面板里点"放大镜"图标(Track Path Editor),看每条轨道的目标节点是不是依然能解析到。**改名后一定要回来点 "重绑路径"(Re-Path)**。

### 坑 2:状态机条件设了但切不过去

**症状**:`is_running = true` 写了,角色还是不动。

**排查**:
1. `Advance Condition` 里的 `parameter` 字符串要**严格匹配** `parameters/conditions/<name>`,前缀 `parameters/` 经常被忘掉
2. 状态机的转换线**必须连对方向**(`idle → run` 和 `run → idle` 是两条不同的线)
3. `Auto Advance` 开了的话,会**忽略条件自动切**,把不想要自动切的状态机里关掉

**解决**:`AnimationTree` 节点有运行时调试面板 —— 在编辑器里点 "调试"(Debugger) → 选 AnimationTree,看 `is_running` 当前值。生产代码里用 `print(anim_tree.get("parameters/conditions/is_running"))` 也能打出来。

### 坑 3:Tween 节点被 free 后还在跑

**症状**:角色死亡被 `queue_free()`,但屏幕上还在飘字、UI 还在弹。

**排查**:Tween 默认绑在调用它的节点上,节点 free 时**会**自动 kill —— 但**如果 Tween 在回调里又创建了脱离父节点的资源**(比如 `TweenProperty` 改全局节点的属性),这条链不会自动释放。

**解决**:
```gdscript
# 显式保存 Tween 引用,在 _exit_tree 里 kill
var _tween: Tween

func _ready() -> void:
    _tween = create_tween()
    _tween.tween_property(...)

func _exit_tree() -> void:
    if _tween and _tween.is_valid():
        _tween.kill()
```

---

## 7. 三个延伸阅读链接

1. **Godot 官方文档 - Animation 概述**:<https://docs.godotengine.org/en/stable/tutorials/animation/index.html> —— 必读,包含 AnimationPlayer / AnimationTree / Tween 三件套的完整 API
2. **Heartbeast 的 "Godot 4 AnimationTree 教程"(YouTube)**:<https://www.youtube.com/results?search_query=heartbeast+animationtree> —— 一个 30 分钟视频带做完整 2D/3D 角色状态机,跟着做一次就懂
3. **GDQuest 的 "State Machines in Godot 4"**:<https://www.gdquest.com/tutorial/godot/design-patterns/finite-state-machine/> —— 状态机的设计模式专题,把"为什么状态机比 if/else 链好"讲得很透

---

## 8. 软件版本 & 数据来源

- **Godot**: 4.6(2026 年 6 月稳定版,`ENetMultiplayerPeer` / `AnimationPlayer` / `AnimationTree` API 沿用 4.x 体系,语法与 4.0+ 兼容)
- **Blender**: 本篇未涉及
- **数据来源**:Godot 官方文档 + Heartbeast / GDQuest 公开教程(2026 年 6 月访问)+ 个人 06-20 角色控制器的实战经验
- **下一篇预告**:6-25 以后会进入"接口打磨"阶段(着色器 / GDScript 进阶 / UI),6-29+ 起将用 2-3 篇做"Blender 建模 → 导出 glTF → Godot 接骨骼 → 联机跑起来"的**端到端综合实战**,把你前面 12 篇学的东西串成一条完整工作流

---

> ✍️ **写给 breeze**:动画这件事,**别一上来就想着"做一段超酷的攻击连招"**。先做"待机 → 跑 → 跳"这三段,把状态机的循环跑通,你就理解了 80% 的游戏动画架构。后面学 Blender 骨骼、Mixamo 自动骨骼、IK 反向动力学,都是在这个基础上的"加法"。
