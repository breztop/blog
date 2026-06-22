---
title: "Blender & Godot 3D 教程 - 2026-06-23 - Blender 几何节点零基础入门：程序化造一座'无限房子'工厂"
date: 2026-06-23T04:00:00+08:00
description: "零基础打开 Blender 4.x 几何节点(Geometry Nodes):理解程序化建模的'节点拼装'思维,做出一个'给我一块地就自动长出一片低多边形房子村'的小工厂,并导出 glTF 给 Godot 直接用。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-23.png
  alt: "几何节点程序化生成的低多边形城市夜景,深蓝背景,蓝色数据流"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-23.png)

> 系列第十一篇。6-13 我们徒手挤出过一个低多边形房子——挤一面墙、抠一扇门、做一片屋顶,**全部是手动**,一栋就要 10 分钟。  
> 今天用 Blender 4.x 的**几何节点(Geometry Nodes)** 重做这件事:**画一个网格平面,挂几个节点,一秒钟"长"出 100 栋不同形状的房子**。  
> 这一篇只解决一个问题:**怎么用"节点拼乐高"的方式,让电脑帮你建模**。学完你不仅能批量造房子,还掌握了"程序化建模"这个在游戏圈/影视圈越来越核心的思维方式,后面 6-29+ 的"综合实战"模式也会大量用到它。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 理解几何节点的"输入 → 处理 → 输出"三段式思维
- 打开 Blender 4.x 的 Geometry Nodes 编辑器,认识 5 个最常用节点(`Mesh Grid` / `Extrude Mesh` / `Instance on Points` / `Random Value` / `Join Geometry`)
- 用 8 个节点拼出一个"**平地放随机大小房子**"的小场景
- 进阶版:把 6-13 的房子做成 **Collection 资产**,让节点去"种"它而不是默认 Cube
- 导出 glTF,扔进 Godot 4.6 看到一片低多边形村庄
- 理解几何节点和"手动建模"的边界——**不是所有东西都该用节点**

**前置:**

- Blender 4.x 已装好(6-12 讲过界面)
- 不要求会 Python、不要求懂着色器代码
- **不要求**看过 6-13 的房子教程(没看过也能跟上;看过会更容易理解"为什么要程序化")

---

## 1. 为什么"手动建模"不够用？几何节点解决了什么痛点

先讲一个故事。

你做了一个开放世界游戏,美术让你做"100 栋中世纪房子"撒到地图上。手动做,即使每栋 5 分钟,**也要 8 小时不眨眼**。更惨的是——**关卡设计师改地图布局,房子位置全变,你得重摆 100 次**。

几何节点想说的是:**别摆了,我帮你算**。你只告诉 Blender 三件事:

1. 在哪里放(`Mesh Grid` 给一块地)
2. 放什么东西(`Collection Instance` 给一个房子资产)
3. 怎么变(`Random Value` 让每栋房子随机大小/旋转/位置)

剩下的它自己跑。**位置变了你改一行参数,100 栋房子全跟着重排**。这就是"程序化"的威力——**改参数,不动手**。

> 💡 **为什么要这样做**:游戏圈的"程序化"已经不只是"省时间"——它解决了**协作**和**可维护性**两个根本问题。关卡设计师、地形艺术家、shader 美术可以**用同一份节点图**,各自调自己的输入参数,不会动到别人的工作。这跟 Godot 里把场景拆成可复用的 `Node3D` 子场景,是一个道理。

### 1.1 一句话心法

> **几何节点 = 用"拼节点"的方式,描述"一堆模型应该长什么样、在哪、怎么变"。它不是替代手模,是替代"重复的手模"**。

---

## 2. 5 分钟认识 Geometry Nodes 编辑器

打开 Blender,场景里随便放一个 `Cube`(默认就有)。**选中它**,然后到顶部菜单栏点 **Geometry Nodes** 标签(在 Shading 标签旁边)。

界面瞬间变成这样:

| 区域 | 位置 | 作用 |
|---|---|---|
| 3D Viewport(左) | 整个左侧 | 实时看节点图运算结果 |
| Spreadsheet(右上) | 默认隐藏,按 `F8` 调出 | 看每个点/面的属性(进阶用) |
| Geometry Node Editor(右下) | 整个右半屏 | 你即将待的地方:拖节点、连线的画布 |

底部会自动出现一个 **"GeometryNodes" 修改器**(在 Modifier Properties 标签里也能看到)。这个修改器说:**"从现在开始,这个 Cube 的形状不再由我手模决定,由右边的节点图决定"**。

Node Editor 顶部有个空空的**节点画布**。你会看到最左边有两个**已存在**的节点:

- **`Group Input`**(左):Blender 喂给你的"原始几何"——也就是那个默认 Cube
- **`Group Output`**(右):节点图算完的结果,送回给修改器

中间是空的——**你接下来的工作就是在中间加节点、连线**。

> 💡 **为什么要这样做**:理解"输入 → 处理 → 输出"三段式,就掌握了**所有**节点编辑器(几何节点、材质节点、合成)的统一思维。这跟 GDScript 里"输入参数 → 业务逻辑 → 返回值"完全同构,先把这个心智模型立起来,后面学啥节点都快。

---

## 3. 第一个程序化房子:3 个节点搞定

我们做最简单的:**拿 Grid 铺一块地,在每个格点上挤出一栋随机高度的"火柴盒房子"**。

### 3.1 步骤 1:把输入从 Cube 换成 Grid(网格平面)

1. Node Editor 里**选中 `Group Input` 节点**
2. 按 `Shift + A` 调出添加菜单,搜 **`Mesh Grid`**(网格平面)
3. 选中,放进去,你会看到它出现在 `Group Input` 旁边
4. **`Group Input` 的 `Geometry` 输出 → 拖线到 `Mesh Grid` 的 `Mesh` 输入** ——这一下,Cube 就被替换成一张平面了
5. 选中 `Mesh Grid` 节点,看左下角小面板(按 `N` 调出,如果没开的话):
   - `Size X` = 10(横向 10 米)
   - `Size Y` = 10(纵向 10 米)
   - `Vertices X` = 10(横向 10 段)
   - `Vertices Y` = 10(纵向 10 段)

Viewport 里 Cube 消失了,变成一张 10×10 米的薄平面,**上面有 100 个格点**。

### 3.2 步骤 2:把每个格点"挤"出高度,变成火柴盒

1. 按 `Shift + A` → 搜 **`Extrude Mesh`**(挤出面)
2. `Mesh Grid` 的 `Mesh` → `Extrude Mesh` 的 `Mesh`
3. 选中 `Extrude Mesh` 节点,把 `Offset Scale` 改成 `1`(意思是"沿法线方向挤出")
4. 接下来我们要让"挤出多少"**随机**——按 `Shift + A` → **`Random Value`**
5. `Random Value` 节点的设置:
   - 类型留作 `Float`(浮点)
   - `Min` = 1.0
   - `Max` = 4.0
   - `Seed` = 42(随便,改它换一组随机值)
6. `Random Value` 的 `Value` 输出 → `Extrude Mesh` 的 `Offset Scale` 输入

Viewport 上,100 个格点瞬间"长高"了,但**每栋高度不一样**,高的 4 米矮的 1 米。

### 3.3 步骤 3:把"挤出后多余的顶面"盖成屋顶(可选但建议)

现在的房子顶是平的,没有 6-13 那种斜屋顶的味儿。要斜屋顶,有两条路:

- **A. 朴素路线**:在 `Extrude Mesh` 之后,再加一个 `Scale Elements` 节点,把 `Scale` 设为 `(0.6, 1.0, 0.6)`(顶面缩成 60%),就是简易金字塔屋顶
- **B. 进阶路线**:用 `Delete Geometry` 删顶面 + `Cone` 节点拼上去

我们走 A,简单直接:

1. `Shift + A` → `Scale Elements`
2. `Extrude Mesh` 的 `Mesh` → `Scale Elements` 的 `Mesh`
3. `Scale` 填 `(0.6, 1.0, 0.6)`
4. 选中 `Scale Elements`,把 `Selection` 输入**留空**(全部都缩)

Viewport 里,100 栋**带斜屋顶的火柴盒房子**站起来了。

### 3.4 把结果送回输出

最后,`Scale Elements` 的 `Mesh` 输出 → 拖线到 `Group Output` 节点的 `Geometry` 输入。

整张节点图长这样(从左到右):

```
[Group Input] → [Mesh Grid] → [Extrude Mesh] ← [Random Value]
                                     ↓
                              [Scale Elements]
                                     ↓
                              [Group Output]
```

> 💡 **为什么要这样做**:你可能注意到——**整个过程我们没碰过 mesh 的顶点、循环边、面**。这就是"程序化"的本质:**用参数和随机描述形状,而不是用鼠标描述每一个点**。当关卡设计师说"100 栋变 500 栋",你只需要把 `Vertices X/Y` 改成 `22 × 22`,啥都不用动。

---

## 4. 进阶:用 Collection 做"房子资产库"

第 3 步做的是"火柴盒",单调。我们让它"种"真正的 6-13 那种低多边形房子——**只改两个节点**。

### 4.1 准备房子资产

1. 打开 **一个新的 Blender 文件**(File → New,选 General)
2. 跟着 6-13 的步骤做一个简单的低多边形房子(没有就放个 `Cube` 凑数)
3. **关键:把房子移到 `Collection`**——在 Outliner(右上角大纲)里,你会看到默认的 `Collection` 集合,**右键房子 → Move to Collection → New Collection**,命名 `Houses`
4. **回到房子所在 Collection,选中房子,Ctrl+G 打个组**,命名为 `House_01`(或者就 `House`)
5. **文件 → 选 Append(追加) → 选你刚保存的 .blend → 选 `Collection` → 选 `Houses` → Link/Append**
   - **Append** = 复制一份到当前文件(适合你的资产库存在另一个文件里)
   - **Link** = 链接引用(改原文件会同步,适合多文件共享)

我们用 **Append**,简单。

> 💡 **为什么要这样做**:Append/Link 的区别跟"复制粘贴 vs 快捷方式"是一回事。Append 是"我手上有一份",改它不影响别人;Link 是"我手上有个引用",改它所有引用方都跟着变。生产环境多用 Link,但教学用 Append 不会把人搞晕。

### 4.2 把节点图改成"种房子"版本

回到主场景的 Node Editor,做 3 件事:

1. **删除** `Extrude Mesh` 节点(连同它后面的 `Scale Elements` 一起)
2. 按 `Shift + A` → **`Instance on Points`**(在点上生成实例)
3. `Mesh Grid` 的 `Mesh` → `Instance on Points` 的 `Points`(注意是 Points 不是 Mesh)
4. 选中 `Instance on Points` 节点,在左侧小面板:
   - `Instance` 输入先**留空**(下一步连)
5. 按 `Shift + A` → **`Collection Info`**(拿到一个 Collection 里的所有物体)
6. 选中 `Collection Info`:
   - `Collection` 下拉选 `Houses`
   - **勾选 `Separate Children`**(把 Collection 里的每个物体当作独立实例,而不是整体)
   - **勾选 `Pick Instance`**(让 Instance on Points 随机选一个)
7. `Collection Info` 的 `Instances` → `Instance on Points` 的 `Instance`
8. `Instance on Points` 的 `Instances` → `Group Output` 的 `Geometry`

**重新连一根随机旋转的线**(让房子朝向不同):

1. `Shift + A` → `Random Value` (类型 `Vector`)
2. 它的 `Value` → `Instance on Points` 的 `Rotation` 输入

整张图变成:

```
[Group Input] → [Mesh Grid] → [Instance on Points] ← [Collection Info] (Houses)
                          ↑                                ↑
                          └──── [Random Value (Vector)] ────┘
                                     ↓
                               [Group Output]
```

Viewport 上,**100 个位置,每栋位置随机旋转一个角度放了一个 House 资产**。

> 💡 **为什么要这样做**:用 `Collection Info` 而不是"挂房子本体"是因为——**Collection 可以装一整个零件库**。你做 10 种房子都丢进 `Houses` Collection,节点自动"随机抽一种"种下去,以后加新房子改 Collection 就够了,**节点图本身不用动**。这是程序化建模的可维护性核心。

---

## 5. 调参实战:5 个"小旋钮"让村庄活起来

到这一步,村庄"长得"已经能看了。**真正区分"程序化资产好 vs 不好"的是这 5 个旋钮**:

| 在哪儿调 | 改什么 | 视觉效果 |
|---|---|---|
| `Mesh Grid` 的 `Vertices X/Y` | 数值越大房子越密 | 10×10 变 5×5 = 稀疏村庄;20×20 = 密集城镇 |
| `Mesh Grid` 的 `Size X/Y` | 数值越大房子间距越远 | 5 米 = 紧凑社区;20 米 = 郊区独栋 |
| `Random Value` (Vector) 的 `Max` | 房子最大旋转角度 | 90° = 歪七扭八自然村庄;0° = 整齐划一军营 |
| `Collection Info` 里 Collection 的内容 | 增删房子资产 | 加 1 栋 → 自动出现新样式;删 1 栋 → 自动消失 |
| `Instance on Points` 的 `Scale` 输入 | 全部房子的统一缩放 | 1.0 = 原大小;2.0 = 巨人村;0.5 = 拇指村 |

**实验顺序建议**:先动 `Vertices X/Y` 体会"密/疏",再动 Random 的 `Max` 体会"乱/齐",最后改 Collection 体会"换资产",**这 3 个实验做完,你就掌握了程序化建模 80% 的日常调参**。

---

## 6. 导出到 Godot 4:glTF 一把梭

程序化出来的"100 栋房子"在 Blender 里看着热闹,但 Godot 引擎**只认 mesh 数据,不认节点图**。所以最后一步是**烘焙**(Bake)成普通 mesh。

### 6.1 在 Blender 里烘焙

1. 选中挂了 `GeometryNodes` 修改器的那个对象(我们的 Grid 平面)
2. 顶部菜单 **`Object` → `Convert` → `Mesh`**
3. 弹窗提示会"应用修改器"——**点 Apply**
4. 节点图就消失了,变成了一个**实实在在的 mesh**——你可以在 Edit Mode 里看,真的有 100 栋房子的面/顶点
5. 这下**你做过的所有程序化操作都"固化"成了几何体**,可以导出

### 6.2 导出 glTF

1. **`File` → `Export` → `glTF 2.0 (.glb / .gltf)`**
2. 弹窗里关键设置:
   - `Format` 选 `glTF Separate`(.gltf + .bin + textures 分开,Godot 喜欢)
   - **`Compression` 勾上**(压一下,小一点)
   - `Material` 选 `Export`(材质保留)
3. 选个路径,文件名 `low_poly_village.gltf`,Export

### 6.3 在 Godot 4.6 里加载

1. Godot 新建 3D 项目
2. **把 `.gltf` 和 `.bin` 文件直接拖进 Godot 的 FileSystem 面板**(左下角)
3. 双击 `.gltf`,你会看到预览图
4. **新建一个 `Node3D` 根节点的场景**,加一个 `Camera3D`(位置 `(0, 5, 10)` 旋转下),加一个 `DirectionalLight3D`
5. **从 FileSystem 拖 `.gltf` 进 3D 场景**——100 栋房子站起来了
6. F5 运行,鼠标右键拖可以看,滚轮缩放

整套 5-6 步,Blender 里的程序化村庄就跑进 Godot 了。

```gdscript
# main.gd - 可选:加个相机环绕脚本
extends Node3D

@onready var camera: Camera3D = $Camera3D
@onready var pivot: Node3D = $Pivot
var angle: float = 0.0

func _process(delta: float) -> void:
    angle += delta * 0.2
    pivot.rotation.y = angle
    # 镜头永远看着村庄中心
    camera.look_at(Vector3.ZERO, Vector3.UP)
```

把 `Camera3D` 拖进一个新建的 `Pivot (Node3D)` 节点下,挂这个脚本,F5 跑起来就是**镜头绕村庄转一圈**。

> 💡 **为什么要这样做**:Blender 的几何节点图是**创作工具**,Godot 是**运行引擎**。两者"脑回路"不同——Blender 关心"这堆东西怎么长",Godot 关心"运行时一帧算多少三角形"。所以一定要 `Convert → Mesh` 烘焙,把"过程"变成"结果",Godot 才接得住。

---

## 7. 三个新手必踩的坑

### 坑 1:改了节点图,Viewport 没反应

**症状**:连了线、调了参数,Viewport 里啥都没变。

**原因**:90% 是你**没连到 `Group Output`**。节点图默认是"数据流",**没接输出 = 结果没送出去 = 啥也没发生**。

**解决**:`最后处理的节点的输出` 一定要拖线到 `Group Output` 的 `Geometry`。最稳的检测方法是——**`Group Output` 节点应该至少有一根线连进来,而不是孤零零的**。

### 坑 2:Instance on Points 之后想编辑单个房子,选不中

**症状**:点击一栋房子,进入 Edit Mode,结果**所有房子都进编辑了**。

**原因**:程序化出来的实例在 Blender 里**就是同一个 mesh 的多个引用**(类似 Godot 的 `MultiMeshInstance3D`),不是独立的 100 个 mesh。

**解决**:这是"程序化的代价"——**你不能在节点图外面单独改某一栋**。要么 `Object → Convert → Mesh` 烘焙后单独改(但失去了"批量改参数"的便利),要么**改回手动模式**给那几栋"特别的房子"做单独资产,扔进 Collection 让节点去"种"。

### 坑 3:导出 glTF 后 Godot 加载报错/材质全黑

**症状**:glTF 拖进 Godot,弹个红字错误,或者房子是黑乎乎一片。

**原因**:最常见 2 个:
1. 用了 **Linked Collection** 而**没在导出前 Apply**,Godot 找不到引用的源文件
2. 用了**非 PBR 材质**(比如 6-17 之前的纯色材质),glTF 不带完整信息

**解决**:
- 导出前 **`Object → Make Local` → `Object → Convert → Mesh`** 一次,确保**所有引用都已固化**
- 材质严格用 **Principled BSDF**(PBR),用 6-17 的方法做的
- 导出选项里 `Compression` 不要开 `Draco`,Godot 4.6 还没原生支持 Draco 压缩

---

## 8. 延伸阅读

1. **[Blender 官方手册:Geometry Nodes 章节](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/index.html)**  
   节点字典大全,每一个节点都有例子。**查节点用,别从前往后读,会睡着**。
2. **[Blender Studio:Geometry Nodes 入门系列视频](https://studio.blender.org/films/)**  
   官方出品的免费教学系列,讲得比官方文档生动 10 倍,有中文字幕。
3. **[Godot 4 官方文档:glTF 导入](https://docs.godotengine.org/en/stable/tutorials/io/gltf/importer.html)**  
   glTF 在 Godot 里的"完整说明书"——材质、动画、骨骼怎么映射,全在这。

---

## 9. 今天的小结

| 你学到了什么 | 怎么用 |
|---|---|
| 几何节点 = 节点图 + 输入输出 | "程序化建模"的统一思维,材质/合成节点同构 |
| `Mesh Grid` + `Extrude Mesh` | 批量生成"火柴盒城市",调两个参数就够 |
| `Collection Info` + `Instance on Points` | 用"资产库"代替"手模房子",可维护性翻倍 |
| `Object → Convert → Mesh` | 烘焙节点图为真 mesh,Godot 才接得住 |
| glTF 导出 + Godot 拖入 | 5 步让 100 栋程序化房子进游戏 |

---

## 10. 软件版本 & 数据来源

- **Blender**:4.3 LTS(2026 年 6 月主流稳定版,Geometry Nodes 在 4.x 各版本接口一致)
- **Godot**:4.6(2026 年 6 月最新稳定版,glTF 导入器无变化)
- **截图/数据时间**:2026-06-22(UTC)
- **参考资料**:
  - Blender 4.3 LTS 官方手册
  - Godot 4.6 官方 glTF 文档
  - Blender Studio 公开教学系列

---

> **下一篇预告(6-24)**:几何节点讲完,我们停一停 Blender,回到 **Godot 4 GDScript 基础**——变量类型、信号、生命周期、`@onready` 这些"看起来琐碎但每行代码都在用"的小细节。**会用 GDScript 看代码,和真正懂 GDScript 是两回事**——下一篇我们用 3 个反直觉例子,把这层"懂"立起来。
