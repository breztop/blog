---
title: "Blender & Godot 3D 教程 - 2026-06-17 - Blender 材质与纹理：PBR 节点编辑器 + UV 展开零基础入门"
date: 2026-06-17T04:00:00+08:00
description: "零基础打开 Blender 4.x 着色器编辑器：理解 PBR（Base Color / Metallic / Roughness）、给低多边形房子上色、UV 展开挂贴图、用程序化纹理做地面，导出后 Godot 直接用。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-17.png
  alt: "Blender 中低多边形物体附带 PBR 材质节点的特写"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-17.png)

> 系列第六篇。前两篇我们从「打开 Blender 看懂界面」到「挤出 / 循环边 / 镜像建出低多边形房子」。今天解决下一个灵魂问题：**模型灰扑扑一片白，怎么让它好看？** 答案就是 **材质（Material）+ 纹理（Texture）**。我们这一篇只做一件事：把 6.13 那个低多边形房子从「灰色塑料」变成「木头墙 + 红瓦顶 + 玻璃窗」，最后导出 `.glb` 直接丢进 Godot 用。

## 0. 本篇目标 & 前置

**学完你能做什么：**
- 打开 Blender 自带的 Shader Editor，看懂节点怎么连
- 给任何物体加 Principled BSDF 材质，调 Base Color / Roughness / Metallic
- 用 UV 编辑器给物体「拆封」展开贴图
- 加载一张图片纹理（墙纸、瓦片、法线贴图）贴上去
- 用 Noise Texture + ColorRamp 做程序化地面
- 导出 `.glb` 给 Godot 用

**前置：**
- Blender 4.x 已安装并能正常打开（参考 6.12）
- 你手头至少有一个简单模型（跟着 6.13 建一个房子，或者就用默认的 Cube / Monkey）
- 这一篇**不要求**任何美术基础

---

## 1. 三分钟理解 PBR：游戏 3D 物件为什么「真实」

PBR = **Physically Based Rendering**（基于物理的渲染）。听起来吓人，其实就是把「光怎么打在物体上」拆成 4 个旋钮，调一调就有 90% 的真实感。

### 1.1 四大核心参数（Blender 默认的 Principled BSDF 节点）

| 参数 | 做什么 | 常见物体 |
|---|---|---|
| **Base Color**（基础色） | 物体本身的颜色，木头就是棕、玻璃就是青 | 红瓦、棕墙 |
| **Roughness**（粗糙度） | 表面有多光滑。0 = 镜子，1 = 砂纸 | 镜面 0.05，砖墙 0.85 |
| **Metallic**（金属度） | 是不是金属。0 = 非金属（塑料/木头/皮肤），1 = 金属（铁/铜/金） | 铁门 1.0，皮肤 0.0 |
| **Normal / Bump**（凹凸） | 不改形状也能「假装」有凹凸细节（砖缝、划痕） | 砖墙表面的小坑 |

> 💡 **为什么要这样做**：游戏里如果用「传统」贴图（一张图搞定一切），换到不同光线下立刻穿帮——太阳光下亮、阴天就全黑。PBR 用这 4 个参数描述「物理上真实的材质」，无论在 Blender、Godot、Unreal 还是浏览器里渲染，**长得都差不多**。这是 3D 行业过去 10 年最重要的统一标准。

### 1.2 一句话心法

> **颜色靠 Base Color，质感靠 Roughness + Metallic，细节靠 Normal/Bump。**

接下来所有操作就是把这 4 个旋钮接到节点上。

---

## 2. 第一次给物体上色（30 秒版）

打开 Blender，跟着建一个 cube 当练习：

1. 启动 Blender → 默认场景里有个 Cube
2. 右下角属性面板找到 **红色小球图标**（Material Properties，点不到就用鼠标滚轮找，最右边一列图标）
3. 点 **New** → 名字改成 `Mat_Cube`
4. 默认连的就是 Principled BSDF
5. 点 **Base Color** 左边的小色块 → 选红色 → 视图里 Cube 立刻变红

![Material tab with red material applied](placeholder)

恭喜，你完成了人生第一份 PBR 材质。

> 💡 **为什么要这样做**：Material 是「配方」，Principled BSDF 是「最常用的标准配方」。所有游戏 / 电影 3D 物件**几乎都用**这个节点起步。

### 2.1 玩一下旋钮看效果

在视图右上角把 shading 模式改成 **Rendered**（按 Z 键在 Wireframe / Solid / Material Preview / Rendered 四种视图间切）：

- Roughness 拉到 **0.05** → 立方体变成镜子，反射 HDRI
- Roughness 拉到 **0.9** → 立方体变成磨砂塑料
- Metallic 拉到 **1.0** + Base Color 设金黄 → 黄金块
- Metallic 拉到 **0** + Base Color 设白灰 + Roughness 0.4 → 塑料玩具

切回 **Solid / Material Preview**（按 Z）继续编辑更省显卡。

---

## 3. 打开节点编辑器：Shader Editor

光靠旋钮能做 80% 的材质，但要加载图片、做程序化纹理，就必须进节点。

### 3.1 切到 Shader Editor

最上方菜单 **Editor Type** → **Shader Editor**。

![Editor type menu showing Shader Editor](placeholder)

你会看到：
- 左边一栏是节点列表（Add → 输入）
- 中间是当前选中物体的节点树
- 右边是「世界输出」（World Output）和「材质输出」（Material Output）

### 3.2 看懂节点连线

每条线都代表「数据从左流到右」：
- **节点** = 一个处理单元（取颜色、做运算、生成噪声）
- **输入 / 输出** = 数据插孔
- **Material Output** = 终点 → 决定表面看起来什么样

最常见结构：

```
[Image Texture]  ──Color──>  [Principled BSDF]  ──BSDF──>  [Material Output]
                          ^                        ^
                          |                        |
                  [Noise Texture]──>  [Bump]──>     Normal
```

把鼠标悬停在节点之间，能看到黄色高亮提示。**点住一个输出拖到另一个输入**就连上了；想断开就 **Ctrl + 右键拖**。

> 💡 **为什么要这样做**：节点式 = 图形化编程。比「写代码生成材质」快 10 倍，比「画一张巨大贴图」灵活 100 倍——你改一个节点参数，所有贴这个材质的物体实时变。

---

## 4. 加载一张图片纹理（用 Blender 自带的砖墙）

### 4.1 准备测试图

Blender 自带素材里就有一张砖墙：
1. 顶菜单 **Edit → Preferences → File Paths**
2. 看看 `Data` 那栏记下来（默认是 `/Users/<你>/.../Blender/4.x/datafiles/`）
3. 不同系统不一样，但有这张：`/datafiles/studiolights/world/internal_tree_background.exr` 之类……**不一定**有砖墙，我们直接用程序化纹理更稳（下一节）。

更省事的办法：**直接用 Noise Texture** 节点生成木纹 / 大理石 / 砖墙，不用找任何图片。

### 4.2 给 Cube 加程序化噪声贴图

回到 Shader Editor：

1. **Add → Texture → Noise Texture**
2. 默认会出来一个，参数：
   - **Scale** = 5（花纹大小，越大越密）
   - **Detail** = 8（细节层数）
   - **Roughness** = 0.7
3. Noise Texture 的 **Fac** 输出 → 拖到 **Principled BSDF** 的 **Base Color**

立刻看到 Cube 变成灰白大理石感。

### 4.3 调颜色：加 ColorRamp

光黑白太单调，加颜色：

1. **Add → Converter → ColorRamp**
2. ColorRamp 默认两个色标（左黑右白），点左下角 **+** 加到 3 个色标
3. 三个色标颜色分别设：**深棕 → 赭石色 → 浅米色**（木头色调）
4. Noise Texture 的 Fac → ColorRamp 的 Fac → ColorRamp 的 Color → Principled BSDF 的 Base Color

![ColorRamp connected to Noise Texture](placeholder)

搞定！立方体现在是木头色。

### 4.4 加法线凹凸：假装有木纹

1. **Add → Vector → Bump**
2. Noise Texture 的 Fac → Bump 的 Height
3. Bump 的 Normal → Principled BSDF 的 Normal
4. Bump 节点的 Strength 调到 **0.3**（太强会变成凹凸不平的月球表面）

切到 Material Preview 看效果——立方体表面有了细腻的磨砂感。

> 💡 **为什么要这样做**：没有法线贴图也能「假装」凹凸——Bump 节点通过扰动法线方向，让光照算出来「这里好像有凸起」。3D 游戏里 90% 的细节都是这么「骗」出来的，省贴图内存、省建模时间。

---

## 5. UV 展开：把 3D 表面摊平给 2D 贴图用

### 5.1 UV 是什么

把 3D 模型想象成一个「折纸盒子」。**UV 就是把每个面拆开、摊平到 2D 平面上**的过程。摊平后的 2D 图叫「UV 贴图」，对应到 2D 图片上就是贴图位置。

```
3D 立方体           UV 摊平后
+----+----+        +---+---+
|    |    |   →    |   |   |
+----+----+        +---+---+
|    |    |        |   |   |
+----+----+        +---+---+
```

不展开 UV，Image Texture 节点就不知道怎么把图片「贴」到哪里——可能整张图挤到一个角，也可能拉伸变形。

### 5.2 进 UV Editing 工作区

顶上工作区标签 **Layout** → 拉下来选 **UV Editing**。

你会看到：
- **左边** 3D 视图（Edit Mode，下面会教进）
- **右边** UV 编辑器（2D 平面）

### 5.3 标记接缝 + 展开

假设是 6.13 的低多边形房子：

1. 选中房子（Tab 进 Edit Mode → A 全选）
2. 选一条「屋顶和墙交接」的边（Alt + 左键）
3. 边菜单 **Ctrl + E** → **Mark Seam**（标记接缝）——变橙色
4. 屋顶四周 + 底部一圈都标记
5. **U** 键 → **Smart UV Project**（新手向）或 **Unwrap**（精细向）

右边 UV 编辑器立刻出现一堆小方块（UV Island）。恭喜，UV 展开了。

### 5.4 把 UV 当成「画布」加载贴图

1. 回到 Shader Editor
2. **Add → Texture → Image Texture**
3. **Open** → 加载任意一张图片（比如你想用砖墙图，去 https://ambientcg.com 免费下载 `BrickWall_2K`）
4. Image Texture 的 Color → Principled BSDF 的 Base Color

现在房子每个面都按 UV 位置贴上了砖墙图。

> 💡 **为什么要这样做**：UV 展开是「建模 → 上贴图」之间最关键的桥梁。UV 排布越均匀、岛之间留缝越合理，最终贴图越不拉伸、越清晰。Blender 的 Smart UV Project 是新手 80% 场景的最佳默认选择，不用死磕手动 Unwrap。

---

## 6. 实战：给 6.13 的房子做完整材质（复制这套配置）

打开 6.13 那篇做的低多边形房子（或者跟着建一个新的），按下面给每个部位分材质：

### 6.1 墙体（棕色木头）

1. 选中所有墙面 → Material Properties → 点 **+ New** → 命名 `Mat_Wall`
2. 节点：Noise + ColorRamp（深棕→浅棕）+ Bump
3. ColorRamp 三个色：R 0.25 / 0.55 / 0.78，G 0.15 / 0.35 / 0.55，B 0.05 / 0.20 / 0.35
4. Roughness **0.85**

### 6.2 屋顶（红色瓦片）

1. 选中屋顶所有面 → Material Properties → **+** → **New** → 命名 `Mat_Roof`
2. 直接 Base Color 设纯红 `#C52A2A`
3. Roughness **0.45**（瓦片微反光）
4. Metallic **0**

### 6.3 窗户（玻璃）

1. 选中窗户面 → **+ New** → `Mat_Glass`
2. Base Color 浅青 `#A8DADC`
3. Roughness **0.05**
4. Metallic **0**
5. Alpha **0.5**（半透明）+ Material Settings 里的 **Blend Mode → Alpha Hashed**

### 6.4 地面（程序化草地）

新建一个大 Plane → 命名 `Mat_Grass`：

```
[Noise Texture] (Scale 15, Detail 6) ──┬─Fac─→ [ColorRamp] ──Color──→ [Principled BSDF.Base Color]
                                          │
                                          └─Fac─→ [Bump Strength 0.5] ──Normal──→ [Principled BSDF.Normal]
```

ColorRamp：深绿 `#2D4A2B` → 中绿 `#4F7942` → 浅绿 `#90B77D`

切到 Rendered 视图看效果——房子和草地已经像那么回事了。

---

## 7. 导出给 Godot 用

材质做完，得让 Godot 看得懂。

### 7.1 推荐 `.glb`（glTF 2.0）

1. **File → Export → glTF 2.0 (.glb)**
2. 左边勾上 **Materials → Export**
3. 右边 Format 选 **glb**（二进制单文件，方便）
4. 命名 `house.glb` → Export

`.glb` 把模型 + UV + 材质 + 贴图路径全打包，Godot 4.x 直接拖进场景树就能用。

### 7.2 Godot 端导入

1. 把 `house.glb` 拖到 Godot 项目 `res://models/` 目录
2. 双击打开 → 顶部勾 **Mesh → Generate Tangents**（让法线贴图生效）
3. 拖进 3D 场景，立刻看到完整材质

> 💡 **为什么要这样做**：`.glb` 是 3D 行业的「PDF」——通用、稳定、Godot / Unreal / Three.js / Web 全都支持。Blender 原生 `.blend` 文件 Godot 不认，必须先导出。

---

## 8. 常见坑点 + 解决方案

### 坑 1：Image Texture 加载了但材质没变化

**症状**：图片在节点编辑器里正常显示，连了 Base Color 但物体没变。

**排查**：
1. 切到 Material Preview 视图（不是 Solid！）
2. 确认物体在 **Edit Mode 下 UV 已经展开了**（没 UV 的话贴图会缩到一点）
3. 检查 Image Texture 节点是否设了 Color Space = **sRGB**（Base Color 必须 sRGB；Normal 必须 Linear）

### 坑 2：贴图拉伸变形

**症状**：砖墙图在某个面上被拉成长条。

**解决**：回 UV 编辑器，选中变形的 UV 岛 → **S** 缩放 + **R** 旋转让它在 0-1 UV 空间内分布更均匀。**Smart UV Project 的 Island Margin 调到 0.02** 以上避免接缝穿插。

### 坑 3：Rendered 视图黑屏 / 颜色不对

**症状**：Solid 视图正常，切到 Rendered 整个物体变黑。

**排查**：
1. 右上角 Scene 光源是否够（没有灯 + 没有 HDRI = 漆黑）
2. **World Properties → Surface → Color** 改浅灰试试
3. Roughness 太低 + 没有 HDRI = 镜面反射了黑色 = 看着全黑
4. **Viewport Shading → Scene World / Scene Lights** 至少勾一个

### 坑 4（附赠）：法线贴图方向反了

**症状**：凹凸看起来「凹」的方向反了（应该凸的反而凹）。

**解决**：在 Image Texture 节点上点 **Color Space → Linear**（ColorRamp 之类的非数据贴图才用 sRGB），法线贴图永远是 Linear。

---

## 9. 30 天扩展练习

按顺序挑感兴趣的做，每天 30 分钟足够：

1. **Day 1**：把 6.13 的房子整套材质做完，导出 `.glb`，丢进 Godot 跑起来
2. **Day 2**：用免费素材（https://ambientcg.com 下载 `Wood039`、`BrickWall_2K`、`Ground037`）替换程序化节点
3. **Day 3**：学 Emissive（自发光）让窗户晚上亮起来
4. **Day 4**：学 Subdivision Surface 修改器给房子加圆角
5. **Day 5**：把材质做成 `.blend` 资产库，下次新项目直接 Append 复用

---

## 10. 延伸阅读

- 📖 [Blender Manual: Materials](https://docs.blender.org/manual/en/latest/render/materials/index.html) — 官方权威，覆盖每个节点
- 📖 [Blender Manual: UV Mapping](https://docs.blender.org/manual/en/latest/modeling/meshes/uv/index.html) — UV 编辑器所有功能详解
- 🎓 [CG Geek 的 PBR 材质入门 YouTube 系列](https://www.youtube.com/@CGGeek) — 英文但画面直观
- 🎨 [ambientCG](https://ambientcg.com/) — 1000+ 张 CC0 免费 PBR 贴图（颜色 / 法线 / 粗糙度），游戏开发起步期必收藏

---

## 版本与数据来源

- **软件版本**：Blender 4.3 LTS（实测命令兼容 4.0+），Godot 4.6
- **数据来源**：Blender 4.x Manual、Godot 4.x Docs、ambientCG 公开素材库
- **撰写时间**：2026-06-17 04:00 (Asia/Shanghai)
- **配套**：本系列前 5 篇见同目录下 `blender-godot-2026-06-12.md` 至 `blender-godot-2026-06-16.md`

> 下一篇预告：Blender **灯光与渲染**（HDRI / 三点光 / Cycles vs Eevee）——让模型在 Godot 里白天 / 夜晚都好看。
