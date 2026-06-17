---
title: "Blender & Godot 3D 教程 - 2026-06-18 - Godot 4 3D 基础:搭一个有光、有相机、能转的 3D 场景"
date: 2026-06-18T04:00:00+08:00
description: "零基础打开 Godot 4.6 的 3D 世界:理解 MeshInstance3D / Camera3D / DirectionalLight3D / WorldEnvironment 四件套,做出第一个能 WASD 走、鼠标转视角、有太阳光和蓝天的 3D 场景。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-18.png
  alt: "Godot 4 3D 场景中一个低多边形小方块站在带阴影的地面上,蓝天太阳"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-18.png)

> 系列第七篇。前两天我们造了模型、上了材质;06-14 讲了 Node / Scene / Instancing 那套"一切皆节点"的抽象。但到现在,**我们还没在 Godot 里真正搭过一个"有光、有相机、能看"的三维场景**。今天补上这一环:打开 Godot → 摆一盏太阳 → 立几块方块 → 架个相机 → 用 WASD 走两步 → 鼠标转视角。30 分钟,从空白工程到一个能"玩"起来的 3D 小场景。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 在 Godot 4 里新建一个 3D 场景,看明白 `MeshInstance3D` / `Camera3D` / `DirectionalLight3D` / `WorldEnvironment` 四个节点的分工
- 给场景换天空、调太阳角度、让阴影出现
- 用 `CSGShape3D` 在场景里"布尔"出几个方块、挖个洞
- 写 30 行 GDScript 实现 WASD 移动 + 鼠标视角控制
- 理解为什么 Godot 4 默认用 **前向 (Forward+) 渲染管线**

**前置:**

- Godot 4.6 已安装并能打开(没装的话去 [godotengine.org](https://godotengine.org/) 下一个,完全免费)
- 看过 06-14,知道 Node / Scene / 实例化是什么
- 不需要 Blender,今天我们只用 Godot 自带的 CSG 几何体——但你昨天做的那个低多边形房子如果想塞进来,文末会教你怎么做

## 1. 为什么"3D 基础"要单独拿一篇出来?

2D 时代,一个 `Node2D` + 一个 `Sprite2D` + 一段 `move_and_slide` 就能开干。3D 时代不行——你最少需要 **4 个节点** 才能看到一个像样的画面:

| 节点 | 2D 时代对应 | 干什么的 |
|---|---|---|
| `MeshInstance3D` | `Sprite2D` | 把你看见的"几何形状"放进去(立方体、球、模型) |
| `Camera3D` | `Camera2D` | 决定你"从哪个角度"看 |
| `DirectionalLight3D` | 没对应,2D 默认就有光 | 模拟太阳光,产生明暗和阴影 |
| `WorldEnvironment` | `WorldEnvironment`(2D 也有) | 控制"天空、雾、环境光"等全局环境 |

少一个,画面要么全黑、要么全白、要么啥也看不见。这一篇我们就把这四件套一次配齐。

## 2. 新建一个 3D 场景

**步骤 1**:打开 Godot,`项目管理器(Project Manager) → 新建(New Project) → 选个文件夹 → 创建并编辑(Create & Edit)`。

**步骤 2**:在场景面板(Scene 面板,左上角)点 `+ 其他节点(Other Node)` → 搜 `Node3D` → 创建。**注意:不要选 Node,Node 是 2D / 3D 通用的纯逻辑节点,没有 transform,摆到场景里你也看不见坐标轴。**

> 💡 **为什么要从 Node3D 开始?**
> Godot 的根节点类型决定了整个场景的"维度":`Node2D` 的子节点默认走 2D 物理和 2D 渲染;`Node3D` 的子节点才能用 3D transform、3D 物理、3D 碰撞。**根节点选错,后面所有节点都要重命名,bug 排到哭。**

**步骤 3**:把根节点重命名成 `Main`,Ctrl + S 存为 `main.tscn`(项目根目录或自建 `scenes/` 文件夹都行)。

## 3. WorldEnvironment:给场景一个"世界"

不挂 WorldEnvironment,场景会用一个很难看的纯色背景,而且没有环境光。

**步骤 4**:在 Main 下点 `+` → 搜 `WorldEnvironment` → 添加。

**步骤 5**:选中 `WorldEnvironment`,在 Inspector 面板(右侧)找到 `Environment` 属性。**点一下那个空槽,Godot 会弹窗让你"新建(New Environment)"或"加载(Load)";选新建。**

**步骤 6**:在弹出的 `environment.tres` 子资源里,展开 `Background` 模式(Background Mode),把默认的 `Clear Color` 改成 `Sky`(天空)。

**步骤 7**:`Sky` 下拉里选 `New Sky`,会再生成一个 `sky.tres` 子资源。展开 `Sky` 资源,你会看到一个 `Sky` 节点(不是文件,是个逻辑节点)——再点 `+` 添加 `ProceduralSkyMaterial`(程序化天空材质)。这时候 viewport 应该有蓝色天空 + 太阳了。

> 💡 **为什么用 ProceduralSkyMaterial 不用 HDRI?**
> ProceduralSkyMaterial 是 Godot 用公式算出来的"假天空",**零文件、零依赖、加载 0 毫秒**。新手 / 原型阶段永远用它。等项目要出片了再去 [polyhaven.com](https://polyhaven.com/hdris) 下免费 HDRI 替换掉。HDRI 替换方法:把 ProceduralSkyMaterial 删了,在 Sky 资源里加 `PanoramaSkyMaterial`,`Shader Parameters/Sky Texture` 拖一张 `.hdr` 进去。

**步骤 8(可选)**:在 Environment 资源里把 `Ambient Light` 打开,`Source` 选 `Sky`,`Energy` 调到 `0.3`。这表示"场景里没有被太阳直接照到的地方,也会被天空的散射光微微照亮",画面立马不会黑乎乎。

## 4. DirectionalLight3D:放一盏太阳

不挂 DirectionalLight3D,场景里所有物体都是均匀亮的,**没有明暗、没有体积感、没有阴影**。

**步骤 9**:在 Main 下 `+` → 搜 `DirectionalLight3D` → 添加。

**步骤 10**:选中这个节点,看 Inspector 顶部有个 `Rotate` 小工具(和其他 3D 物体一样,红绿蓝三个环),把蓝色环往上拖一点(让"太阳"斜射下来)。你也可以直接在 Inspector 里改 `Rotation` 数值,比如 `X: -45°` 就是一个下午的太阳角度。

**步骤 11**:勾上 `Shadow` → `Enabled`。瞬间你就能看见方块的影子投到地上了。

> 💡 **为什么 DirectionalLight3D 的"位置"不重要?**
> 这种光模拟的是"无限远的平行光"(太阳),所以不管你把它放 `(0,100,0)` 还是 `(1000000, 1000000, 0)`,打出来的光线方向都是平行的。**真正决定光线方向的是它的 Rotation,不是 Position。** 这一点和点光源 `OmniLight3D` 完全不同。

## 5. MeshInstance3D:把"几何体"放进去

**步骤 12**:在 Main 下 `+` → 搜 `MeshInstance3D` → 添加。

**步骤 13**:选中 `MeshInstance3D`,Inspector 里有 `Mesh` 属性,点空槽 → `New BoxMesh`(新建一个立方体网格)。你会看到场景里出现一个白色立方体,**尺寸默认 2×2×2**,位于原点。

**步骤 14**:用 Inspector 改 `Transform` → `Position` 为 `(0, 1, 0)`(抬高一点,免得沉到地下),`Scale` 留默认。

**步骤 15**:再添加两个 `MeshInstance3D`,分别给它们 `New SphereMesh` 和 `New CylinderMesh`,位置随便摆,比如 `(-3, 1, 0)` 和 `(3, 1, 0)`。

> 💡 **为什么 MeshInstance3D 要单独搞一个 Mesh 资源?**
> 这是 06-14 讲的"组合优于继承"的实践:同一个 `BoxMesh` 资源可以被 N 个 `MeshInstance3D` 共享,改一个地方所有方块都变。**做场景的时候把"形状"和"实例"分开,后期改起来省一半时间。**

## 6. 铺一块地面

场景浮空太丑,铺个地板。

**步骤 16**:再添加一个 `MeshInstance3D`,Mesh 选 `New BoxMesh`,但这次改它的 `Size` 属性为 `(50, 0.2, 50)`(又大又扁),`Position` 为 `(0, -0.1, 0)`(稍微下沉一点,让上面站着的物体脚不陷进去)。

**步骤 17**:给地面改个材质,免得全是白塑料:在 MeshInstance3D 里找到 `Material Override` → `New StandardMaterial3D` → 展开 `Albedo` → Color 改个浅灰(`#a8a8a8` 就行)。

## 7. Camera3D:架个相机

不挂 Camera3D 的话,viewport 里你看到一个**带坐标轴的默认测试相机**,但它不会跟着你的代码动。手动加一个:

**步骤 18**:`+` → `Camera3D`,`Position` 设 `(0, 2, 5)`(站在小方块后面两米,身高两米),`Rotation` 的 `X` 设 `-10`(稍微往下俯一点点)。

**步骤 19**:在 Inspector 里把 `Current` 勾上(就是 `current = true`)。**如果场景里同时存在多个 Camera3D,只有 current=true 的那个会被渲染。**

> 💡 **为什么 Godot 没有"主相机"概念,要靠 current 标记?**
> 灵活。游戏里经常需要在多个相机之间切换(过场动画、监控视角、第一人称/第三人称),把"哪台相机正在用"做成运行时属性,切起来就是一行 `camera.make_current()`。

## 8. 跑一下,看效果

按 F5(或者工具栏 ▶️),Godot 会让你选主场景,选 `main.tscn`。你会看到一个**蓝天下、立着三个白色几何体、地面带阴影**的小场景。鼠标按住右键拖动可以转视角,滚轮缩放。

如果地面看不见,八成是相机位置太矮或者方向不对,按 F1 看 gizmo 调一调。

## 9. 让它动起来:WASD 移动 + 鼠标视角

现在它是个雕塑,不是个游戏。我们给它加个"玩家"。

**步骤 20**:新建一个 `CharacterBody3D`,改名为 `Player`,`Position` 设 `(0, 1, 0)`。

**步骤 21**:在 Player 下加一个 `MeshInstance3D`,Mesh 选 `New CapsuleMesh`,`Scale` 设 `(0.7, 0.7, 0.7)`,给它的 `Material Override` → `New StandardMaterial3D` → Albedo 改成红色 `#cc3333`(让玩家能从一堆白方块里认出来)。

**步骤 22**:在 Player 下加一个 `Camera3D`,`Position` `(0, 0.5, 0)`(头的高度),`Rotation` 的 `X` 设 `-15` 度(稍微往下看)。**记得勾 Current。**

**步骤 23**:在 Player 下加一个 `CollisionShape3D`,`Shape` → `New CapsuleShape3D`,`Radius` `0.4`,`Height` `1.6`。**这是物理碰撞,没它就走不动也撞不了东西。**

> 💡 **为什么用 CharacterBody3D 不用 RigidBody3D?**
> CharacterBody3D 是给"玩家"用的——**物理引擎不会自动推它**,全靠你代码里的 `move_and_slide()` 控制;RigidBody3D 是给"箱子、球、可被推动的物体"用的,物理引擎会接管它。**搞反了,你写的角色会像皮球一样满地滚。**

**步骤 24**:Player 节点上挂一个 GDScript,右键 → `Attach Script`,文件名叫 `player.gd`。粘贴下面这段:

```gdscript
extends CharacterBody3D

# 鼠标灵敏度
@export var mouse_sensitivity := 0.0025
# 玩家移动速度(米/秒)
@export var speed := 5.0
# 跳跃速度
@export var jump_velocity := 5.0

# 拿到挂在子节点上的相机
@onready var camera: Camera3D = $Camera3D

# 累积的 Y 轴(水平)旋转和 X 轴(俯仰)旋转
var _yaw := 0.0
var _pitch := 0.0


func _ready() -> void:
	# 鼠标独占模式:鼠标光标隐藏并锁在窗口中心
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED


func _unhandled_input(event: InputEvent) -> void:
	# 鼠标移动事件
	if event is InputEventMouseMotion:
		_yaw -= event.relative.x * mouse_sensitivity
		_pitch -= event.relative.y * mouse_sensitivity
		# 限制俯仰角,别让人头能转到屁股后面去
		_pitch = clamp(_pitch, -1.4, 1.4)
		# 应用旋转:Y 是水平(转一圈),X 是俯仰(低头抬头)
		rotation.y = _yaw
		camera.rotation.x = _pitch

	# ESC 释放鼠标;鼠标点击重新捕获
	if event.is_action_pressed("ui_cancel"):
		Input.mouse_mode = Input.MOUSE_MODE_VISIBLE
	if event is InputEventMouseButton and event.pressed:
		if Input.mouse_mode == Input.MOUSE_MODE_VISIBLE:
			Input.mouse_mode = Input.MOUSE_MODE_CAPTURED


func _physics_process(delta: float) -> void:
	# 1. 处理重力
	if not is_on_floor():
		velocity.y -= 9.8 * delta

	# 2. 处理跳跃(空格)
	if Input.is_action_just_pressed("ui_accept") and is_on_floor():
		velocity.y = jump_velocity

	# 3. 处理 WASD 输入
	var input_dir := Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	# 注意:Godot 的 forward 是 -Z 方向,所以基础方向向量直接用就行
	var direction := (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

	# 4. 实际移动
	if direction:
		velocity.x = direction.x * speed
		velocity.z = direction.z * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
		velocity.z = move_toward(velocity.z, 0, speed)

	# 5. move_and_slide 是 CharacterBody3D 的内置方法,会自动处理撞墙/撞地
	move_and_slide()
```

**步骤 25**:回到 Main 场景(把 Main 节点选中,或者场景面板里点 `main.tscn` 标签),**把 Player 拖到 Main 下面**,运行。

你应该能:
- WASD 移动
- 鼠标转视角
- 空格跳
- 撞上前面摆的几个白方块会停下(物理碰撞生效)
- 按 ESC 释放鼠标,点窗口重新捕获

> 💡 **为什么用 `transform.basis * Vector3(...)` 而不是直接 `Vector3(input_dir.x, 0, input_dir.y)`?**
> 直接用的话,角色转身后"按 W"还是往世界 -Z 走,但你已经背对它了。乘以 `transform.basis` 后,输入会跟着角色的朝向走——**这才是第一人称该有的"W 一直向前"。**

## 10. (可选)把昨天 Blender 做的房子塞进来

06-17 我们在 Blender 里建过低多边形房子,导出 `.glb` 了吗?没导的话回 6.17 看"导出"那一节,导出时勾上 **"Apply Modifiers"** 和 **"UV"**。

**步骤 26**:在 Godot 里 `FileSystem` 面板(左下)找到 `house.glb`,**双击它**——Godot 会自动生成一个 `.glb.import` 文件,并在场景根目录出现一个同名 `.tscn`(house.tscn)。**这个 .tscn 就是 .glb 的"Godot 场景版"**,把它直接拖进 Main 场景就能用。

> 💡 **为什么要双击?**
> 双击 .glb 是 Godot 的"导入"动作——它会读 .glb 的网格 / 材质 / UV,**并把贴图自动转成 Godot 的 .ctex 格式**(压缩纹理,加载更快)。跳过这步直接拖 .glb 也能用,但首次加载会现场转格式,卡顿 1-2 秒。

## 11. 常见坑点

### 坑 1:场景全黑 / 全白 / 一片荧光色

**原因 99% 是 WorldEnvironment 没配**。你看到的是 Godot 默认的 fallback 渲染。检查 `WorldEnvironment` 节点的 `Environment` 槽有没有挂环境资源;环境资源里的 `Background Mode` 是不是 `Sky`;`Sky` 资源里有没有 `ProceduralSkyMaterial`。

### 坑 2:物体有颜色但没影子

**原因 1**:DirectionalLight3D 没勾 `Shadow → Enabled`。
**原因 2**:项目的全局阴影设置没开。`项目(Project) → 项目设置(Project Settings) → Rendering → Global Illumination → SDFGI` 或 `VoxelGI` 没启用的话,大面积阴影还是会丢。**最稳的方案**:在场景里加一个 `VoxelGI` 节点(用 `+` → 搜 `VoxelGI`),覆盖到 Main 范围,场景自动计算"环境光遮蔽"。

### 坑 3:WASD 跑得飞快,撞墙像撞棉花

**原因**:你没加 `CollisionShape3D`,或者 `CollisionShape3D` 的 `Shape` 槽是空的。`CharacterBody3D` 没碰撞体的话,`move_and_slide` 找不到任何东西可以"滑",所有方向都是真空。

### 坑 4:Camera 鼠标拖一下就乱转

**原因**:你把 `Camera3D` 旋转写在 `transform.basis` 上了,但 `CharacterBody3D` 的 `transform` 是物理引擎管的。**永远用 `rotation.y = _yaw` 这种"直接赋值"**,别用 `rotate_y()`(那是增量,会累积到天上去)。

### 坑 5:导入的 .glb 模型"穿模"到地面下面

**原因**:Blender 里模型原点不在 (0,0,0),或者导出时没勾 `Apply Transforms`。回 Blender 里 `Object → Apply → All Transforms`,再导出。

## 12. 延伸阅读

- [Godot 4 官方文档 — 3D 入门](https://docs.godotengine.org/en/stable/tutorials/3d/introduction_to_3d.html):和你今天看到的内容一样的官方版,英文但有图
- [Godot 4 官方文档 — WorldEnvironment](https://docs.godotengine.org/en/stable/classes/class_worldenvironment.html):所有环境参数(雾、SSAO、Glow、SSR)的完整解释
- [GDQuest — Make a 3D Game with Godot 4](https://www.gdquest.com/tutorial/godot/4/3d-game/):免费 4 节 3D 入门视频,讲得比官方文档更口语化
- [KidsCanCode — Godot 3D Platformer](https://kidscancode.org/godot_recipes/4.x/3d/):一个完整 3D 平台跳跃 Demo,看完能做出"超级玛丽 3D 版"

## 13. 系列进度 & 下期预告

到目前为止系列进度:

| 篇 | 日期 | 大主题 | 子方向 |
|---|---|---|---|
| 1 | 06-12 | Blender | 界面与基础操作 |
| 2 | 06-13 | Blender | 基础建模(挤出 / 循环边 / 镜像) |
| 3 | 06-14 | Godot 3D | Node / Scene / Instancing |
| 4 | 06-15 | 联机 | 联机架构总览 |
| 5 | 06-16 | 联机 | ENet 高层多人 API |
| 6 | 06-17 | Blender | 材质与纹理(PBR / UV) |
| **7** | **今天** | **Godot 3D** | **3D 基础(MeshInstance3D / Camera / Light / Environment)** |

下期预告:**Godot 3D 物理**——给场景里的方块加 `RigidBody3D`,玩家走过会撞飞它们;给地面加 `StaticBody3D`,撞上去有"咚"的物理反馈。从"看场景"到"玩场景"。

---

**软件版本**:Blender 4.x / Godot 4.6(2026 年 6 月时点的最新稳定版)。**数据来源**:本文所有 API 与默认值基于 Godot 4.6 官方文档(2026-06 查询);`move_and_slide` / `MultiplayerSpawner` 等 API 在 Godot 4.0 ~ 4.6 行为一致,代码可直接复制到 4.x 任一版本使用。
