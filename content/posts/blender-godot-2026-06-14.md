---
title: "Blender & Godot 3D 教程 - 2026-06-14 - Godot 4 核心抽象:Node / Scene / Instancing 三件套"
date: 2026-06-14T04:00:00+08:00
description: "零基础打开 Godot 4.6:看懂四大区域,理解一切皆节点、场景即模板、实例化即复用,做出第一个能跑起来的 3D 根场景。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-14.png
  alt: "Godot 4.6 3D 场景中网格化实例化箱子包围角色"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-14.png)

> 系列第三篇。前两篇我们用 Blender 4.x 做了一个低多边形房子,并导出 `.glb`。今天正式切到 **Godot 4.6** —— 这才是真正要把 3D 场景"跑起来"的引擎。Godot 是 3D 游戏里**最容易上手**的免费开源引擎,尤其 Node / Scene / Instancing 三个核心抽象一旦吃透,后面加物理、加脚本、做联机,都是顺水推舟。

## 一、为什么从 Blender 切到 Godot?

Blender 4.x 让我们能"捏"出模型和材质,但它本质上是一个 **DCC 工具**(Digital Content Creation,数字内容创作)——你做出来的是**资产(Asset)**,不是**游戏(Game)**。要把模型、灯光、相机、玩家角色、敌人、关卡逻辑**组装成一个可玩的 3D 世界**,需要一个**游戏引擎**。我们这个系列选 **Godot 4**:

- **完全免费 + 开源**(MIT 协议,无授权费、无分成)
- **真·轻量**:编辑器安装包约 90 MB,启动 1 秒,vs Unity 的 5 GB / Unreal 的 30 GB
- **GDScript 语言**和 Python 几乎一样,新手当天能写
- **3D 能力 4.x 大跳**:Godot 3.x 的 3D 一直被人诟病,4.x 用了 Vulkan 渲染、Forward+ 后端,**完全可以做 AAA 级项目**
- **.glb 导入零摩擦**:直接拖进 Godot,模型/材质/动画/碰撞体一次到位

> **小贴士**:Godot 在 4.5(2025-09)和 4.6(2026-01)做了一波大改:4.6 默认开启 **Modern 编辑器主题**、默认物理引擎换成 **Jolt Physics**、完整重写 **SSR 屏幕空间反射**。我们这篇写于 4.6,所有 UI 截图描述、快捷键都按 4.6 走。

## 二、下载与安装

1. 官网:<https://godotengine.org/download/>
2. **标准版(Standard)** —— 我们用 GDScript,选这个就够(`.NET` 版是给 C# 用户用的,体积更大)
3. 平台对应:
   - Windows:下载 `.exe`,免安装,双击运行
   - macOS:下载 `.dmg`,拖入 Applications
   - Linux:`.zip` 解压即用,或者用 Snap / Flatpak
4. **首次启动**会问"导入已有项目"还是"新建",选**新建**,路径选一个空文件夹(比如 `~/godot-projects/03-godot-intro`),项目名 `godot_intro`

> **为什么要这样做**:Godot 不像 Blender 把启动文件存在本地,Godot 是**整个项目 = 一个文件夹**。项目名、路径、配置(`project.godot`)都在这个文件夹里,**养成"一个项目一个文件夹"的习惯**,将来团队协作、上 Git、迁移电脑都顺。

## 三、五分钟看懂 Godot 编辑器四大区域

打开 Godot,你看到的窗口和 Blender 不一样——Godot 是**多面板浮动布局**,中间是 3D 视口,周围是各种 Dock 面板。**新手阶段锁定的四大区域**:

| 区域 | 默认位置 | 作用 |
|---|---|---|
| **Scene 面板** | 左侧 | 当前打开场景的**节点树**(类似 Blender 的 Outliner + 更多) |
| **FileSystem 面板** | 左下 | 项目所有文件(场景、脚本、贴图、模型) |
| **Inspector 面板** | 右侧 | 当前选中节点的**所有属性** |
| **Viewport 视口** | 中间最大块 | 3D/2D 实际渲染预览 |

> **小贴士**:右上角有一排图标(2D / 3D / Script / AssetLib 四个工作区标签)。**做 3D 永远停在 `3D` 标签**。顶部的菜单栏是 `Scene / Project / Debug / Editor` 四大类,记住 `Scene` 菜单就够 80% 的工作。

如果某个面板不小心关了,从**主菜单 `Editor → Editor Layout → Reset`** 可以一键复原。

## 四、概念一:Node —— "一切皆节点"

打开 Godot 你听到的第一句黑话:**Node**(节点)。Godot 整个世界由**节点**构成,节点是**所有可见/不可见东西的最基本单位**:

- 相机 = `Camera3D` 节点
- 立方体 = `MeshInstance3D` 节点
- 阳光 = `DirectionalLight3D` 节点
- 物理碰撞 = `CollisionShape3D` 节点
- UI 按钮 = `Button` 节点
- 声音播放器 = `AudioStreamPlayer` 节点
- **整个游戏 = 一个 `Node3D` 根节点 + 无数子节点**

> **为什么要这样做**:**树状结构**让 3D 场景里的"父子关系"非常自然——一个 `Player` 节点下挂一个 `Camera3D`,相机就自动跟着玩家移动;删一个父节点,子节点全部跟着删。这是引擎设计里 30 年验证的最佳实践。

### 4.1 加节点的两种姿势

**姿势 A:在 Scene 面板右键**
1. 选中任意节点
2. **右键 → Add Child Node**
3. 弹搜索框,输入 `MeshInstance3D` → 选第一个 → Create
4. 新节点出现在选中节点下面,默认带个名字 `MeshInstance3D`

**姿势 B:在 Viewport 用 + 图标**
1. 把鼠标停在 3D 视口
2. 左上角有一排小图标,其中 **`+`** 图标(或者快捷键 `Ctrl + A`)打开 Add Node 弹窗
3. 同上

> **坑点 1:加完节点它在 3D 视口里"找不到"?**
> 新节点默认坐标 `(0, 0, 0)`,如果场景里已经有别的物体,它们就重叠了。**选中节点 → Inspector → Transform → Position**,改成 `(0, 0, 5)` 把它挪开,或者按 `F`(Frame Selected)在视口里对焦它。

### 4.2 父子关系 vs 世界坐标

节点**父子关系**就是字面意义的树:父节点挪了,子节点跟着挪;父节点缩了,子节点也缩;父节点删了,子节点跟着删。

- **Local Position**(本地位置):子节点**相对父节点**的位置
- **Global Position**(全局位置):子节点**相对世界原点**的位置

Inspector 里**最右边那把锁 + Transform 标签**可以切 Local / Global。

> **小贴士**:做 3D 第一天你大概不用纠结这个,先把"父移动子跟着"当肌肉记忆即可。等做角色系统(角色头节点下挂相机),Local Position 才会显出威力。

## 五、概念二:Scene —— "一个 .tscn 文件 = 一棵节点树"

刚刚我们在 Scene 面板里堆了几个节点,但**它们只活在内存里**。要保存下来给别人用、给代码 `preload()`,必须**存成场景文件**。

### 5.1 第一次保存场景

1. 在 Scene 面板**选中你希望当"根"的节点**(通常是 `Node3D`)
2. 顶部菜单 `Scene → Save Scene As`(或 `Ctrl + S` / `Cmd + S`)
3. 文件名输 `main.tscn`(Godot 场景文件后缀)
4. 保存到项目根目录 → FileSystem 面板出现新文件

> **小贴士**:`.tscn` 是**文本格式**的场景文件,可以直接用记事本/VSCode 打开,你能看到里面的节点树、属性值、引用关系。这点比 Unity 的 `.unity` 二进制 YAML 友好得多,**出 bug 时直接 `cat main.tscn` 就能看场景到底有啥**。

### 5.2 一个 .tscn 文件长啥样

保存后打开 `main.tscn`,你会看到类似这样的内容(我们的实战 1 产物):

```ini
[gd_scene load_steps=5 format=3 uid="uid://b7g3rootmain1"]

[sub_resource type="ProceduralSkyMaterial" id="ProceduralSkyMaterial_1"]
sky_horizon_color = Color(0.646, 0.655, 0.667, 1)
ground_horizon_color = Color(0.646, 0.655, 0.667, 1)

[sub_resource type="Sky" id="Sky_1"]
sky_material = SubResource("ProceduralSkyMaterial_1")

[sub_resource type="Environment" id="Env_1"]
background_mode = 2
sky = SubResource("Sky_1")

[sub_resource type="PlaneMesh" id="PlaneMesh_1"]
size = Vector2(20, 20)

[node name="Main" type="Node3D"]

[node name="WorldEnvironment" type="WorldEnvironment" parent="."]
environment = SubResource("Env_1")

[node name="Sun" type="DirectionalLight3D" parent="."]
transform = Transform3D(0.866, -0.25, 0.433, 0, 0.866, 0.5, -0.5, -0.433, 0.75, 0, 8, 0)
shadow_enabled = true

[node name="Camera3D" type="Camera3D" parent="."]
transform = Transform3D(1, 0, 0, 0, 0.866, 0.5, 0, -0.5, 0.866, 0, 6, 10)

[node name="Ground" type="MeshInstance3D" parent="."]
mesh = SubResource("PlaneMesh_1")
```

> **为什么要这样做**:Godot 把场景存成**纯文本**,意思是:
> 1. **Git diff 友好**——你能看到"老张加了哪个节点,删了哪行"
> 2. **出错时手修可能救你一命**——比如删了不该删的引用、Transform 数值莫名变 NaN
> 3. **学习时打开看比看文档更直观**——文件就是真相

## 六、概念三:Instancing —— "场景 = 模板,实例 = 复制品"

**这是 Godot 最值钱的一个概念,也是和 Blender 思维最大的不同**。

Blender 里:想复制 20 个箱子?`Shift + D` 复制 20 次,改 20 次 Transform。
Godot 里:**做 1 个箱子场景(`crate.tscn`),然后实例化 20 次,改 20 次 Transform**。

听起来一样?关键差别:

| | Blender `Shift+D` 复制 | Godot 实例化 |
|---|---|---|
| 数据存在 | **同一个物体的 N 份拷贝** | **同一个 .tscn 模板的 N 份实例** |
| 改模板 | 不影响已复制的 | **所有实例跟着变** |
| 改属性 | 各改各的 | 可以在 Inspector 覆盖(Override),只改一个实例 |
| 性能 | 每个物体独立 mesh 数据 | 共享 mesh 数据,GPU 友好 |

> **小贴士**:这个"模板 → 实例"的关系,和**OOP 里 Class → Object** 一模一样。`crate.tscn` 是类,每个 `crate_instance` 是对象。**一旦接受这个心智模型,所有 Godot 复用逻辑都通了**。

### 6.1 两种实例化姿势

**姿势 A:编辑器里拖**
1. FileSystem 面板 → 找到 `crate.tscn`
2. **拖到 Scene 面板**(不是 Viewport),作为某个节点的子节点释放
3. 实例化完成!Inspector 里能看到 `crate.tscn` 路径

**姿势 B:代码里 instantiate()**
1. 新建脚本 `box_spawner.gd`,内容如下(整段 copy-paste 可用):
```gdscript
extends Node3D

@export var box_scene: PackedScene
@export var count: int = 20
@export var spacing: float = 2.0

func _ready() -> void:
    if box_scene == null:
        push_warning("请在 Inspector 里把 box_scene 字段拖入你的 crate.tscn")
        return
    var cols: int = 5
    for i in range(count):
        var instance: Node3D = box_scene.instantiate()
        var row: int = i / cols
        var col: int = i % cols
        instance.position = Vector3(col * spacing, 0.5, row * spacing)
        add_child(instance)
```
2. 把脚本**挂**到任意 `Node3D` 节点上(Inspector 底部 `Script` 字段 → 选这个 .gd)
3. Inspector 顶部出现 `Box Spawner` 分类,`Box Scene` 字段**拖入** `crate.tscn` 文件
4. 按 `F5` 跑起来,场景里 20 个箱子就位

> **小贴士**:`@export var box_scene: PackedScene` 是 Godot 4 引入的**类型化导出**——它让 Inspector 字段类型安全,你只能拖入场景文件,不能拖错。`@export` 关键字仅作用于本类的 `var`,但所有继承 Node 的类都能用。

> **坑点 2:`preload()` 报错 "Can't preload resource"?**
> `preload("res://crate.tscn")` 是**编译期**就尝试加载路径,如果路径错、文件不存在,Godot 启动就崩。**新手用 `@export var` 拖入更安全**(运行时才加载,Inspector 还能改)。更高级的懒加载写法是 `load("res://crate.tscn")`,运行时按需加载。

## 七、实战 1:做出 3D 根场景(能在视口里跑起来)

**目标**:一个能 `F5` 跑起来,看到天空、太阳、地面、20 个箱子的 3D 场景。**预计 15 分钟**。

### 7.1 新建项目 + 第一个场景

1. 启动 Godot 4.6 → New Project → 路径 `~/godot-projects/03-godot-intro`,Renderer 选 **Forward+**(4.x 默认就是这个,3D 项目不用动)
2. Project 打开后,自动进 3D 视口
3. 顶部菜单 `Scene → New Scene`(或 `Ctrl + N`)→ 选 `Node3D` → Create
4. 选中新建的 `Node3D`,Inspector 顶部 Name 字段**改成 `Main`**
5. `Ctrl + S` 保存为 `main.tscn`

### 7.2 加上"渲染环境"—— WorldEnvironment

> **为什么要这一步**:不告诉 Godot 天空长啥样、地板要不要反射,它默认给你一个**纯粉红色背景**(debug 占位色),丑得没法看。

1. Scene 面板选中 `Main` → 右键 `Add Child Node` → 搜 `WorldEnvironment` → Create
2. 选中 `WorldEnvironment` → Inspector 找到 `Environment` 属性(写着 `<empty>`)→ 点击 `▼` → `New Environment`
3. 选中刚生成的 `Environment` 子资源 → Inspector 顶部 `Background` 模式选 `Sky`
4. 下面 `Sky` 字段也点 `▼` → `New Sky` → 自动创建 `Sky` 子资源 + `ProceduralSkyMaterial`
5. 现在 3D 视口背景变成了**蓝天**

### 7.3 加上太阳

1. Scene 面板选中 `Main` → 加 `DirectionalLight3D` 节点
2. 选中它 → Inspector 把节点名字改成 `Sun`
3. 在 Viewport 里你会看到一个**白色圆圈** = 灯光图标,`Shift + 拖动` 旋转它的朝向,让光从斜上方照下来
4. Inspector → `Shadow` → `Enabled` 勾上,启用阴影

### 7.4 加上相机

1. Scene 面板 → 加 `Camera3D` 节点
2. 选中它 → 按 `F` 让视口对焦到相机位置
3. 工具栏上方找到 **"Preview"** 下拉框(写着 `Perspective`)→ 保持默认
4. 选中相机节点 → Inspector 找到 `Transform`,调整 Position 到 `(0, 6, 10)`,Rotation X 改成 `-30°`,相机现在斜俯视场景

### 7.5 加上地面

1. Scene 面板 → 加 `MeshInstance3D` 节点,改名为 `Ground`
2. Inspector → `Mesh` 字段 → 点 `▼` → `New PlaneMesh` → 默认就是 10×10 米的平面
3. 选中 `Ground` → Inspector → `Scale` 改成 `(2, 1, 2)`,地面扩大到 20×20 米

### 7.6 跑起来

按 **`F5`** → 弹窗问 "Create a new main scene?" → 选 `main.tscn` → OK
你应该看到一个**蓝天 + 阳光 + 斜俯视地面**的 3D 场景。这就是你的第一个 Godot 3D 场景!

> **坑点 3:跑起来后是黑屏?**
> 99% 的可能是**没有 Camera3D**,或者**Camera3D 在世界原点被地面挡住**。解决:回到编辑器 → 选中 Camera3D → 按 `F` 相机对焦 → 看 Transform Position Y 是不是 5+、Z 是不是 10+。按 `F5` 再跑一次。

## 八、实战 2:做一个"箱子场景"然后实例化 20 个

### 8.1 新建 crate 场景

1. 顶部菜单 `Scene → New Scene`(`Ctrl + N`)→ 选 `Node3D` → Create
2. 选中根节点,改名 `Crate`
3. Scene 面板 → 加 `MeshInstance3D` 节点,改名 `Mesh`
4. 选中 `Mesh` → Inspector → Mesh → New BoxMesh → 默认就是 1×1×1 立方体
5. 选中根节点 `Crate` → `Ctrl + S` → 文件名 `crate.tscn` → 保存

### 8.2 实例化到主场景

回到 `main.tscn`(顶部菜单 `File → Recent → main.tscn`):

1. Scene 面板选中 `Main`
2. FileSystem 面板 → 把 `crate.tscn` **拖到 Scene 面板的 `Main` 上**释放
3. 场景里出现一个箱子,位置 `(0, 0.5, 0)`(把 Crate 根节点 Position 设为 0.5,避免穿过地面)
4. Inspector 顶部能看到 `Instance: crate.tscn` 标识

### 8.3 用代码撒一片箱子

1. 顶部菜单 `FileSystem` 面板 → 右键 `res://` → `New Resource` → 选 `GDScript` → 文件名 `box_spawner.gd`
2. 双击打开它(默认进 Godot 内置 Script Editor),把第 6.1 节那段 `BoxSpawner.gd` 完整粘进去,`Ctrl + S` 保存
3. 回到 `main.tscn` → 选中 `Main` 节点 → Inspector 底部 `Script` 字段 → 选 `box_spawner.gd`
4. Inspector 顶部出现 `Box Spawner` 分类,`Box Scene` 字段拖入 `crate.tscn`,`Count` 设 20,`Spacing` 设 2
5. `F5` 跑起来 → **5×4 网格的 20 个箱子**整整齐齐摆好

> **为什么要这样做**:20 个箱子 = 同一个 `crate.tscn` 的 20 个实例。如果你想给所有箱子换皮,只改 `crate.tscn` 一处,20 个全跟着变。这就是 Godot 实例化的复利价值。

## 九、常见坑点速查表

| 坑 | 症状 | 解决 |
|---|---|---|
| 节点加了找不到 | 视口里啥也没出现 | 选中节点按 `F` 对焦;或者把 Position 改成 `(0, 0, 5)` 挪开 |
| 跑起来画面纯粉/纯黑 | 没 WorldEnvironment / 没 Camera3D | 加 `WorldEnvironment` + `Camera3D`,并选中 Camera3D 按 `F5` |
| `F5` 跑完无窗口 | 可能全屏被另一窗口挡住 | `Project Settings → Display → Window → Viewport Width/Height` 调小一点 |
| 拖文件到 Scene 面板不生效 | 拖到了 Viewport | 必须**拖到 Scene 面板**(节点树),不是中间视口 |
| 实例化 1000 个箱子 FPS 暴跌 | 用了 1000 个独立 mesh | 等下篇学 **MultiMeshInstance3D**(一次 draw call 画上千个) |
| `extends Node3D` 报错脚本没生效 | 节点类型和脚本不匹配 | 3D 节点必须 `extends Node3D` 或子类,不能 `extends Node2D` |
| Inspector 字段类型不对 | 拖入文件不识别 | `@export var xxx: 类型` 的类型注解要写对(如 `PackedScene`、`Texture2D`) |
| `@export` 字段编辑器里看不到 | 忘了 `Ctrl + S` 保存脚本 | Godot 会读脚本里**最新保存**的 `@export`,不保存等于没写 |

## 十、今天的小结

- **Node**:Godot 世界由树状节点组成,相机/网格/灯光/物理/UI 都是节点
- **Scene**:`.tscn` 文件 = 一棵节点树的**序列化**,可保存、可 preload、可 Git diff
- **Instancing**:场景是模板,实例是复制品,改模板所有实例跟着变
- **编辑器四区**:Scene / FileSystem / Inspector / Viewport
- **3D 根场景配方**:`Node3D` + `WorldEnvironment` + `DirectionalLight3D` + `Camera3D` + `MeshInstance3D`
- **代码实例化**:`@export var scene: PackedScene` + `scene.instantiate()` + `add_child()`

## 十一、明天预告

下一篇进入 **Godot 4.6 - 3D 基础**:把前两天在 Blender 做的**低多边形房子 `.glb` 导入 Godot**,用 `MeshInstance3D` 加载,给地面贴 PBR 材质,加上天空光照,**把这套"Blender 制作 → Godot 消费"流水线真正打通**。

## 十二、延伸阅读

1. **Godot 官方文档 - Nodes and Scenes**:<https://docs.godotengine.org/en/stable/tutorials/best_practices/node_alternatives.html> — 节点设计的取舍哲学,Why Node-based,推荐先读。
2. **Godot 官方文档 - Instancing**:<https://docs.godotengine.org/en/stable/tutorials/best_practices/instancing.html> — 实例化的所有姿势(编辑器拖、preload、load、_ready 里动态建),官方权威。
3. **KidsCanCode - Godot 4 入门系列**:<https://kidscancode.org/godot_recipes/4.x/g101/> — 国外最经典的 Godot 4 零基础视频/文字教程,从安装到 3D 第一人称都覆盖,**和我们的系列风格最契合**。

---

> **软件版本**:本文基于 Godot 4.6-stable(2026-01-26 发布)撰写。  
> **数据来源时间**:2026-06-14(参考 Godot 官方文档 4.6 分支、社区教程交叉验证)。
