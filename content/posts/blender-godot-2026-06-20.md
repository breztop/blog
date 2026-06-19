---
title: "Blender & Godot 3D 教程 - 2026-06-20 - Godot 4 3D 物理零基础入门：RigidBody3D + CharacterBody3D + 碰撞体"
date: 2026-06-20T04:00:00+08:00
description: "零基础打开 Godot 4.6 物理世界:理解 StaticBody3D / RigidBody3D / CharacterBody3D 三种「碰东西」的方式,搞懂 CollisionShape3D + Shape 家族,做出一个能推箱子、能跳、能在半空翻滚物理块的 3D 小场景,并把 Layer / Mask 碰撞过滤讲清,为后面的联机物理同步铺路。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-20.png
  alt: "Godot 4 3D 场景中一个低多边形小方块角色站在带阴影的地面上,旁边堆着几个可推动的物理箱子,蓝天太阳"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-20.png)

> 系列第九篇。06-18 我们搭了"有光、有相机、能转"的 3D 场景——但那个场景里的方块是**钉死的**,WASD 走过去直接穿过去;跳一跳也跳不动。今天补上**物理这一环**:学会让方块掉下来、被推走、被撞飞,学会让玩家**真的站**在地上、真的撞到墙、真的能跳。30 分钟,从"看见场景"到"玩起来"。学完这篇,后面讲联机状态同步、做物理回滚才有地基。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 搞懂 Godot 4 里三种"会撞东西"的节点:`StaticBody3D` / `RigidBody3D` / `CharacterBody3D` 各自的分工
- 给任意节点挂 `CollisionShape3D` + 选对 `Shape`(盒子 / 球 / 胶囊)
- 用 30 行 GDScript 写出"WASD 走 + 空格跳 + 鼠标看"的角色控制器
- 让一堆方块掉在地上、被推、被撞飞,会自己翻滚落定
- 理解 `collision_layer` / `collision_mask` 这一对**最容易把人绕晕**的过滤开关
- 跑通一个**完整可玩**的 3D 小场景:玩家在平台上推箱子、跳上箱子、堆箱子

**前置:**

- Godot 4.6 已装能开
- 看完 06-18(知道 `MeshInstance3D` / `Camera3D` / `WorldEnvironment` 怎么摆)
- 知道 `Node3D` 是根节点(06-14),GDScript 能看懂 `@onready` / `func` / `if`(06-18 末尾)

## 1. 为什么"3D 物理"要单独拿一篇出来?

2D 时代,`KinematicBody2D` + `move_and_slide()` 一招鲜吃遍天。3D 时代 Godot 把"碰撞这件事"拆成了**三个不同性格的节点**——你**选错一个**,表现就完全不一样:

| 节点 | 性格 | 谁在算"怎么动" | 典型用途 |
|---|---|---|---|
| `StaticBody3D` | 不动 | Godot 不算,你摆哪就停哪 | 地面、墙、关卡几何 |
| `RigidBody3D` | 完全交给物理引擎 | Godot 用 `PhysicsServer` 算 | 箱子、球、爆破碎片、可被推的任何东西 |
| `CharacterBody3D` | 半托管:你写位移,Godot 算"有没有撞到" | 你写代码给 `velocity`,Godot 帮你"撞墙不穿过" | 玩家、NPC、可控敌人 |

> 💡 **为什么要拆三种?**
> 这不是 Godot 故弄玄虚,是物理世界的一个根本矛盾:**精确控制**(玩家跳多高)vs **真实感**(箱子掉下来会滚)用同一套逻辑写不出来。所以 Godot 把"被力学的物理引擎接管"和"我自己写位移"明确分开——你选 `RigidBody3D` 就别指望 100% 控制每一步;选 `CharacterBody3D` 就别指望它会自动被风刮跑。

## 2. CollisionShape3D:所有碰撞的"挂载点"

不管上面三种 Body 是谁,**都必须挂一个 `CollisionShape3D` 子节点,才能在物理世界里"有体积"**。光有 `MeshInstance3D` 看起来有形状,实际等于**透明**——角色会穿过去、子弹会穿过去。

**步骤 1**:新建一个 3D 场景,根节点 `Main (Node3D)`,Ctrl+S 存为 `main.tscn`。

**步骤 2**:在 `Main` 下加 `+` → 搜 `StaticBody3D` → 添加。

**步骤 3**:在新建的 `StaticBody3D` 下加 `+` → 搜 `CollisionShape3D` → 添加。

**步骤 4**:选中 `CollisionShape3D`,看 Inspector,有个 `Shape` 属性是空的。**点一下,弹窗选 `New BoxShape3D`(新建盒形碰撞)**。你会看到右边"Shape"槽里出现一个 `BoxShape3D` 资源,默认尺寸 `(2, 2, 2)` 米。

**步骤 5**:展开 `BoxShape3D` 资源,把 `Size` 改成 `(20, 1, 20)`——这是地面用的,20×20 米大薄板。

> 💡 **为什么 Shape 是资源,不能直接在节点上改?**
> Godot 把"形状数据"(`BoxShape3D` / `SphereShape3D` / `CapsuleShape3D` / `CylinderShape3D` / `ConvexPolygonShape3D` / `ConcavePolygonShape3D`)做成可复用资源——一个 `BoxShape3D` 资源可以被 100 个 `CollisionShape3D` 引用。改一个,100 个全跟着变。**初学阶段保持"一个节点一个独立 Shape"也行,够用。**

`CollisionShape3D` 本身有个 `Shape` 属性 + 一个 `Transform`(碰撞的本地偏移和旋转)。你可以让"视觉网格"和"碰撞形状"**完全错开**——比如视觉上是一把长矛,碰撞用一个细长的胶囊。

**步骤 6**:把 `StaticBody3D` 重命名成 `Ground`,`Position` 改成 `(0, 0, 0)`。再给 `Ground` 加一个 `+ MeshInstance3D` 子节点(放进去一个 `BoxMesh`,尺寸 `(20, 1, 20)`,材质用 `StandardMaterial3D` 调个草绿色)。这样**视觉上**和**物理上**都有这个地面了。

> 💡 **为什么 Mesh 和 CollisionShape 尺寸要一样?**
> 这一步**最容易翻车**:视觉网格是 `(20, 1, 20)` 米,碰撞形状也是 `(20, 1, 20)` 米——它们必须对得上,否则"看上去踩到了"但"物理上还悬空"或者反过来。**简单规则:视觉多大,碰撞多大,中心点对齐。** 真要做"看起来不规则、碰撞用简化形状"(比如一棵复杂的树,碰撞只用一个胶囊),那才需要刻意错开——新手阶段先别玩这个。

跑一下场景(F5),你应该看到一块绿地,站在上面啥也不会发生(因为还没东西能掉)。

## 3. RigidBody3D:会掉、会滚、被推会飞的物理体

`RigidBody3D` 是这一篇的"主角"——所有**你不直接控制、但要它在物理世界里有反应**的东西,都用它:箱子、桶、苹果、子弹、爆破碎片、推塔的木块。

**步骤 7**:在 `Main` 下加 `+` → `RigidBody3D` → 重命名成 `Box1`。

**步骤 8**:在 `Box1` 下加 `+` → `CollisionShape3D` → Shape 选 `New BoxShape3D`,Size `(1, 1, 1)`(1 米见方的盒子)。

**步骤 9**:在 `Box1` 下再加 `+` → `MeshInstance3D` → Mesh 选 `BoxMesh`,Size `(1, 1, 1)`,材质橙色。

**步骤 10**:把 `Box1` 的 `Position` 改成 `(0, 3, 0)`(3 米高,正好悬在地面上)。

**步骤 11(关键)**:F5 跑一下,你会看到橙色方块**从 3 米高掉到地上**,可能还会晃两下。这就是 Godot 内置物理引擎在干活——它**自动按重力**把方块拉下来,落到地面**自动停住**,还给你算了一点旋转阻尼让它**不会一直抖**。

> 💡 **为什么 RigidBody3D 不能用 `_process()` 改 `position`?**
> 这是 Godot 新手第一大坑。你会本能地写 `box.position = ...`,然后发现方块**瞬间穿地**或者**抽搐**——因为物理引擎是**独立线程、固定步长**算的,你每帧 60 次的赋值它根本来不及响应,最后两套逻辑打起来,物体就疯了。
>
> **正确做法**:
> - 想"持续给个力":用 `apply_central_force()` / `apply_impulse()`
> - 想"瞬移到某处但还要后续物理正常":用 `linear_velocity` / `angular_velocity`,或者先 `freeze = true` 移完再 `freeze = false`
> - 想"让它死掉不再参与物理":用 `freeze = true` 或者 `queue_free()`
> - **永远不要每帧直接 `position = xxx`**

**步骤 12**:再复制 4 个 `Box1`(Ctrl+D 选中按 4 次,或者右键 Duplicate Node),把它们在 Y=2~8 之间错开摆成一摞。F5 跑一下,你会看到一摞方块**自由落体堆到地上**——这就是"刚体物理"最经典的样子。

## 4. CharacterBody3D:玩家角色,半托管

玩家"自己"是不能用 `RigidBody3D` 的——因为它会被一个斜坡绊住,会自己滚下去,会撞一下墙就乱跳。玩家需要的是"**我说往哪走就往哪走,但撞墙自动停**"——这就是 `CharacterBody3D`。

**步骤 13**:在 `Main` 下加 `+` → `CharacterBody3D` → 重命名成 `Player`。

**步骤 14**:在 `Player` 下加 `+` → `CollisionShape3D` → Shape 选 `New CapsuleShape3D`(胶囊形碰撞,标准人形身高 1.8 米),Radius `0.4`,Height `1.8`。

**步骤 15**:在 `Player` 下加 `+` → `MeshInstance3D` → Mesh 选 `CapsuleMesh`,Radius `0.4`,Height `1.8`,材质蓝色(做区分)。

**步骤 16**:把 `Player` 的 `Position` 改成 `(0, 1, 5)`(站在地面上、距离方块堆 5 米远)。

**步骤 17**:在 `Player` 下加 `+` → `Camera3D`(挂在 Player 下面,而不是 Main 下面——这样**相机跟着角色走**,做成"第三人称/第一人称跟随")。把 `Camera3D` 的 `Position` 改成 `(0, 0.6, 0)`(眼睛高度 0.6 米,刚好在胶囊头顶下一点,做第一人称)。

> 💡 **为什么 Camera 挂在 Player 下面?**
> 把 Camera 放在 `Player` 子节点,意味着"**相机的世界坐标 = Player 的位置 + Camera 的本地偏移**"。Player 移动,Camera 自动跟;Player 旋转,Camera 自动转。**后面要做的"鼠标控制视角",其实就是旋转 Player 这个节点**——Camera 是乘客。

**步骤 18(核心代码)**:在 `Player` 上挂一个 GDScript,叫 `player.gd`:

```gdscript
extends CharacterBody3D

# 角色控制器:WASD 移动 + 空格跳 + 鼠标看

@export var move_speed := 5.0      # 移动速度 米/秒
@export var jump_velocity := 4.5   # 跳跃初速度 米/秒
@export var mouse_sensitivity := 0.002  # 鼠标灵敏度 弧度/像素
@export var gravity := 9.8         # 重力加速度 米/秒²

@onready var camera: Camera3D = $Camera3D

func _ready() -> void:
    # 进游戏时锁鼠标到窗口中央,做 FPS 视角
    Input.mouse_mode = Input.MOUSE_MODE_CAPTURED

func _unhandled_input(event: InputEvent) -> void:
    # 鼠标移动 → 旋转角色 Y 轴 / 相机 X 轴
    if event is InputEventMouseMotion:
        rotate_y(-event.relative.x * mouse_sensitivity)
        camera.rotate_x(-event.relative.y * mouse_sensitivity)
        # 限制相机上下角度,别让人把脖子转到天上去
        camera.rotation.x = clamp(camera.rotation.x, -PI/2, PI/2)
    # 按 Esc 释放鼠标,方便你点编辑器
    if event.is_action_pressed("ui_cancel"):
        Input.mouse_mode = Input.MOUSE_MODE_VISIBLE

func _physics_process(delta: float) -> void:
    # --- 重力 ---
    if not is_on_floor():
        velocity.y -= gravity * delta

    # --- 跳跃 ---
    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = jump_velocity

    # --- WASD 移动(基于角色朝向,不是世界朝向) ---
    var input_vec := Input.get_vector("move_left", "move_right", "move_forward", "move_back")
    # transform.basis 是当前节点的"朝向", * input_vec 得到世界方向上的速度
    var direction := (transform.basis * Vector3(input_vec.x, 0, input_vec.y)).normalized()
    velocity.x = direction.x * move_speed
    velocity.z = direction.z * move_speed

    # --- 关键 API:move_and_slide() 处理"撞墙不穿过" ---
    move_and_slide()
```

> 💡 **为什么用 `_physics_process` 不用 `_process`?**
> 物理计算要"**固定步长**"——默认 60 次/秒(可在项目设置 `Physics/Common/Physics Ticks Per Second` 改)。`_process` 跟着帧率走,144Hz 显示器下你每秒算 144 次物理位移,会和别的 60Hz 客户端对不上(联机会出 bug)。**所有和速度/位置相关的逻辑,必须放在 `_physics_process` 里。**

**步骤 19**:Godot 内置的输入动作(`ui_accept` / `ui_cancel` / `ui_left`...)默认就映射了键盘和手柄。但 `move_left/right/forward/back` **没有**——需要你自己加:

1. 顶部菜单 `Project → Project Settings → Input Map`
2. 右边 `Add` 按钮,Action Name 填 `move_forward` → `+` → 选 `Key Event` → 按 `W` → Save
3. 重复 3 次:`move_back` = `S`,`move_left` = `A`,`move_right` = `D`

> 💡 **为什么不用 `_input.is_action_pressed` 也不直接 `Input.is_key_pressed`?**
> 用 Input Map 的好处:**一份代码,键盘 / 手柄 / 触屏 / 自定义按键全支持**。以后玩家想"用手柄玩",你只要加一个 `Joypad Button` 事件到 `move_forward` 就行,代码不用动。

**步骤 20**:F5 跑一下,蓝色胶囊出生,鼠标被锁;WASD 走,Space 跳,鼠标转视角——你应该能**推到那堆橙色箱子,看着它们滚出去**;再 W 跳到箱子上,**站在那摞箱子上面**;按 Esc 释放鼠标。

🎉 **这就是一个完整的 3D 角色控制器 + 物理物体交互。**

## 5. Layer 与 Mask:碰撞过滤的"权限表"

现在场上只有玩家和箱子,你推一下能动——很爽。但你会立刻发现一个问题:**玩家自己会撞自己**、**两个箱子之间也会互相算碰撞**——这些**正常**;但你也会想:

- 想加一个"**只对玩家触发,不被箱子挡**"的收集物 → 怎么让"玩家层"和"收集物层"**互相认得**,又**不被物理推动**?
- 想做"**队友不互撞**"或"**友军子弹穿自己人**" → 联机时 100% 需要。

这就是 `collision_layer` 和 `collision_mask` 的用途。Godot 4 给**每个 3D 物理体 32 个层级**(0~31),用 32 位整数表示:

- **`collision_layer`**:我是谁(我**属于**哪几个层)
- **`collision_mask`**:我和谁(我**关心**哪几个层,其他的不算)

`mask & layer != 0` 时才会算碰撞。听起来抽象,直接用刚才的 4 个对象演示:

**步骤 21**:选中 `Player`,Inspector 找到 `Collision → Layer`,默认是 `1`(只在第 1 层)。把它改成 `2`(在第 2 层)。

**步骤 22**:选中 `Box1`,`Layer` 保持 `1`,**`Mask` 改成 `2`**(意思是"我(箱子)只关心第 2 层的对象(玩家),其他层我不理")。

**步骤 23**:F5 跑一下,你会发现:

- 箱子会**互相堆**(因为大家都默认在第 1 层,`Mask=1` 会和第 1 层互相算)
- 玩家**推得动**箱子(玩家在第 2 层,箱子的 Mask 包含第 2 层,认得)
- **但你走到箱子上,玩家会"穿过"箱子掉下去!**

**步骤 24**:把玩家的 `Mask` 改成 `1`(或 `1 + 2 = 3`)。再跑,玩家**又能站在箱子上了**。

**步骤 25**:再加一个"收集物"试试:`+ Area3D`(注意:Area3D 不挡路,只**触发事件**)→ 给它一个球形 `CollisionShape3D` → 把它放在 `Layer=4`,玩家 `Mask=4` → F5 走到它附近,可以看到 Inspector 的 `Monitoring` 监控状态在变(下一步教程我们会用 `body_entered` 信号触发"得分")。

**Layer / Mask 速查表**——把你现在的场景配成这样最舒服:

| 节点 | Layer(我是谁) | Mask(我和谁算) | 解释 |
|---|---|---|---|
| `Ground` (Static) | 1 | (留空 / 全 1) | 我是"地形层",所有物理体都要踩我 |
| `Player` (Character) | 2 | 1 + 4 = 5 | 我关心"地形"和"收集物",其他(比如别的玩家)暂时不管 |
| `Box1` (Rigid) | 1 | 1 + 2 = 3 | 我是"地形层",也认得玩家(被推得动) |
| `Coin` (Area) | 4 | (空) | 我是"收集物层",不主动碰谁,等别人撞我 |

> 💡 **为什么 Layer 用"位掩码"不用下拉选?**
> Inspector 里 `Layer` 那个 0~31 的勾选框,**点一下是位 0,再点一下是位 1...** 实际是 32 位二进制。`Layer = 1 | 2 = 3` 表示"我在第 0 位 + 第 1 位"。想加第 3 位,就在代码里写 `collision_layer = 1 | 2 | 4 = 7`。**习惯之后,你会觉得这比 Unity 的 Named Layer 灵活得多**——不用去 Project Settings 命名 32 个层,直接按位用。

## 6. 三个最容易踩的坑(以及怎么修)

**坑 1:玩家"卡墙里"或者"踩不到地上"**

现象:你把 Player 的 `Position` 摆到了 `(0, 0.5, 0)`,胶囊高度 1.8 米——**结果胶囊的中心在 Y=0.5,顶部在 Y=1.4,底部在 Y=-0.4,直接**插进了地面**。跑起来你会发现玩家疯狂抖、或者"被弹飞"。

**修法**:**`Position` 表示节点的 origin**,胶囊形的 origin 默认在**中心**,所以玩家**站地上需要 `Y = 胶囊高度 / 2 = 0.9`**。或者把胶囊的 origin 改到底部(Inspector 里 `CollisionShape3D` 的 `Transform` → 调整 `Position.Y = -胶囊高度/2`),然后 `Player.Position` 摆 `(0, 0, 0)` 就行。**规则:Position 是胶囊中心,不是脚。**

**坑 2:RigidBody3D 跑着跑着"睡着"了 / 穿地**

现象:你做了一个箱子静止在地上,过几秒它会**突然往下穿 1 米**,或者**抖一下**——这是 Godot 物理"睡眠"机制的副作用。`RigidBody3D` 在速度极小时会"睡眠"以省 CPU,被某个力唤醒时如果重力没重新计算就会**掉一帧**。

**修法**:1) 把 `RigidBody3D` 的 `Sleeping` 属性设到 `false` 关掉自动睡眠;2) 把 `Linear/Angular Damp` 调高一点(比如 `0.5`);3) 项目设置里 `Physics/3D/Sleep Threshold Linear/Angular` 调大一些,让物理"更晚才睡"。

**坑 3:`move_and_slide()` 不工作 / 角色穿墙**

现象:代码里写了 `move_and_slide()`,但角色直接**穿过 RigidBody3D 飞过去**。原因:`move_and_slide()` **只和 StaticBody3D、CharacterBody3D、TileMap 等"会做 kinematic 阻挡"的层算碰撞**,**不会**自动推 RigidBody3D。

**修法**:推 RigidBody3D,得在 `_physics_process` 里**手动检测** `move_and_slide()` 后,新位置是否"撞到了刚体"——Godot 提供了 `get_slide_collision(idx)` API。简化版本:

```gdscript
for i in get_slide_collision_count():
    var c := get_slide_collision(i)
    if c.get_collider() is RigidBody3D:
        c.get_collider().apply_central_impulse(-c.get_normal() * 0.5)
```

或者更优雅:用 `CharacterBody3D.pushable` 节点(4.x 才有的实验属性),或者干脆把"箱子"换成 `KinematicBody3D`(Godot 4 没有了)……

**实际工程建议**:**角色推箱子用 trigger / 区域事件**——把箱子改成"碰到玩家就给它一个 -normal * 速度的冲量",比物理撞更可控。这块后面讲联机会再展开。

## 7. 一个完整可玩的 Demo:10 行修成 50 行

如果上面都跑通了,试试这个完整 5 元素的场景:

```
Main (Node3D)
├── WorldEnvironment (天空)
├── DirectionalLight3D (太阳)
├── Ground (StaticBody3D + BoxShape3D 20x1x20 + BoxMesh 草绿)
├── Stair01~05 (StaticBody3D 5 级台阶,做出一个斜坡可以跳上去)
├── Box1~10 (RigidBody3D, 颜色随机,堆在台阶顶上)
└── Player (CharacterBody3D + CapsuleShape3D + MeshInstance3D + Camera3D + Script)
```

跑起来:WASD 走 → 跳上台阶 → 冲进箱子堆 → 看着橙色蓝色紫色的方块漫天飞 → 找最高的那摞站上去看风景。

## 8. 收尾 & 后面会讲什么

到这里,你已经**能做出一个能玩、能撞、能推的 3D 小场景了**。但下面这些还没讲,我们后面一篇篇来:

- **`Area3D` 触发器**:做收集物、伤害区域、关卡出口(0 阻力 0 反弹,只发信号)
- **RayCast3D**:做"脚底下到底有没有地",是 CharacterBody3D 判断 `is_on_floor` 之外的第二保险
- **Joints3D**:做链条、摆锤、可破坏桥
- **NavigationAgent3D**:做敌人 AI 寻路
- **多层 Layer / Mask 在联机时的应用**:**预测 vs 权威**——客户端用 layer=1 算"我自己的预测",服务端用 layer=2 算"真实世界"然后纠偏

下一篇我们讲 **Godot 4 着色器零基础入门**:`ShaderMaterial` + 空间着色器怎么写,做出水面、玻璃、火焰、卡通描边——零基础也能跟着做出"会动的水"。

## 9. 软件版本 & 数据来源

- 软件版本:**Godot 4.6(2026-06 当前最新稳定版)**
- 物理后端:Godot Physics 3D(默认,不是 Jolt;Jolt 在 4.4 后是实验性的)
- 数据来源时间:2026-06-20(Godot 官方文档 + 自测)

## 10. 延伸阅读

1. **[Godot 4 物理系统官方文档](https://docs.godotengine.org/en/stable/tutorials/physics/index.html)**——这一篇的"理论源头",有完整的"ray cast / collision shape / physics layers"几节
2. **[Godot 4 CharacterBody3D 完整示例](https://docs.godotengine.org/en/stable/tutorials/3d/using_character_body_3d.html)**——官方给的"走 + 跳 + 斜坡"完整 60 行代码,我们这篇的 `_physics_process` 就是从这精简来的
3. **[Brackeys "Godot 4 Physics Tutorial" YouTube 系列](https://www.youtube.com/results?search_query=brackeys+godot+4+physics)**(英文,30 分钟视频)——跟着做一遍比我们这篇更适合"看视频型"学习者
4. **[GDQuest "Physics Layers 教程"](https://www.gdquest.com/docs/guidelines/godot/godot3/physics-layers/)**——Layer / Mask 那一节我参考了他们的位运算图示
