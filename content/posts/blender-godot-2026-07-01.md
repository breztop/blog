---
title: 'Blender & Godot 3D 教程 - 2026-07-01 - Godot 4 3D 基础零基础入门:从空场景到能转视角的 3D 小世界 (MeshInstance3D / Camera3D / Light / WorldEnvironment)'
date: 2026-07-01T04:00:00+08:00
description: '零基础打开 Godot 4.6 做 3D 场景:从新建项目开始,依次学会 3D 场景根节点 Node3D、MeshInstance3D(地面/立方体/胶囊/球体)、Camera3D 视角、DirectionalLight3D + WorldEnvironment(天空 + 环境光 + 雾)、StandardMaterial3D 基础 PBR 材质,最后写 30 行 GDScript 让相机跟着鼠标转视角,做出一个能转能看的 3D 小世界。'
tags: [godot, 3d, tutorial, meshinstance, camera, light, worldenvironment]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-07-01.png
  alt: 'Godot 4 编辑器暗色 UI 里一个 3D 视口,显示低多边形草地上有红蓝绿三色立方体胶囊和球体,蓝天带云,软阴影,角落是相机视角辅助图标'
---

![cover](/images/posts/blender-godot-2026-07-01.png)

> **上一篇回顾**:2026-06-29 我们用 GDScript 写了一个「点地面就生成小球、点小球就跳一下」的 Demo,把 **变量 / 信号 / 生命周期** 摸了一遍。今天我们换个维度——**3D 场景**。不写代码先把 3D 世界搭起来,再让相机转起来,这是所有 3D 游戏的「根」。

## 一、为什么先学 3D 基础?

如果你打开过 Unity、Unreal、Godot 任何一个 3D 编辑器,大概率会被一坨陌生的英文节点名劝退:`MeshInstance3D`?`Camera3D`?`WorldEnvironment`?别慌,它们其实就 3 个角色:

- **MeshInstance3D** =「3D 模型在场景里的演员」(地板、方块、人物都是它)
- **Camera3D** =「你的眼睛」,决定你看到 3D 世界的哪个角度
- **Light / WorldEnvironment** =「太阳 + 天空 + 大气」,决定世界长什么样

今天目标:**0 基础从空项目,15 分钟内搭出一个能转视角的 3D 小世界**,全靠点击和拖拽,只需要 ~30 行 GDScript。

## 二、准备:安装 Godot 4.6

> **软件版本**:本教程基于 **Godot 4.6**(2026-06 最新稳定版,数据采集时间 2026-07-01)。

1. 打开 [godotengine.org/download](https://godotengine.org/download),下载 **Godot 4.6 std** 版(microsoft store / Steam 版也行,不要下 .NET 版除非你要写 C#)
2. 双击运行,点 **New Project**
3. **Project Name** 输入 `MyFirst3DWorld`
4. **Project Path** 选个空目录,比如 `~/projects/MyFirst3DWorld`
5. **Renderer** 选 **Forward+**(默认就是,3D 必备)
6. 点 **Create & Edit**

> 💡 **为什么要 Forward+?** Godot 4 有 3 个渲染后端:Forward+(3D 主力,支持完整 PBR + 全局光照)、Mobile(手机用,功能缩水)、Compatibility(老显卡兼容,2D 为主)。**学 3D 就选 Forward+**,等你哪天要发到手机再切 Mobile。

打开后你会看到一个空场景编辑器,左上角是 Scene 树(目前只有 `Node` 一个根节点),中间是 Viewport(黑色,啥都没有),右边是 Inspector(空)。

## 三、把 2D 场景升级成 3D 场景

新建的项目默认是 2D 场景(`Node2D` 根),我们要换成 3D。

1. **Scene 面板顶部**找到 3 个小图标:`2D Scene` / `3D Scene` / `User Interface`
2. 点中间的 **`3D Scene`**
3. 弹出确认框「Add a Node3D as root?」,点 **Create**
4. 左侧 Scene 树里现在显示一个 `Node3D` 节点,根图标变成了 3D 坐标轴

> 💡 **为什么根节点必须是 Node3D?** Godot 用「父子节点层级」组织场景,根节点类型决定整个场景是 2D 还是 3D。`Node3D` 拥有 X/Y/Z 三个轴的 3D 变换(位置 / 旋转 / 缩放),子节点如果也是 3D 节点,会跟着父节点一起动。`Node2D` 只能管 X/Y,所以在 3D 场景里它会被当成「永远在 Z=0 平面上」的东西,基本废了。

## 四、第一个 3D 演员:地面

地板是一切的开始。

1. 在 Scene 树里**右键 `Node3D` 根节点** → **Add Child Node**
2. 搜索框输入 `MeshInstance3D` → 选中 → **Create**
3. 选中新加的 `MeshInstance3D` 节点,右侧 **Inspector** 找到 **Mesh** 属性(带 `<empty>` 字样)
4. 点 **Mesh** 旁边的下拉或 **`<empty>` 文字** → 选 **New PlaneMesh**
5. PlaneMesh 默认是 2m × 2m 的平面,**Inspector** 里把 **Size** 改成 `X = 20, Y = 20`(20 米见方,够摆东西)
6. 顺便把它**重命名**为 `Ground`(右键 → Rename,或 F2)

> 💡 **PlaneMesh vs BoxMesh vs SphereMesh vs CapsuleMesh vs CylinderMesh**:这 5 个是 Godot 4 内置的「5 种基本几何」,足够搭场景原型。**PlaneMesh** 是只有一面、永远朝上的薄板(常做地面/水面);**BoxMesh** 是立方体(箱子/房子/墙);**SphereMesh** 球(太阳/球类游戏);**CapsuleMesh** 胶囊(角色控制器);**CylinderMesh** 圆柱(树干/柱子)。**PBR 网格以后导 Blender 做的 .glb / .gltf 进来用**。

现在你能在中间的 Viewport 看到一个浅灰色的方块,但它现在是「横着的」——不对啊,PlaneMesh 默认是躺在 X-Y 平面上的(像一张纸),我们踩着它就该让它水平(让它沿着 X-Z 平面展开)。

不,等等,PlaneMesh **默认就是水平的**(顶点法线朝 +Y),你看到的「横着」是因为 Camera3D 还没加,我们是从一个很偏的角度看下去的。

## 五、加 Camera3D:你的第一只「眼睛」

1. **右键 `Node3D` 根节点** → **Add Child Node** → 搜 `Camera3D` → Create
2. 选中新 `Camera3D` 节点
3. **Inspector** 里改 **Position**:`X = 0, Y = 5, Z = 10`(站在 5 米高,离原点 10 米外)
4. **Rotation**:`X = -25`(向下看 25 度)

> 💡 **为什么 X 旋转是负数?** Godot 4 遵循右手坐标系:Z 轴正方向指向屏幕外,X 轴向右,Y 轴向上。**顺时针绕轴为正方向**(从轴的「正方向」往原点看)。绕 X 轴「低头」是顺时针 → 负值。Godot 早期版本是左手系 + 逆时针,老教程容易踩坑,**4.x 一定以这个为准**。

按 **F5** 跑一下(或者右上角 ▶ 按钮),会弹出一个「Select a Main Scene」窗口,选你当前的 `Node3D` 根场景(还没保存的话先 **Ctrl+S** 存为 `main.tscn`)。

跑起来后你应该看到:一个灰白色方块(地面),前面有个立方体,**没有阴影、没有天空**——像在一个无菌实验室里。

按 **Esc** 退出运行。

## 六、加 DirectionalLight3D + WorldEnvironment:让世界「活」起来

### 6.1 加太阳光

1. **右键 `Node3D` 根** → **Add Child Node** → 搜 `DirectionalLight3D` → Create
2. 选中新节点,改 **Rotation**:`X = -45, Y = 30, Z = 0`(模拟下午的太阳,从西北斜照下来)
3. 展开 **Shadow** → 勾选 **Enabled**(开启阴影,这是 3D 游戏的灵魂)
4. 顺带把 Ground 也开阴影接收:选 `Ground` 节点,展开 **MeshInstance3D 下的 Material Override / GeometryInstance3D** → **Cast Shadow** 保持 `On`,**Receive Shadows** 勾上(默认其实就开了)

### 6.2 加 WorldEnvironment(天空 + 环境光 + 雾)

1. **右键 `Node3D` 根** → **Add Child Node** → 搜 `WorldEnvironment` → Create
2. 选中新节点,Inspector 里 **Environment** 属性点 `<empty>` → **New Environment**
3. 点开新建的 Environment 子资源,逐项配置:
   - **Background → Mode**: 选 `Sky`
   - **Background → Sky** → 选 `New Sky`
   - 展开 **Sky** 子资源,顶部 **Sky Material** 选 `New ProceduralSkyMaterial`
   - 调一下 **Sky Top Color** = 浅蓝、**Sky Horizon Color** = 接近白、**Ground Bottom Color** = 暗灰
4. **Ambient Light → Source** 选 `Sky`(让环境光跟天空一致,阴影部分不会死黑)
5. **Tonemap → Mode** 选 `Filmic`(让颜色更接近电影感,不刺眼)
6. 可选:**Fog → Enabled** 勾上,**Fog Light Color** 调淡蓝(远处的物体会淡淡融入天空,提升「真实感」)

按 **F5** 跑一下——现在应该有蓝天、白云、软阴影,远处有雾。一个能看的 3D 世界出来了。

> 💡 **DirectionalLight3D vs OmniLight3D vs SpotLight3D**:**DirectionalLight3D**(方向光,无限远,太阳/月亮用)、**OmniLight3D**(点光源,360° 散开,灯泡/篝火)、**SpotLight3D**(聚光灯,锥形,手电筒/车灯)。3D 游戏里**大部分情况下一个 DirectionalLight3D 当太阳就够**,性能压力最小。

## 七、在地面上摆几个 Mesh

让场景不那么空:

1. **右键 `Node3D` 根** → **Add Child Node** → `MeshInstance3D` → 命名 `Box`
2. **Mesh** → `New BoxMesh`,**Size** 保持 `1, 1, 1`
3. **Position**:`X = 0, Y = 0.5, Z = 0`(1 米高,放地面中央)
4. **再添加** `MeshInstance3D` 命名 `Capsule`,**Mesh** → `New CapsuleMesh`,**Radius** = `0.5`,**Height** = `2`,**Position**:`X = 3, Y = 1, Z = 0`
5. **再添加** `MeshInstance3D` 命名 `Sphere`,**Mesh** → `New SphereMesh`,**Radius** = `0.7`,**Position**:`X = -3, Y = 0.7, Z = 0`

你应该看到:中央一个红/灰立方体(默认 PBR 是浅灰),右边一个胶囊,左边一个球。**颜色不对?**往下看。

## 八、上 PBR 材质:StandardMaterial3D

默认网格是「没有任何视觉特征」的灰色,3D 游戏能好看全靠材质。

1. 选 `Box` 节点,Inspector → **Material Override** 旁边的 `<empty>` → **New StandardMaterial3D**
2. 展开 Material:
   - **Albedo Color**:改成红色 `ff3333`
   - **Metallic**: `0.2`(0 = 纯塑料,1 = 纯金属)
   - **Roughness**: `0.5`(0 = 镜面,1 = 粗糙)
3. 同样给 `Capsule` 做个绿色 PBR 材质,**Roughness** = `0.3`(光滑一点)
4. 给 `Sphere` 做个蓝色,**Metallic** = `0.8`,**Roughness** = `0.2`(金属感球)

按 **F5** 跑一次——现在应该是蓝天白云下,红绿蓝三件套 + 软阴影 + 雾,跟封面图差不太多了。

> 💡 **Albedo / Metallic / Roughness 三剑客**:这是 Unity / Unreal / Godot 通用的 PBR(Physically Based Rendering)三参数。**Albedo** 是固有色(去掉光照后的颜色)、**Metallic** 是金属度、**Roughness** 是表面粗糙度(0 像镜子,1 像水泥)。**新手口诀:塑料就 Metallic=0 + Roughness=0.5,金属就 Metallic=0.9 + Roughness=0.2,试试 0.3 / 0.7 就有磨砂感**。

## 九、让相机跟着鼠标转视角(~30 行 GDScript)

现在 3D 场景搭好了,但你不能动——下面写 30 行 GDScript 让相机能转视角,经典 FPS 风格的「按住右键拖动看世界」。

1. 选 `Camera3D` 节点 → Inspector 顶部 **Script** 属性点 `<empty>` → **New Script** → 路径 `camera_controller.gd` → **Create**
2. 双击打开脚本编辑器,**完全替换**内容为下面这段:

```gdscript
extends Camera3D

# 鼠标灵敏度(像素转角度的倍率)
@export var mouse_sensitivity: float = 0.2
# 上下最大仰角(避免「脖子转 360 度」)
@export var max_pitch: float = 80.0

func _unhandled_input(event: InputEvent) -> void:
    # 只响应「按住右键」+「鼠标移动」的情况
    if event is InputEventMouseMotion and Input.is_action_pressed("ui_right_click"):
        rotate_y(deg_to_rad(-event.relative.x * mouse_sensitivity))
        rotate_object_local(Vector3.UP, deg_to_rad(-event.relative.y * mouse_sensitivity))
        # 限制 X 轴仰角,防止翻滚
        rotation.x = clamp(rotation.x, deg_to_rad(-max_pitch), deg_to_rad(max_pitch))
```

3. **Project → Project Settings → Input Map**(顶部菜单)
4. 右上角 **Add New Action**,名字填 `ui_right_click`(或者随便起,比如 `free_look`)
5. 列表里新行点击 **+** → **Mouse → Right Button**
6. (如果你不用 `ui_right_click` 这个名字,代码里要相应改)

按 **F5** 跑——按住鼠标右键在 Viewport 里拖动,相机就会转视角。**左键点击** 不转。

> 💡 **`_unhandled_input` vs `_input`**:Godot 的输入事件会按 **`_input` → GUI → `_unhandled_input` → `_physics_process`** 顺序传递。**鼠标拖动相机用 `_unhandled_input` 最佳**:它会「让 GUI 优先」,这样你以后加按钮 / 文本框时,鼠标拖到按钮上不会乱转视角。`_input` 则会拦截一切,经常出 bug。

## 十、完整项目结构

```
MyFirst3DWorld/
├── project.godot
├── main.tscn          # 主场景
└── camera_controller.gd   # 相机控制脚本
```

**main.tscn 节点树**:
```
Node3D (root)
├── Ground           (MeshInstance3D + PlaneMesh 20x20)
├── Camera3D         (Position 0,5,10 + Rotation -25,0,0 + camera_controller.gd)
├── DirectionalLight3D  (Rotation -45,30,0, Shadow Enabled)
├── WorldEnvironment    (Environment + ProceduralSkyMaterial)
├── Box             (MeshInstance3D + BoxMesh 1x1x1, 红色 PBR)
├── Capsule         (MeshInstance3D + CapsuleMesh R=0.5 H=2, 绿色 PBR)
└── Sphere          (MeshInstance3D + SphereMesh R=0.7, 蓝色金属 PBR)
```

## 十一、3 个常见坑点

### 坑 1:运行后一片粉红 / 黑色

**症状**:按 F5 跑起来,模型是「粉红色 + 黑色棋盘格」(missing material / missing texture)。
**原因**:98% 是**项目目录里有 MeshInstance3D 引用了 `.tres` / `.material` 外部资源,被手动删了或路径不对**。
**解决**:**FileSystem 面板里右键 → Reimport Folders**;如果是 .tres 资源丢失,**FileSystem 里搜不到就重新创建 StandardMaterial3D**。**另一个常见原因**:**你双击了 project.godot** 而不是 main.tscn,Godot 找不到主场景,把所有 Mesh 渲染成粉红。

### 坑 2:有网格没阴影,或阴影是「马赛克」

**症状**:Box 立方体下面没影子,或影子一格一格锯齿。
**原因**:
- 没影子 = **DirectionalLight3D 的 Shadow Enabled 没勾**,或**地面没勾 Receive Shadows**
- 马赛克 = **Shadow Atlas 太小**(默认 4096 够大多数场景,几百个动态光就需要调大)
**解决**:
- 1) 选 `DirectionalLight3D` → Inspector → **Shadow** → **Enabled** ✅
- 2) 选 `Ground` → **GeometryInstance3D → Cast Shadow = On**(默认是),**Receive Shadows = On**(默认是)
- 3) 马赛克 → **Project Settings → Rendering → Lights And Shadows → Directional Shadow → 16 Bit** 勾上(阴影深度精度从 8bit 提到 16bit,质量飞跃)

### 坑 3:相机转视角「脖子能转 360 度」

**症状**:按住右键转视角,能转到「头朝下看脚后跟」。
**原因**:上面代码没加 `clamp` 仰角,或者 `rotate_object_local` 累积到了 ±90° 之后会翻转。
**解决**:确保 `_unhandled_input` 里有这一行:
```gdscript
rotation.x = clamp(rotation.x, deg_to_rad(-max_pitch), deg_to_rad(max_pitch))
```
**另一个坑**:用 `rotate_x()` 而不是 `rotate_object_local(Vector3.UP, ...)` 处理俯仰——`rotate_x` 是**世界轴**的 X,相机 yaw 旋转后世界轴变了,会出问题。**俯仰用 `rotate_object_local(Vector3.UP, ...)` 永远是对的**(本地轴 Y)。

## 十二、延伸阅读

1. **官方文档 — Godot 4 场景树与节点概念**:[docs.godotengine.org/en/stable/tutorials/scripting/scene_tree.html](https://docs.godotengine.org/en/stable/tutorials/scripting/scene_tree.html)——3D 场景里所有「父子关系 / 继承 / 实例化」的真相
2. **官方文档 — WorldEnvironment + Environment**:[docs.godotengine.org/en/stable/tutorials/3d/environment_and_post_processing.html](https://docs.godotengine.org/en/stable/tutorials/3d/environment_and_post_processing.html)——天空/雾/SSAO/辉光/色调映射,做电影感 3D 场景必看
3. **Heartbeast — 3D Godot Tutorial for Beginners(YouTube)**:[youtube.com/watch?v=nk7KHgmkT1M](https://www.youtube.com/watch?v=nk7KHgmkT1M)——心形大叔是 Godot 圈最会教新手的 UP 主,免费系列从「打开 Godot」到「做完一个 3D RPG」

## 十三、下一节预告

明天 7-2 我们会**用 Blender 4.x 做一个低多边形的「机器人」模型 → 导出 glTF → 拖进今天搭好的 3D 场景 → 装上 AnimationPlayer 跑一个走路的循环动画**。从「3D 基础」跨到「跨工具流程 + 动画系统」,继续零基础实战。

---

> **软件版本**:Blender 4.x · Godot 4.6 · GDScript 4.x
> **数据采集时间**:2026-07-01 10:02 (Asia/Shanghai)
> **系列**:Blender & Godot 3D 零基础每日教程,持续更新中
