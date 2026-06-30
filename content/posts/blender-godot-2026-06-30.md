---
title: 'Blender & Godot 3D 教程 - 2026-06-30 - Blender 骨骼绑定与动画零基础入门:让一个低多边形小人「挥手动脚」,导出 glTF 进 Godot 4 跑起来'
date: 2026-06-30T04:00:00+08:00
description: '零基础打开 Blender 4.x 做骨骼绑定 (Rigging) 与关键帧动画:从零搭一个低多边形小人 → 加 Armature 骨架 → 自动绑权重 → Pose Mode 摆姿势 → K 帧做挥手动脚 → 用 glTF 导出 → 拖进 Godot 4.6 跑通带 AnimationPlayer 的 3D 角色。'
tags: [blender, godot, multiplayer, 3d, tutorial, rigging, animation, armature, weight-painting, gltf]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-30.png
  alt: 'Blender 视口里一个低多边形小人的骨架 Armature,八面体骨骼可见,网格表面有红蓝渐变权重色,深色专业 UI'
---

![cover](/images/posts/blender-godot-2026-06-30.png)

# Blender 骨骼绑定与动画零基础入门：让一个低多边形小人「挥手动脚」，导出 glTF 进 Godot 4 跑起来

> 本文写给"我能挤出 / 打灯 / 渲染，但角色一抬胳膊整片网格都飞了"的 Blender 初学者。

## 为什么要学骨骼绑定

如果你一路跟这个系列，已经在 Blender 里做过：

- 06-12 / 06-13：界面、挤出、循环边、镜像
- 06-17：PBR 材质 + UV 展开
- 06-19：HDRI + 三点光 + Cycles / Eevee
- 06-23：几何节点程序化建模
- 06-27：Collections / Outliner / 资产库

——但是你会发现所有模型都是**死的**。游戏角色不可能每帧重新摆顶点位置，我们需要一种"我动一根线，整片网格跟着变"的技术。这就是**骨骼绑定（Rigging）**和**骨骼动画（Skeletal Animation）**。

今天的目标：**用 40 分钟**，从零做一个会挥手、会抬腿的低多边形小人，导出 glTF，拖进 Godot 4.6 看到它在场景里**真的动起来**。

## 0. 你需要准备什么

| 工具 | 版本 | 说明 |
|------|------|------|
| Blender | 4.x (本文写于 4.2 LTS) | 建模 + 绑定 + 动画 |
| Godot | 4.x (本文写于 4.6) | 跑最终场景 |
| 一个能用的鼠标 | — | 中键拖动视图，右键选择 |

> **前置知识**：知道 Blender 怎么切换 Object / Edit / Pose 模式（06-12），会用 Extrude（06-13），知道材质节点基本概念（06-17）。不会也没关系，跟着点也能跑通。

---

## 第 1 步：用 30 秒搭一个低多边形"火柴人"

打开 Blender → 默认场景已经有 Cube / Camera / Light，全选 `A` → `X` → `Delete` 清空。

### 1.1 做躯干

1. `Shift + A` → `Mesh` → `Cylinder`（圆柱）
2. 立即按 `S` → `Z` → `0.5` 压扁成扁圆柱（身体）
3. `Tab` 进 Edit Mode → `Ctrl + R` 加一圈循环边 → 向下拖到大概脖子位置
4. 上面那段 `S` → `Z` → `0.6` 缩成脖子/头连接处

**为什么要这样做**：循环边之后顶面会被分两段，下一步挤出"头"才不会穿模。

### 1.2 挤出头

1. 顶面 `NumPad 7` 切到顶视图（如果你不喜欢切视图就保持 Perspective）
2. `Tab` 回 Edit Mode，`3` 切到面选择
3. 鼠标移到顶部那块面 → `L` 选中相连的面 → `E` 向上挤出 → `0.5` 单位
4. 再 `E` 挤出一次 → `S` → 整体放大 1.3 倍 → 这是头

### 1.3 做四肢

四肢用**两根细圆柱**就够了：

1. `Shift + A` → `Cylinder`，`S` → `X` → `0.15`，`S` → `Z` → `1.5` 拉长
2. `G` → `X` → `1.2` 移到右侧 → 这是右上臂
3. `Shift + D` 复制 → `G` → `Z` → `-1.2` → 下移 → 右上臂 → 右前臂
4. 同样复制到左侧：`S` → `X` → `-1` 镜像
5. 腿：再来两根长圆柱，放到身体下方

> **小贴士**：火柴人阶段先不要纠结好不好看。骨骼绑定的核心是"骨架对" + "权重对"，几何形状粗一点完全不影响学习。

最后 `File` → `Save As` → `stickman.blend`。

---

## 第 2 步：加 Armature（骨骼）

### 2.1 在身体中心加骨架

1. 鼠标点一下场景空白处（取消选择）
2. `Shift + A` → `Armature` → `Single Bone`
3. 你会看到一根白色八面体"骨头"出现在场景原点 → 这就是 Root Bone

### 2.2 进入 Edit Mode 编辑骨架

1. 选中那根骨头 → `Tab` 进 **Edit Mode**（注意：骨头也有 Edit Mode）
2. 视图切到前视图（`NumPad 1`），方便看 Y 轴方向
3. 选中最底端那个"尾巴"小八面体（叫 Tip，是关节） → `E` 挤出 → 向上 `0.5` 单位 → 第 2 根骨头：Spine（脊柱）
4. 继续 `E` 挤出到脖子 → Head（头骨）
5. 现在 Tip 在头顶

### 2.3 做肩膀和手臂

1. 选中肩膀位置那根骨头的 Tip → `E` → `X` 向右 0.6 → Shoulder.R
2. 选中 Shoulder.R 的 Tip → `E` → `X` 再 0.5 → UpperArm.R
3. 选中 UpperArm.R 的 Tip → `E` → `X` 再 0.5 → LowerArm.R
4. **复制到左边**：选中 Shoulder.R / UpperArm.R / LowerArm.R → `Shift + D` → `S` → `X` → `-1`（镜像到 -X 侧） → 记得改名 Shoulder.L / UpperArm.L / LowerArm.L

### 2.4 做髋部和腿

1. 选中 Root Bone 的 Tip → `E` → `Z` 向下 0.5 → Hip（髋骨，分叉点）
2. 选中 Hip 的 Tip → `E` → `X` 向右 0.4 → UpperLeg.R
3. 继续 `E` → 向下 0.7 → LowerLeg.R
4. `Shift + D` 镜像到左边

**完整骨架层级**应该长这样（在 Outliner 的 Armature 折叠里能看）：

```
Root
├── Spine
│   └── Head
├── Shoulder.R → UpperArm.R → LowerArm.R
├── Shoulder.L → UpperArm.L → LowerArm.L
└── Hip → UpperLeg.R → LowerLeg.R
        UpperLeg.L → LowerLeg.L
```

> **小贴士**：Blender 默认骨架是细长八面体，看不清时在 Armature Object Data Properties（绿色骨头图标）→ Viewport Display → **In Front** 打勾 + 勾上 **Display As** → Octahedral，骨架就会穿透网格显示。

---

## 第 3 步：把网格绑定到骨架（自动权重）

### 3.1 父子绑定

1. 鼠标框选**所有网格**（Body / Head / 4 条胳膊腿）
2. **加选骨架**（最后加选，方法是 `Shift + 右键点骨架`，它会变成橙色高亮轮廓）
3. `Ctrl + P` → 选 **Armature Deform` → `With Automatic Weights**

完成！你会在每个网格的 Object Data Properties（绿色三角形图标）→ **Vertex Groups** 里看到自动生成的组：`Spine` / `Head` / `UpperArm.R` ...

**为什么要选 With Automatic Weights**：Blender 会计算每个顶点离每根骨头多近，自动生成 0~1 的权重。这样你旋转骨头，附近的顶点就会跟着动。比手动设权重快 100 倍，适合入门。

> **常见坑 1：Ctrl+P 菜单里看不到 "With Automatic Weights"**
> 原因：你选中时**先选了骨架再选了网格**。父子绑定的规则是「**子先选，父最后**」。反过来选就会出现 `Make Parent without Inverse` 这类反过来的菜单。重新按顺序选一遍即可。

---

## 第 4 步：Pose Mode 摆姿势

### 4.1 进入 Pose Mode

1. 选中骨架（鼠标点骨架）→ `Ctrl + Tab` → 选 **Pose Mode**
2. 蓝色高亮的是骨骼，绿色的是当前选中

### 4.2 试一下能不能动

1. 选中 `UpperArm.R`（右上臂）→ `R` → 上下拖动鼠标
2. 整条右臂应该跟着旋转
3. 试试 `R` → `Z` → 输入 `45` 回车 → 转 45 度

如果某个部位完全不动（比如你选了 LowerArm.R，但整条手臂 + 躯干都跟着转），说明**父子关系断了**。回 Edit Mode 检查这根骨头是不是上一级的子级。

> **常见坑 2：旋转骨头时整片网格像被卷起来一样扭曲**
> 原因：自动权重失败 / Vertex Groups 数量为 0。打开网格的 Vertex Groups 列表检查。如果一个组都没有，回到 Object Mode 重新做 `Ctrl + P`，确认弹出的菜单是 With Automatic Weforms。

### 4.3 摆个 A-Pose

Pose Mode 下：
- `UpperArm.R`：`R` → `Z` → `15`（手臂微微张开）
- `LowerArm.R`：`R` → `Z` → `15`（前臂也微微张开）
- `LowerLeg.R`：`R` → `X` → `-5`（膝盖微微前弯）
- 左腿同

> **小贴士**：游戏行业默认导出 **A-Pose**（手臂微张）而不是 **T-Pose**（手臂平举），因为 A-Pose 蒙皮变形更平滑，特别是肩膀那块不会出现"拉扯感"。所以我们从一开始就用 A-Pose。

---

## 第 5 步：K 帧做挥手动脚

### 5.1 打开时间轴

默认底部就有一个 Timeline 面板。**更专业的方式**是用右侧的 Dope Sheet 或下方 Editor Type → **Action Editor**：

1. 底部那条横条左侧的 Editor Type 切换器 → 选 **Dope Sheet** → 上方再切到 **Action Editor**
2. 这是动画关键帧编辑器

### 5.2 录第 1 帧：A-Pose

1. 把时间游标拖到第 **1 帧**（最左边）
2. Pose Mode 下，选中所有骨头 → `A`
3. 鼠标停在 3D 视口 → 按 `I` → 选 **Whole Character** → 这一帧就把当前所有骨头状态全部记录下来了
4. 时间条上会出现一个橙色菱形 = 第 1 帧关键帧

### 5.3 录第 20 帧：挥手

1. 时间游标拖到第 **20 帧**
2. 选中 `UpperArm.R` → `R` → `Z` → 输入 `90`（手臂从下垂变成水平挥出）
3. 选中 `LowerArm.R` → `R` → `Z` → 输入 `45`（手肘弯 45 度）
4. 鼠标停在视口 → `I` → `Whole Character`
5. 录第 40 帧：手回到原位（重复上面的角度值）
6. 录第 60 帧：右腿抬起 `UpperLeg.R` `R` `X` `-30` + `LowerLeg.R` `R` `X` `30`

按空格键（Spacebar）回放 —— 小人开始挥手 + 抬腿了！

### 5.4 循环动画

1. 时间轴末端 → 底部 Timeline → **End Frame** 改成 `48`
2. `Shift + E` → **Cyclic** → **Make Cyclic** → 头尾关键帧会自动插值成无缝循环
3. 按 Spacebar 看看是不是真的循环了

> **小贴士**：Shift+E 循环只在 Action Editor 里所有关键帧完美头尾一致时才无缝。如果跳帧了，回 Edit 模式手动微调第 48 帧的角度让它和第 1 帧一模一样。

---

## 第 6 步：导出 glTF（不是 FBX！）

Godot 4 对 glTF 2.0 支持**远好于** FBX（材质、动画、骨骼全保）。所以现在导出 glTF：

### 6.1 安装 glTF 导出器

Blender 4 内置，不需要装额外插件。

### 6.2 导出设置

1. `File` → `Export` → **glTF 2.0 (.glb)**
2. 文件名 `stickman.glb`，格式选 **glb**（单文件，比 gltf+bin+texture 简单）
3. 右侧 **Include** 选项卡：
   - **勾选** `Armature`（必有）
   - **勾选** `Mesh`
   - **勾选** `Animation`（如果没勾，导出后 Godot 看不到动画）
4. **Animation** 选项卡：
   - `Animation Mode` → **Actions**
   - 下方会列出所有 Action，**只保留** `Stickman_Wave`（默认名是文件名），其他删掉
5. **Transform** 选项卡：
   - `+Y Up` ✓（Blender 默认 Z Up，Godot 默认 Y Up，勾这个 Godot 里模型才是正的）
6. 点 **Export glTF**

文件大小应该 100-300 KB，看贴图复杂度。

---

## 第 7 步：拖进 Godot 4 跑起来

### 7.1 创建 Godot 项目

1. 打开 Godot 4.6 → New Project
2. 资产根目录（res://）下建一个 `models/` 文件夹
3. 把 `stickman.glb` 拖进 `models/` 文件夹
4. 双击它，Godot 会自动导入

### 7.2 创建 3D 场景

1. 在场景树点 + → **Node3D** → 改名为 `StickmanScene`
2. 双击 stickman.glb，**右键 instanced 出来的根节点** → `Save as...` → `stickman.tscn`（存到 `scenes/` 下）
3. 把 `stickman.tscn` 拖到 `StickmanScene` 下作为子节点
4. 给 `StickmanScene` 加一个 `Camera3D`：`+` → Camera3D → `Transform.Position` = `(0, 1.5, 4)`，rotation.x = `-15` 度（俯视一点）
5. 加个 `DirectionalLight3D`（默认就有）和 `WorldEnvironment`（默认就有）

按 F5 跑——你应该看到那个低多边形小人站在灰色地面上，**但不会动**。下一步让它动。

### 7.3 加载 glTF 自带的动画

打开 `stickman.tscn`，你会看到根节点有个 `$AnimationPlayer`（glTF 导入时自动生成）。

> **常见坑 3：AnimationPlayer 里 lib 列表是空的 / 显示 "animations" 但找不到名字**
> 原因：导出时没勾 Animation，或者 Action 名字被重命名了。回 Blender 检查 Action Editor 里 Action 名字是不是叫 `Stickman_Wave`（或者你取的名字），导出时确认 Animation Mode = Actions + 列表里勾选了这个 Action。

写一个简单的脚本让它循环挥手：

```gdscript
# res://scripts/stickman_controller.gd
extends Node3D

@onready var anim: AnimationPlayer = $Stickman/AnimationPlayer

func _ready() -> void:
    if anim:
        # 播放 glTF 自带的动画
        anim.play("Stickman_Wave")
        # 设为循环
        var lib := anim.get_animation("Stickman_Wave")
        if lib:
            lib.loop_mode = Animation.LOOP_LINEAR

func _input(event: InputEvent) -> void:
    # 按 R 重新触发挥手
    if event is InputEventKey and event.pressed and event.keycode == KEY_R:
        if anim and anim.has_animation("Stickman_Wave"):
            anim.stop()
            anim.play("Stickman_Wave")
```

把这个脚本挂到 `StickmanScene` 根节点上。F5 跑，按 R 就能看到循环挥手。

---

## 第 8 步（加分项）：用 Godot AnimationPlayer 做"附加动画"

如果你想在小人挥手的同时**让整个场景的相机绕着转**，Godot 自带的 `AnimationPlayer` 可以做到：

1. 选中 `Camera3D` → 顶部菜单 Animation → **Animation Player** 面板 → `Animation` → `New` → 命名 `Camera_Orbit`
2. 时间轴设 60 帧
3. 第 1 帧：插值相机位置 `(0, 1.5, 4)`
4. 第 30 帧：位置 `(4, 1.5, 0)`，rotation.y = `deg_to_rad(90)`
5. 第 60 帧：回到 `(0, 1.5, 4)` + rotation.y = 0
6. `loop_mode` 改 `Loop Linear`

现在场景里**有两套独立动画**：模型内部骨骼动画 + 相机外部动画，同时跑互不干扰。

> **小贴士**：Godot 4 的 AnimationPlayer **不是状态机**，只是单条动画播放器。要做"待机 / 跑步 / 攻击"状态切换需要 AnimationTree（这个我们之前在 06-24 Godot 动画系统那篇详细讲过）。今天先用 AnimationPlayer 跑通单条动画，状态机留到后面。

---

## 常见坑点 + 解决方案（总结）

| # | 坑 | 原因 | 解决 |
|---|----|------|------|
| 1 | `Ctrl+P` 看不到 "With Automatic Weights" | 父子选择顺序反了 | **先选子（网格）→ Shift 加选父（骨架）**，再 Ctrl+P |
| 2 | 旋转骨头整片网格扭曲 | 自动权重失败 / Vertex Groups 是空的 | 重做 Ctrl+P → With Automatic Weforms；或手动 Weight Paint |
| 3 | Godot 里 AnimationPlayer 找不到动画 | 导出 glTF 时没勾 Animation / 没勾具体 Action | 重导：勾 Include.Animation + Animation Mode = Actions + 勾选 Action 名 |

---

## 延伸阅读

1. **Blender 官方骨骼绑定手册（最权威）**：https://docs.blender.org/manual/en/latest/animation/armatures/index.html —— 包含 IK 约束、Driver、Shape Keys 进阶技巧
2. **Godot 4 官方 Importing 3D Scenes 文档**：https://docs.godotengine.org/en/stable/tutorials/io/index.html —— glTF / FBX / COLLADA 各种导入器的坑和配置
3. **Mixamo**：https://www.mixamo.com —— Adobe 免费的角色 + 动画库，不会做绑定可以直接下载现成的，但要注册账号。今天学完绑定逻辑你就能改 Mixamo 导出的模型了

---

## 软件版本 & 数据来源

- Blender：4.2 LTS（4.x 系列通用）
- Godot：4.6（4.x 系列通用）
- glTF 导出器：Blender 内置
- 教程截图数据来源：Blender 4.2 LTS 官方手册 + Godot 4.6 官方文档 + Mixamo 角色参考
- 撰写日期：2026-06-30

> **下一篇预告**：Blender 子方向已经全部覆盖完了。明天（07-01）开始进入「端到端 3D 场景工作流」综合实战：用 Blender 做一个小房子 → 导出 glTF → 拖进 Godot → 加 3D 场景 → 给两台机器开 ENet 让对方看到你建的房子。系列老粉不要错过。