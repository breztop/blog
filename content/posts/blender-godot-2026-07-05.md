---
title: 'Blender & Godot 3D 教程 - 2026-07-05 - Blender 4 灯光与渲染零基础入门:HDRI + 三点光 + Cycles vs Eevee,把你昨天的赛博朋克街区「点亮」'
date: 2026-07-05T04:00:00+08:00
description: '零基础搞懂 Blender 4.2 的灯光与渲染:HDRI 环境光让街区瞬间有「外景感」、三点光(Key/Fill/Rim)给主角打出立体感、Cycles vs Eevee 的速度/画质权衡、Glossary 渲染降噪、bpy 脚本一键导出 glTF,直接拖进 Godot 4 联机大厅当战斗场景。'
tags: [blender, lighting, rendering, hdri, three-point-light, cycles, eevee, cyberpunk, gltf, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-07-05.png
  alt: 'Blender 4 暗色编辑器里一座程序化生成的赛博朋克街区被点亮,HDRI 天空泛着蓝紫色,主楼打上 Key 暖光、背面填 Fill 冷光、楼顶 Rim 描边,地面反光有窗户光斑,Eevee 实时预览与 Cycles 最终渲染对比窗口同时显示'
---

![cover](/images/posts/blender-godot-2026-07-05.png)

> **上一篇回顾**:2026-07-04 我们用几何节点造了一座「赛博朋克街区」—— 5×5 楼阵、霓虹招牌、反光地面都齐了。但截图发现画面**灰扑扑没灵魂**?
>
> 这是因为没有任何「灯」。**建模决定形状,灯光决定情绪**。今天就专门解决这件事。

> 适用版本：Blender 4.2 LTS（4.0+ 全部适用），约 35 分钟可跑通
> 面向读者：完全没做过灯光的初学者，今天第一次听到 "HDRI" / "三点光" / "Cycles / Eevee"

---

## 一、为什么「灯」决定一切？

先看对比。一模一样的赛博朋克街区（沿用昨天的几何节点）：

| 状态 | 画面感觉 |
|---|---|
| 无灯光（默认环境） | 灰白、所有材质都像被漂白 |
| 只有太阳（Point Sun） | 影子很硬、阴影面漆黑 |
| 加 HDRI 天空 | 有了「外景感」、金属/玻璃开始反光 |
| 加三点光 | 主角楼被「打亮」、有立体感 |

> 💡 **为什么灯光比建模更重要？**
> 在游戏里玩家会**盯着看的不是所有方块,而是「主角 + 视觉焦点」**。哪怕街区 5000 栋楼,只要重点灯打得到位,玩家就觉得「画风好看」;反之,做得再精致的楼,全灰也像 1990 年的 Voodoo 演示。

---

## 二、Blender 里灯的四种类型（先认脸）

打开 Blender,在右上角「添加」菜单 → 「Light」可以看到四种:

| 类型 | 图标 | 用途 | 类比 |
|---|---|---|---|
| **Point** | ⚫ | 一个灯泡,360° 均匀发光 | 台灯 |
| **Sun** | ☀ | 无限远的平行光,无距离衰减 | 太阳 |
| **Spot** | 🔦 | 锥形光,有 Cone Angle | 手电筒/车灯 |
| **Area** | ▭ | 面光源,面积越大越柔和 | 窗户光、屏幕光 |

**关键参数（必看）**:
- **Power**:功率(瓦特),默认数值很小,新手第一反应是「不亮」,把 Power 调到几百到几千瓦才对
- **Color**:色温(暖黄 3000K ~ 冷蓝 8000K)
- **Shadow**:开关软阴影、阴影贴图分辨率

> 💡 **为什么 Blender 默认数值「不亮」?**
> 因为 Power 单位是 Watts,而我们直觉感受是「够不够亮」。游戏场景常用范围 **Point 100-1000W / Sun 1-5W / Spot 200-2000W / Area 500-5000W**。先无脑调大,再按视觉效果收。

---

## 三、HDRI 环境光：让街区瞬间有「外景感」

HDRI = High Dynamic Range Image(高动态范围图像)。它是一张 360° 全景图,既当背景也当光源,**金属/玻璃一放进去就自动反光出整张图**。

### 3.1 下载一张免费 HDRI

推荐 **Poly Haven CC0**（无版权、可商用）:
- 网址:<https://polyhaven.com/hdris>
- 搜索「night」「cyberpunk」「rooftop」,下载 `.hdr` 文件
- 选 1k 或 2k,不要 8k(Blender 跑不动)

### 3.2 在 Blender 里挂载 HDRI

1. 顶部菜单 **Edit → Preferences → Add-ons**(可选)
2. 主视图右上角**着色器图标**(球体)→ 切换到「World」选项卡
3. **Use Nodes** 打勾
4. 默认有一个 `Background` 节点,点击 **Color** 旁边黄色圆点 → **Environment Texture**
5. 弹出文件选择器,选你下载的 `.hdr`
6. 你会看到背景天空 + 街区材质开始反射这张图

> 💡 **为什么必须 Use Nodes?**
> Blender 4 默认是非节点的世界设置(纯色背景)。但只有节点模式才能加载 HDRI 和精细调整曝光/色温。这是新手第一个卡点,记住:**看到「Use Nodes」就勾**。

### 3.3 控制 HDRI 强度

`Background` 节点有个 **Strength** 参数,默认 1.0。

- 太高(5+):过曝、全白、丢失细节
- 太低(0.1):不起作用
- **游戏场景推荐 0.8 ~ 1.5**

实战中常常把 HDRI 强度调低作为"环境补光",再用手动灯做"重点光"。

---

## 四、三点光：经典「打灯」套路（Key / Fill / Rim）

三点光 = 主光 + 补光 + 轮廓光。任何电影/游戏/CG 教学 30 秒内都会提到它。

```
        [Rim]（后方轮廓）
            ↓
         主体
        /    \
     [Fill]  [Key]（主光）
   弱/冷色    强/暖色
```

### 4.1 Key 主光（最强、最有方向性）

1. `Shift + A` → **Light → Area**(推荐,最柔和)
2. 放到主体(主楼)前上方 45°,大致如图位置
3. 参数:
   - **Power**:3000 ~ 8000 W
   - **Color**:暖黄(选 `#FFC080` 或 RGB ~ `(1.0, 0.75, 0.5)`)
   - **Shape → Size**:0.5(越大越柔和,越小越硬)
   - **Shadow → Ray Visibility**:勾 Diffuse + Glossy

### 4.2 Fill 补光（柔化阴影、降低对比）

1. 再 `Shift + A` → **Light → Area**
2. 放到主体另一侧(斜对面),**Power 调到 Key 的 1/3 ~ 1/4**(比如 Key=5000W,Fill=1500W)
3. **Color**:冷色(`#A0C0FF` 或灰蓝)
4. 可以**关闭 Shadow**(纯补光,不抢戏)

### 4.3 Rim 轮廓光（把主体从背景「剥」出来）

1. 再 `Shift + A` → **Light → Area**
2. 放到主体**正后方**,对着摄像机
3. **Power**:中等(2000 ~ 4000W)
4. **Color**:强对比色(常用冷蓝或紫红,适配赛博朋克)
5. 这盏灯专门让主体边缘「亮起来」,形成"勾边"

> 💡 **为什么三点光这样分工?**
> - **Key 决定主光源方向**(太阳/路灯/屏幕)
> - **Fill 救阴影**(不然阴面全黑,看不清材质)
> - **Rim 把主体从背景里"切"出来**(逆光,让玩家第一眼看到焦点)
>
> 三盏灯总功率比单灯大,但**画面戏剧性**天差地别。

### 4.4 实战小技巧:按 `N` 打开右侧 Transform 面板手输坐标

| 灯 | X | Y | Z | 朝向 |
|---|---|---|---|---|
| Key  | 4  | -4  | 5  | 指向 (0, 0, 2) |
| Fill  | -5 | -3  | 2  | 指向 (0, 0, 2) |
| Rim   | 0  | 5   | 4  | 指向 (0, 0, 2) |

(主体假设在原点,高 2 米)

---

## 五、Cycles vs Eevee：怎么选？

| 维度 | **Eevee** | **Cycles** |
|---|---|---|
| 渲染速度 | 🚀 实时(预览即时) | 🐢 慢(看机器,5 秒 ~ 5 分钟) |
| 画面真实度 | 卡通、干净、像游戏 | 电影、真实、像 3D 电影 |
| 物理正确性 | 近似 | 物理正确(光线追踪) |
| 全局光照 | 默认没有,要 Light Probe | 自带 |
| 焦散/散射 | 不支持 | 支持 |
| 体积光 | 不支持 | 支持 |
| 适用 | **游戏资产预览** | 影视/广告 |

### 5.1 切换渲染引擎

右上角**渲染引擎下拉框** → 选 Eevee 或 Cycles。

### 5.2 推荐工作流

1. **建模阶段全程 Eevee**:实时调灯、实时看效果
2. **最终出图 / 导出时切到 Cycles**:得到最高画质
3. **Cycles 必开降噪**:Render Properties → Sampling → **Denoise = OpenImageDenoise**,Samples 调到 128 即可

> 💡 **为什么不是 Samples 越高越好?**
> Cycles 的 Samples 是「光线数量」。越高越干净,但时间线性增加。128 + 降噪 ≈ 4096 + 无降噪,但**快 30 倍**。游戏场景几乎不需要 4096+。

---

## 六、实战:把昨天赛博朋克街区"点亮"

### 6.1 打开昨天的 .blend

如果你跟着昨天的教程做过,直接打开 `cyberpunk_block.blend`。
否则可以临时跑一段最小化代码(纯几何节点 + 三点光):

```python
# save as make_block.py, run with `blender --python make_block.py --`
import bpy

# 清空场景
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

# 创建地面
bpy.ops.mesh.primitive_plane_add(size=40, location=(0, 0, 0))
ground = bpy.context.active_object
ground.name = "Ground"

# 创建 5x5 楼阵
for x in range(-2, 3):
    for y in range(-2, 3):
        h = 2 + abs(x + y) * 0.5
        bpy.ops.mesh.primitive_cube_add(
            size=2,
            location=(x * 4, y * 4, h / 2),
        )
        cube = bpy.context.active_object
        cube.scale = (1, 1, h / 2)
        cube.name = f"Building_{x+2}_{y+2}"

# 设置渲染引擎为 Eevee(快速预览)
bpy.context.scene.render.engine = 'BLENDER_EEVEE_NEXT'

print("场景搭建完成。请手动加灯或继续运行 add_lights.py")
```

### 6.2 加灯 + HDRI 的 bpy 脚本

```python
# save as add_lights.py, run with `blender --python add_lights.py --`
import bpy

def make_area_light(name, location, rotation_euler, power, color, size=0.5, shadow=True):
    """添加一盏 Area 灯并设置参数"""
    bpy.ops.object.light_add(type='AREA', location=location, rotation=rotation_euler)
    light = bpy.context.active_object
    light.name = name
    light.data.energy = power
    light.data.color = color
    light.data.size = size
    if not shadow:
        light.data.use_shadow = False
    return light

# 1. Key 主光(暖黄,右上)
make_area_light(
    name="Key_Light",
    location=(6, -5, 7),
    rotation_euler=(1.1, 0, 0.8),  # 大致指向场景中心
    power=5000,
    color=(1.0, 0.75, 0.5),
    size=1.0,
)

# 2. Fill 补光(冷蓝,左侧低)
make_area_light(
    name="Fill_Light",
    location=(-7, -3, 3),
    rotation_euler=(0.9, 0, -0.6),
    power=1500,
    color=(0.6, 0.7, 1.0),
    size=1.5,
    shadow=False,
)

# 3. Rim 轮廓光(紫色,后方)
make_area_light(
    name="Rim_Light",
    location=(0, 6, 5),
    rotation_euler=(0.6, 0, 3.14),
    power=3000,
    color=(0.8, 0.4, 1.0),
    size=0.8,
)

# 4. 配 HDRI(假设你已经下载了 rooftop_night_1k.hdr)
import os
hdri_path = "/path/to/rooftop_night_1k.hdr"
if os.path.exists(hdri_path):
    world = bpy.context.scene.world
    world.use_nodes = True
    bg_node = world.node_tree.nodes.get("Background")
    if bg_node:
        bg_node.inputs[1].default_value = 0.8  # Strength
        env_tex = world.node_tree.nodes.new("ShaderNodeTexEnvironment")
        env_tex.image = bpy.data.images.load(hdri_path)
        world.node_tree.links.new(env_tex.outputs["Color"], bg_node.inputs["Color"])

print("灯光已就位。F12 渲染看效果。")
```

> 💡 **为什么 Key 选暖色、Fill 选冷色?**
> 冷暖对比是**电影感**的核心套路。暖色(类似日光)做主光、冷色(类似天光)做补光,大脑自动解读为「早上 / 室内外对比」,画面就有戏剧性。两者越冷暖差异越大,戏剧性越强。

### 6.3 渲染并对比

1. **视图预览**:`Z` 切换到 `Rendered` 模式,实时看效果
2. **最终出图**:`F12` → 渲染窗口 → `Image → Save As`
3. **比较两引擎**:
   - 切到 **Eevee**,`F12` 渲染(秒出)
   - 切到 **Cycles**,打开降噪,`F12` 渲染(几十秒)
   - 对比保存为 `street_eevee.png` 和 `street_cycles.png`

---

## 七、导出 glTF 进 Godot 4 联机大厅

昨晚我们讲过街区导出,但**带灯的街区要怎么导**?

### 7.1 关键勾选

`File → Export → glTF 2.0 (.glb)`:

| 选项 | 推荐 |
|---|---|
| Format | glTF Separate (`.gltf` + `.bin` + 贴图) 或 Binary (`.glb`) |
| **Lighting** | ⚠️ **改成「Standard」或「None」** |
| Cameras | 不勾(用 Godot 自己的相机) |
| Punctual Lights | ❌ **不勾** ⚠️ |
| Animation | 不勾 |

### 7.2 为什么「灯」不能一起导出?

**Godot 4 不支持 glTF 的灯节点**(技术规范里是「Not Implemented」)。

正确做法:
- **灯光只用来预览 / 出图**
- **进 Godot 后重新用 Godot 的 OmniLight3D / SpotLight3D / DirectionalLight3D 重新打**
- 昨天的 `WorldEnvironment.environment` 是另一个坑——HDRI 背景可以导出为 KTX2 贴图,但灯不行

### 7.3 在 Godot 里重现三点光

```gdscript
# res://scripts/three_point_light.gd
# 挂在主场景的根节点上
@tool
extends Node3D

@export var key_power: float = 5.0
@export var fill_power: float = 1.5
@export var rim_power: float = 3.0

func _ready() -> void:
    # Key 主光
    var key := OmniLight3D.new()
    key.name = "KeyLight"
    key.light_color = Color(1.0, 0.75, 0.5)
    key.light_energy = key_power
    key.position = Vector3(6, -5, 7)
    add_child(key)

    # Fill 补光
    var fill := OmniLight3D.new()
    fill.name = "FillLight"
    fill.light_color = Color(0.6, 0.7, 1.0)
    fill.light_energy = fill_power
    fill.position = Vector3(-7, -3, 3)
    add_child(fill)

    # Rim 轮廓光
    var rim := DirectionalLight3D.new()
    rim.name = "RimLight"
    rim.light_color = Color(0.8, 0.4, 1.0)
    rim.light_energy = rim_power
    rim.rotation = Vector3(deg_to_rad(60), deg_to_rad(180), 0)
    add_child(rim)
```

> 💡 **为什么 Godot 用 OmniLight3D 而不是 AreaLight3D?**
> Godot 4 没有 AreaLight3D(预计 4.4+ 加入)。OmniLight3D 是点光源,只要**调小 omnilight 的 omni_range + 加 shadow_distance** 也能模拟面积光柔和度。

---

## 八、常见坑点（Debug Checklist）

### 坑 1：「为什么我的 HDRI 不反光？」

**原因**:HDRI 已经加载,但材质的 **Metallic / Roughness** 没设对。
**解决**:
- 金属类材质:`Metallic = 1.0`,`Roughness = 0.2-0.5`
- 玻璃类:`Transmission = 1.0`,`Roughness = 0.0-0.1`
- 只有这两个参数对的时候,HDRI 才会"被看到"

### 坑 2：「Eevee 看不到 HDRI 反光,Cycles 才有」

**原因**:Eevee 默认**不开反射探针**(Reflection Probe),金属面只能反射环境贴图,不是真实反射。
**解决**:
- 添加 `Light Probe → Reflection Cubemap`
- 调整影响范围,勾 `Interior`
- 或在材质里把 `Specular → Specular IOR` 调高

### 坑 3：「Cycles 渲染 30 分钟没完」

**原因**:Samples 调到 4096+ 又没开降噪,机器顶不住。
**解决**:
- Samples 降到 **128 + OpenImageDenoise**
- 关 `Caustics`(`Light Paths → Caustics = 0`)
- 改用 GPU 渲染(`Render Properties → Device = GPU Compute`)

### 坑 4：「HDRI 强度 1.0 但背景漆黑」

**原因**:`.hdr` 文件色域是线性,但 Blender 默认视图是 sRGB,显示器范围 0~1,超出范围就显示成纯白/纯黑。
**解决**:
- `Background` 节点 **Strength 调到 0.5 ~ 1.5**
- 或者 `Color Management → Look = High Contrast`

### 坑 5：「三点光打完,场景反而比之前还暗」

**原因**:Area Light 默认 `Power` 单位是 `W`(瓦特),数值 100 是真的 100 瓦——台灯水平。
**解决**:`Power` 直接调大:室外主光 3000~8000W,室内主光 1000~3000W。

---

## 九、延伸阅读

1. **Blender 官方灯光手册**(权威、全面,有数学公式)<https://docs.blender.org/manual/en/latest/render/lights/index.html>
2. **Poly Haven HDRI / Texture 库**(CC0 商用免费)<https://polyhaven.com/hdris>
3. **"Cinematic Lighting in Blender" – Blender Guru**(YouTube,25 分钟三点光实战)<https://www.youtube.com/watch?v=YsJ3j5gJrtA>
4. **Godot 4 Lights and Shadows 文档**(导出后如何在 Godot 里还原)<https://docs.godotengine.org/en/stable/tutorials/3d/lights_and_shadows.html>

---

## 十、版本与数据来源

- **Blender 版本**:Blender 4.2 LTS(2025-07 发布的长期支持版本)
- **HDRI 来源**:Poly Haven CC0 素材库(rooftop_night 系列)
- **教程代码**:基于 `bpy` 4.2 API,Godot 端基于 Godot 4.6 GDScript 4.x
- **数据抓取时间**:2026-07-05 10:02 Asia/Shanghai

---

## 今日小结

| 你学到了 | 一句话 |
|---|---|
| **HDRI** | 一张 360° 全景图,既当背景又当环境光 |
| **三点光** | Key(暖主光)+ Fill(冷补光)+ Rim(轮廓光) |
| **Cycles vs Eevee** | Eevee 快、Cycles 真,工作流用 Eevee 调,Cycles 出图 |
| **导出陷阱** | glTF 不导灯,Godot 里重新打 |

**明天预告**:07-06 我们把这座亮起来的街区**拖进 Godot 4 联机大厅**,让它变成 4 人混战的「霓虹战场」——多个玩家同时进出,Server 同步灯的开关(比如"停电事件")。

> 如果这篇让你终于搞懂了「三点光」到底在干嘛,点个赞 / 留个评论让我知道 ✨