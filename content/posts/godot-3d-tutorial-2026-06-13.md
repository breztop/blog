---
title: "Godot 3D 完全指南 2026-06-13 - 从节点、材质到 PBR 渲染的 14 个核心概念"
date: 2026-06-13T00:30:00+08:00
description: "一份面向独立开发者的 Godot 4.x 3D 系统全景教程:从坐标系、节点树、网格、材质、光照、物理到性能优化,带你把'Hello Cube'做出'可以发布'的 3D 场景。"
tags: [godot, 3d, tutorial, game-dev, blender]
categories: [game-dev]
draft: true
cover:
  image: /images/posts/godot-3d-tutorial-2026-06-13.png
  alt: "Godot 4.x 3D 场景编辑器 + 节点树视图"
  relative: false
---

![cover](/images/posts/godot-3d-tutorial-2026-06-13.png)

> 这是「**Blender & Godot 3D 每日教程**」系列的第 2 篇。上一篇我们在 Blender 里认识了五区界面、做了第一次建模导出,本篇正式进入 **Godot 4.x 的 3D 世界**。如果你已经能熟练地保存 `.blend` 文件和导出 `.glb`,那么你完全可以跟上本篇的节奏。
>
> **目标读者**:写过一两个 Godot 2D 小游戏,想系统学 3D 的开发者。或者,Unity 3D 开发者转 Godot,需要一份"概念地图"。
>
> **预计阅读 + 实操时长**:3-4 小时(章节内带可运行的代码片段,建议每个 `### 小项目` 都动手做一遍)。

## 目录

1. 为什么 Godot 适合做 3D 游戏
2. 坐标系、Transform、场景树 —— 3D 世界的"语法"
3. 3D 节点全景:Node3D 家族的 12 个核心类
4. 第一个 3D 场景:Hello Cube
5. 网格与资源:从 `.glb` 导入到运行时实例化
6. 材质系统:StandardMaterial3D 与 PBR 四大贴图
7. 相机与视口:FOV、near/far、第一人称 / 第三人称切换
8. 光照:Directional / Omni / Spot 与阴影优化
9. 环境与天空:Sky、Fog、SSAO、SSR、SDFGI
10. 物理:StaticBody / RigidBody / CharacterBody 全家福
11. 输入与角色控制:Input Map + CharacterBody3D
12. 动画:AnimationPlayer、AnimationTree、状态机
13. 性能优化:MultiMesh、LOD、OcclusionCulling、Rendering Scalability
14. 实战项目:从零做一个可发布的 3D Demo 场景

---

## 一、为什么 Godot 适合做 3D 游戏

我们这个系列的最终目标是「在 Godot 4 里做一个可以联机的 3D 小游戏」。在动手写代码之前,先回答一个很现实的问题:**为什么选 Godot,而不是 Unity / Unreal / 自研引擎?**

### 1.1 三家主流引擎的对比

| 维度 | Godot 4.x | Unity 6 | Unreal 5.4 |
|---|---|---|---|
| 许可证 | MIT(完全开源、零分成) | 商业(年营收超 20 万美元需订阅) | 商业(5% 营收分成) |
| 渲染后端 | Vulkan(主)、OpenGL 兼容、Forward+ 渲染器 | URP / HDRP | Lumen / Nanite |
| 包体积(空场景) | ~15-30 MB | ~50-80 MB | ~200+ MB |
| 主语言 | GDScript(类 Python)、C#、C++ | C# | C++ / Blueprints |
| 学习曲线 | 低(GDScript 几小时上手) | 中 | 高 |
| 3D 生态 | 上升期(4.x 大改) | 成熟 | 顶级 |
| 与 Blender 协作 | 一流(`glb` 导入即用) | 一流 | 一流(FBX 链路) |

> **小贴士**:Godot 4.0 是 2023 年发布的重大版本,**几乎重写了整个 3D 渲染器**(从 OpenGL ES 3 切到 Vulkan)。如果你之前看过 Godot 3.x 的 3D 教程,**很多 API 名字都变了**,认准 4.x。

### 1.2 Godot 4.x 3D 的真实水平

不夸大也不贬低:**Godot 4.x 的 3D 能力已经能做到"独立游戏 + 中小商业项目"的级别**。它不是 AAA 引擎,但对 90% 的 3D 需求是够的:

- **够用**:第一人称 / 第三人称射击、解谜、平台跳跃、开放世界(Minecraft 类)、赛车、模拟经营、3D 塔防
- **勉强够用**:大世界开放世界(>10km² 视距,需要自己优化 LOD)、多人 MMO(网络层够,但生态工具少)
- **不擅长**:电影级写实渲染(用 Unreal)、超大场景流式加载(Unity DOTS 更成熟)

### 1.3 我们的学习路径

从这一篇开始,我们会按这个顺序学:

1. **基础**:节点 / 场景 / 坐标系(本篇第 2-4 章)
2. **资源**:网格 / 材质 / 纹理(5-6 章)
3. **视觉**:相机 / 光照 / 环境(7-9 章)
4. **交互**:物理 / 输入 / 角色(10-11 章)
5. **动态**:动画 / 粒子 / 特效(12 章,本篇略)
6. **性能**:优化与调试(13 章,本篇略)
7. **整合**:实战项目(14 章,本篇略)

> 这一篇涵盖**第 1-5 部分的基础**;第 6-7 部分会放在下一篇;第 8 部分是 1-2 周后的综合实战。

---

## 二、坐标系、Transform、场景树 —— 3D 世界的"语法"

如果你已经写过 2D Godot,这一章会很快。3D 比 2D 只多了**一个轴**和**更复杂的变换**。

### 2.1 Godot 用的是什么坐标系?

**Godot 4 用左手系、Y-up**(Unreal 也这么用)。

| 概念 | Godot 4 | Unity | Blender |
|---|---|---|---|
| X 轴 | 右 | 右 | 右 |
| Y 轴 | 上 | 上 | 后 |
| Z 轴 | 前(指向屏幕外) | 前 | 上 |
| 坐标系 | 左手 | 左手 | 右手 |
| 旋转 | 绕轴右手定则(从原点看过去顺时针为正) | 左手 | 右手 |

> **重要**:Blender 是 Z-up、右手系!从 Blender 导入模型到 Godot 时,**Godot 会自动转换**(`glb` 自带此信息),所以你看到场景里的模型是"立"着的。如果用 FBX,经常需要手动加 -90° 旋转调整,这就是个历史包袱。

**小项目**:在 Godot 里建一个新 3D 场景,看右上角**小坐标轴 Gizmo**——它会实时显示当前 Camera 看出去的方向,这是调试 3D 方向感的最好工具。

### 2.2 Node3D 三大变换

每个 `Node3D`(3D 节点基类)有三个独立的 transform:

```gdscript
# 位置(Position):节点原点在哪里
node.position = Vector3(1, 2, 3)  # (X=右1, Y=上2, Z=前3)

# 旋转(Rotation):节点朝向哪里
# Godot 4.x 推荐用四元数(Quaternion)或欧拉角
node.rotation = Vector3(0, deg_to_rad(45), 0)  # 绕 Y 轴转 45°
node.rotation_degrees = Vector3(0, 45, 0)      # 同上,度数版

# 缩放(Scale):节点多大
node.scale = Vector3(2, 2, 2)  # 均匀放大 2 倍
```

> **小贴士**:`position` / `rotation` / `scale` 是 **local**(相对于父节点)。要拿**世界坐标**,用 `global_position` / `global_rotation` / `global_transform`。
>
> 在 `_process` 或物理回调里,推荐读 `global_position` 写 `position`(大多数游戏逻辑用世界坐标更直观)。

### 2.3 Transform:把 3 个值打包成一个对象

`Transform3D` 是把 `position + rotation + scale` 打包成的对象,可以一次赋值,也可以做矩阵乘法:

```gdscript
# 构造一个 Transform3D
var t = Transform3D()
t.origin = Vector3(0, 5, 0)  # 位置
t.basis = Basis.from_euler(Vector3(0, deg_to_rad(30), 0))  # Y 轴 30°

# 一次赋值
node.transform = t

# Transform 乘法:把"父变换"作用到"子变换"上
# 等价于"在父节点坐标系下移动子节点"
var world_t = parent.global_transform * child.transform
node.global_transform = world_t
```

**小项目**:建一个 `Node3D`(叫 "SunSystem"),下面挂 3 个 `MeshInstance3D`(代表 "Sun" / "Earth" / "Moon")。在 `_process` 里,让 Earth 绕 Sun 转、让 Moon 绕 Earth 转。体会"父子节点 + transform 乘法"的力量。

### 2.4 场景树:3D 节点的容器

Godot 的场景都是**树形结构**的。3D 场景的根节点必须是 `Node3D`(或 `Node3D` 的子类),子节点可以是任意 `Node3D` 派生类:

```
Main (Node3D)
├── WorldEnvironment (环境与天空)
├── DirectionalLight3D (太阳光)
├── Camera3D (主相机)
├── Level (Node3D)  ← 关卡分组
│   ├── Floor (MeshInstance3D + StaticBody3D + CollisionShape3D)
│   ├── Wall_01 (MeshInstance3D + StaticBody3D + CollisionShape3D)
│   └── Pickup_Coin (MeshInstance3D + Area3D)
└── Player (CharacterBody3D)  ← 玩家
    ├── Body (MeshInstance3D)
    └── CameraRig (Node3D)
        └── Camera3D (玩家视角相机)
```

> **小贴士**:一个节点可以**同时挂多个组件**。比如 `Floor` 这个节点可能同时是:
> - `MeshInstance3D`(显示地板的网格)
> - `StaticBody3D`(物理:不动)
> - `CollisionShape3D`(碰撞形状)
>
> 物理和渲染在 Godot 里是**两个独立的组件**,可以自由组合。这点和 Unity 略有不同(Unity 的 MeshCollider 绑死在 Mesh 上)。

---

## 三、3D 节点全景:Node3D 家族的 12 个核心类

Godot 4 把所有 3D 功能拆成 12 个**核心节点类**。你不需要立刻全记住,但要有个全景图。

### 3.1 12 个核心类(按用途分组)

**位置与变换**
1. `Node3D` — 所有 3D 节点的基类
2. `Marker3D` — 一个空节点,只用来标记位置(刷怪点、路径点)

**可见物体**
3. `MeshInstance3D` — 显示一个 3D 网格(球、立方体、外部模型)
4. `MultiMeshInstance3D` — 同一网格的成百上千份(草、子弹、粒子)
5. `GPUParticles3D` — GPU 粒子系统(火焰、烟雾)

**相机与光照**
6. `Camera3D` — 玩家看到的"眼睛"
7. `DirectionalLight3D` — 太阳光(无限远平行光)
8. `OmniLight3D` — 点光源(灯泡、火焰)
9. `SpotLight3D` — 聚光灯(手电筒、车灯)

**物理交互**
10. `StaticBody3D` — 不动的刚体(地板、墙)
11. `RigidBody3D` — 物理刚体(球、箱子、掉落的杂物)
12. `CharacterBody3D` — 玩家角色(自己控制移动,有内建碰撞解决)
13. `Area3D` — 触发器区域(检测"玩家进入这个区域")

**辅助 / 高级**(本篇不深入)
14. `NavigationRegion3D` / `NavigationAgent3D` — 寻路
15. `AudioStreamPlayer3D` — 3D 位置音效
16. `CSGShape3D` — 构造实体几何(用 Godot 自带"画"出 3D 形状)
17. `VisibleOnScreenNotifier3D` / `VisibleOnScreenEnabler3D` — 屏幕外优化
18. `LightmapGI` / `VoxelGI` / `SDFGI` — 全局光照方案

### 3.2 节点 vs 资源

Godot 把"东西"分成两类:

- **节点(Node)**:场景树里的"实物",有位置、有父子关系。运行时会动起来。
- **资源(Resource)**:文件,只读,放在文件系统里。比如 `Mesh` 资源(一个 `.obj`/`.glb` 文件)、`Material` 资源(一个 `.tres`/`.material` 文件)、`Texture2D` 资源(一张图片)。

节点里"挂"资源。比如 `MeshInstance3D` 节点的 `mesh` 属性指向一个 `Mesh` 资源;`StandardMaterial3D` 资源里挂 `albedo_texture` 指向一个 `Texture2D`。

> **小贴士**:`.tres` 是 Godot 自己的文本资源格式,等价于 Unity 的 `.asset`、Unreal 的 `.uasset`。它可读、可版本管理、便于代码生成。改材质时优先用 `.tres`。

### 3.3 节点选择的"快查表"

| 你想做 | 用这个节点 |
|---|---|
| 在世界里放一个看得见的方块/模型 | `MeshInstance3D` |
| 一大片草/树/同款物体 | `MultiMeshInstance3D` |
| 玩家走动 | `CharacterBody3D` |
| 箱子被撞飞 | `RigidBody3D` |
| 不动的墙 | `StaticBody3D` |
| 玩家踩到地板掉血 | `Area3D` |
| 玩家头顶的血条 | `Sprite3D`(贴图)或 `Label3D`(文字) |
| 玩家走过掉头 | `NavigationAgent3D` |
| 看到某物体的距离事件 | `VisibleOnScreenNotifier3D` |

---

## 四、第一个 3D 场景:Hello Cube

最快上手的方式:打开 Godot,跟着做。

### 4.1 创建场景

1. 打开 Godot 4.x → `Project` → `New Project`
2. 选一个空目录(别选有中文路径的,有些平台会抽风)
3. `Renderer` 选 `Forward+`(默认,4.x 主推)
4. `Mobile` 兼容性按需
5. 创建后,左上角 `+` 按钮 → 选 `Node3D` 作为根节点,改名 `Main`

### 4.2 加一个立方体

1. 选中 `Main` → 右上角 `+` → 搜 `MeshInstance3D` → 添加
2. 在 Inspector 面板找到 `Mesh` 属性 → 点 ` [empty] ` → `New BoxMesh`
3. 场景树里出现一个**白色立方体**,但视图是空的——因为我们没相机、没光

### 4.3 加相机和光

1. 选中 `Main` → `+` → `Camera3D` → 选中它,在 Inspector 里设 `Position` = `(0, 2, 5)`(往后 5 米、往上 2 米)
2. 选中 `Main` → `+` → `DirectionalLight3D` → 设 `Rotation` = `(-45, 30, 0)`(斜向下打光)

按 F5 运行——你应该看到**一个被斜光照亮的白色立方体**。这就是你的第一个 Godot 3D 场景。

> **小贴士**:`F5` 运行后,在编辑器顶部会出现"Remote" 树,显示**运行时**的节点状态(变量值、当前位置等),调试时用得非常多。

### 4.4 让立方体动起来

加个脚本让立方体旋转:

```gdscript
# main.gd(挂在 Main 节点)
extends Node3D

func _process(delta: float) -> void:
    # 让立方体绕 Y 轴每秒转 90°
    $MeshInstance3D.rotation.y += deg_to_rad(90) * delta
```

`$MeshInstance3D` 是路径简写,等价于 `$"MeshInstance3D"`。运行后,立方体开始旋转。

**小项目**:把 `rotation.y` 改成 `rotation.x` 看效果;再改成 `position.y += sin(Time.get_ticks_msec() * 0.001)` 让它上下漂浮。

### 4.5 加上"天空"

现在背景是灰的。选中 `Main` → `+` → `WorldEnvironment` → 在 Inspector 里 `Environment` 属性 → `New Environment` → `Background` → `Mode` 选 `Sky` → `Sky` → `New Sky` → 立刻看到蓝天。

**完整目录结构**:
```
Main (Node3D) [main.gd]
├── WorldEnvironment
├── DirectionalLight3D
├── Camera3D
└── MeshInstance3D (BoxMesh)
```

这就是 3D 场景的**最小可运行单元**——后面所有复杂场景都是这个模板的扩展。

---

## 五、网格与资源:从 `.glb` 导入到运行时实例化

"3D 模型"在 Godot 里叫**网格(Mesh)**。它有三种来源:内置 primitive、外部文件、运行时生成。

### 5.1 内置 5 个 Primitive

不用建模,Godot 内置 5 种基础几何:

| Primitive | 用途 | 性能 |
|---|---|---|
| `BoxMesh` | 立方体、墙、地板 | 12 三角形 |
| `SphereMesh` | 球、行星 | ~500 三角形 |
| `CylinderMesh` | 圆柱、树干 | ~80 三角形 |
| `CapsuleMesh` | 胶囊(角色身体) | ~300 三角形 |
| `PrismMesh` | 棱柱(三角形截面的柱) | 2 三角形 |
| `PlaneMesh` | 平面(地表) | 2 三角形 |
| `PointMesh` | 一个点(几乎用不到) | 0 |
| `RibbonTrailMesh` | 拖尾(用 `Trail3D` 时) | 动态 |
| `TextMesh` | 把文字当 3D 网格 | 动态 |

> **小贴士**:`SphereMesh` 默认分段是 48(高分辨率)。调试用可以改成 16,省 GPU。

### 5.2 从 Blender 导入 `.glb`

这是和上一篇 Blender 教程的接续点。

1. 在 Blender 里建模 → `File` → `Export` → `glTF 2.0 (.glb)`
2. 勾选 `Compression`(默认开)
3. **Format** 选 `glTF Separate (.glb)`,不是 `gltf + textures` 那种文件夹结构
4. 把 `.glb` 拖到 Godot 的 `FileSystem` 面板
5. Godot 会自动弹导入对话框。**几乎全用默认就行**,但有几个关键开关:

| 选项 | 推荐 | 说明 |
|---|---|---|
| **Meshes** | | |
| `Generate Tangents` | ✅(开了 PBR 才需要) | 切线 |
| `Generate LODs` | 看场景规模 | Godot 4.x 支持,自动生成 LOD |
| `Create Shadow Meshes` | ✅ | 阴影专用低模 |
| `Light Baking` | Static(关 Dynamic) | 烘焙光照贴图(本篇略) |
| **Materials** | | |
| `Import Materials` | ✅ | 导入 Blender 里设的材质 |
| `Use External Storage` | 看个人 | 材质存为 `.tres` 方便改 |
| **Animation** | | |
| `Import` | ✅(有动画就开) | 导入关键帧 |
| `Optimization` | ✅ | 移除冗余轨道 |
| **Skinning** | | |
| `Import` | ✅(有骨骼才开) | 蒙皮 |

> **小贴士**:`.glb` 是 Godot **官方推荐**的 3D 交换格式。它比 `.fbx` 简单(没有 Autodesk 那套权限问题),比 `.obj` 完整(支持材质、动画、骨骼),是 Godot / Blender 协作的最佳桥梁。
>
> **不要用 .fbx**:Godot 4.x 对 FBX 的支持是"能用",但**有坑**(坐标系、单位、动画轴向都可能出问题)。`glb` 走原生 glTF 2.0 规范,链路最稳。

### 5.3 实例化 vs 静态网格

一个 `.glb` 文件,有两种用法:

**静态资源**(每个实例一份内存)
```gdscript
# 拖 .glb 到场景,得到 MeshInstance3D
var tree = preload("res://scenes/tree.tscn").instantiate()
add_child(tree)
```

**Mesh 共享 + 多实例**(推荐,内存友好)
```gdscript
# 把 .glb 拆成 Mesh 资源
var tree_mesh: Mesh = preload("res://assets/tree.glb")  # .glb 本身就是 Mesh 资源

# 多个 MeshInstance3D 共享同一份 Mesh
for i in range(100):
    var mi = MeshInstance3D.new()
    mi.mesh = tree_mesh
    mi.position = Vector3(randf() * 100, 0, randf() * 100)
    add_child(mi)
```

100 棵树只占 1 份磁盘和 1 份 GPU 资源。

**小项目**:在场景里放 1000 个 `BoxMesh` 立方体,看 FPS。注意**单个 `MeshInstance3D` 不能塞多个实例**——只能用 `MultiMeshInstance3D`(下篇讲)。

### 5.4 运行时生成几何(高级)

有时候你不想建模,想代码"画"一个形状:

```gdscript
# 用 SurfaceTool 构建网格
var st = SurfaceTool.new()
st.begin(Mesh.PRIMITIVE_TRIANGLES)

# 画一个三角形
var verts = [Vector3(0,0,0), Vector3(1,0,0), Vector3(0,0,1)]
for v in verts:
    st.set_normal(Vector3.UP)
    st.add_vertex(v)

st.index()
var mesh = st.commit()  # Mesh 资源

# 挂到 MeshInstance3D
$MyMeshInstance3D.mesh = mesh
```

`SurfaceTool` / `ArrayMesh` / `PrimitiveMesh` 是 Godot 提供的"代码建模"工具。本系列不做深入,了解即可。

### 5.5 资源加载的 3 个时机

```gdscript
# 1. preload() —— 编译时加载(随脚本一起进内存,启动时)
const TREE = preload("res://assets/tree.glb")

# 2. load() —— 运行时加载(异步、慢)
var tree = await ResourceLoader.load("res://assets/big.glb")

# 3. instantiate() —— 实例化场景(只对 .tscn/.scn)
var tree = preload("res://scenes/tree.tscn").instantiate()
```

> **小贴士**:`preload` 在 GDScript 4.x 里有**更强的类型推断**——比 `load` 快,但路径必须是常量字符串。
>
> 资源管理原则:**能 preload 就 preload**(大资源除外),`load` 用在"按需加载"(比如切换关卡时才加载新地图)。

---

## 未完待续

(本文为分阶段连载,以下是剩余章节大纲,后续将分批发布)

### 即将发布的内容

- **第 6 章** - 材质系统:StandardMaterial3D 与 PBR 四大贴图(Albedo / ORM / Normal)
- **第 7 章** - 相机与视口:FOV、near/far、第一人称 / 第三人称切换
- **第 8 章** - 光照:Directional / Omni / Spot 与阴影优化
- **第 9 章** - 环境与天空:Sky、Fog、SSAO、SSR、SDFGI
- **第 10 章** - 物理:StaticBody / RigidBody / CharacterBody 全家福
- **第 11 章** - 输入与角色控制:Input Map + CharacterBody3D
- **第 12 章** - 动画:AnimationPlayer、AnimationTree、状态机
- **第 13 章** - 性能优化:MultiMesh、LOD、OcclusionCulling、Rendering Scalability
- **第 14 章** - 实战项目:从零做一个可发布的 3D Demo 场景

### 下一篇预告

下一篇我们进入 **Godot 3D 材质与 PBR 渲染**:用 StandardMaterial3D 的 4 个核心属性(Albedo / Metallic / Roughness / Normal),把一个白方块"磨"成**青苔石头**——这是任何 3D 游戏美术最常用的"起手式"。

### 延伸阅读(本篇)

1. **Godot 4 官方 3D 文档**:<https://docs.godotengine.org/en/stable/tutorials/3d/index.html> — 最权威的 3D 章节汇总。
2. **Godot 4 节点树速查图**:<https://docs.godotengine.org/en/stable/tutorials/scripting/ids.html> — 不确定该用哪个节点时,来这里查。
3. **glTF 2.0 规范(KhronosGroup)**:<https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html> — 想深度了解 `.glb` 怎么来的,看官方规范。
4. **Blender → Godot 工作流**:<https://docs.godotengine.org/en/stable/tutorials/io/index.html> — Godot 官方写的 Blender 协作最佳实践。

---

> **软件版本**:本文基于 Godot 4.3 撰写,4.x 各小版本操作基本一致。  
> **数据来源时间**:2026-06-13(参考官方文档、社区教程交叉验证)。
> **状态**:连载中,共 14 章,当前完成 1-5 章。  
> **预计完成时间**:本周末前发完第二篇(6-10 章)、下周内发完第三篇(11-14 章 + 实战)。

---

> 注:本文已 `git add`,待 breeze 自己决定 commit 时机。
