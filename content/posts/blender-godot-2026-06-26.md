---
title: 'Blender & Godot 3D 教程 - 2026-06-26 - Godot 4 着色器零基础入门：空间着色器与屏幕空间后处理，让你的 3D 角色「会发光」'
date: 2026-06-26T04:00:00+08:00
description: '零基础打开 Godot 4.6 着色器：理解着色器是什么、空间着色器 (spatial shader) 的 vertex + fragment 两段式结构、用 Fresnel 边缘光给 3D 角色加赛博朋克发光圈、用 ColorRect 做全屏 CRT 扫描线后处理。'
tags: [blender, godot, multiplayer, 3d, tutorial, shader, fresnel]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-26.png
  alt: '一个低多边形 3D 角色身上覆盖着青色和品红色的霓虹边缘光，背景是简单的地面与深蓝天空，赛博朋克风'
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-26.png)

> 系列第十四篇。06-20 我们让 3D 角色能走能跳，06-24 又给他加上了「待机 / 跑 / 跳」的状态机动画——**但他看起来还是灰扑扑的塑料感，跟「赛博朋克」一点关系都没有**。  
> 今天用 **着色器 (Shader)** 改这件事。**你不需要懂数学、不需要会 GLSL**，只要照着本篇敲 4 段 shader 代码，就能让一个普通立方体 / 角色：
> 1. 边缘自带霓虹光（Fresnel 边缘光）
> 2. 表面像水波一样流动（UV 偏移 + 时间）
> 3. 整个游戏画面多一层「CRT 扫描线 / 故障抖动」（屏幕空间后处理）
> 
> 这一篇不依赖 Blender 建模；如果你手头已经有 06-13 / 06-23 做的低多边形房子，**直接拿来当实验素材**。

## 0. 本篇目标 & 前置

**学完你能做什么:**

- 说出「着色器 (Shader)」在游戏画面渲染管线 (Render Pipeline) 里到底处在哪一环
- 区分 Godot 4 的 5 种着色器类型（spatial / canvas_item / sky / fog / particles），知道 3D 场景下 99% 用的是 `spatial`
- 写出第一个 spatial shader：让立方体随时间在「红 / 绿 / 蓝」之间循环
- 写出第二个实战 shader：**Fresnel 边缘光**，让任何 3D 模型边缘自带霓虹光圈
- 把 ShaderMaterial 挂到从 Blender 导过来的 .glb 模型上
- 写出第一个 **屏幕空间后处理** shader：用 `CanvasLayer + ColorRect + canvas_item shader` 给整个 3D 画面加一层扫描线
- 避开 3 个新手 90% 会踩的坑（看不到效果 / 编译报错 / 后处理黑屏）

**前置知识:**

- ✅ Godot 4 场景结构（Node / Scene / MeshInstance3D）→ 见 06-20
- ✅ Godot 3D 物理与角色控制器 → 见 06-20
- ✅ Blender 基础建模 + 导出 .glb → 见 06-13
- ⛔ **不需要** GLSL 经验；本篇从「第一行 shader_type」开始
- ⛔ **不需要** Blender（用 Godot 自带的 PrimitiveMesh 就能学）

**软件版本:** Godot 4.6-stable (2026-01-26)。

---

## 1. 着色器到底是个啥？为什么我已经在 Blender 调过材质了还要写？

先打消一个最常见的误解：**着色器 ≠ 材质**。

```
Blender / Godot 里的「材质」(Material) = 一份「画这张脸该怎么画」的说明书
  ↓
「着色器」(Shader) = 那份说明书里「真正在 GPU 上跑的小程序」
```

也就是说：
- 在 Blender 里你拖颜色、拖粗糙度、连节点——那是在「拼装」这个程序
- 但只要你想做节点拖不出来的事（**让边缘自动亮起来 / 让表面流动 / 让整个画面抖动**），你就要**直接写那个程序**

> 💡 **为什么要这样做？**  
> 大部分游戏里的"霓虹光" / "水波纹" / "全息投影" / "角色受击闪光"，**没有一个是用「标准材质 + 节点」手搓出来的**。它们都是一段不到 30 行的 shader 代码，运行时由 GPU 跑。**学 shader = 学会用代码跟 GPU 对话**，游戏画面表现力的上限会一下子从「中规中矩」升到「哇好酷」。

Godot 把这件事做得很友好：**它用一种叫 GDShader 的 DSL（类 GLSL 但简化了）**。你不需要管顶点缓冲、矩阵变换的底层细节，**像写 GDScript 一样写 shader**。

---

## 2. Godot 4 的 5 种着色器速览

| Shader 类型 | 写在 `shader_type` | 用在哪 | 用途 |
|---|---|---|---|
| `spatial` | `shader_type spatial;` | 3D 网格（MeshInstance3D 的材质） | **3D 模型表面**——本篇重点 |
| `canvas_item` | `shader_type canvas_item;` | 2D 节点 + ColorRect + Sprite2D | **UI / 2D 精灵 / 屏幕后处理** |
| `sky` | `shader_type sky;` | Sky 资源 | 自定义天空盒（动态云、星空） |
| `fog` | `shader_type fog;` | Environment 的 Fog Volume | 体积雾 / 体积光 |
| `particles` | `shader_type particles;` | GPUParticles3D | 粒子的形状/颜色 |

> 💡 **为什么要分这么多种？**  
> 因为它们在 GPU 渲染管线里处在**不同阶段**：sky 在最前画（背景），spatial 画 3D 物体，canvas_item 画在所有 3D 之上（UI / 后处理）。**这就像 Photoshop 里的图层顺序**——你得知道自己的代码会被插在哪一层画。

**今天我们只碰 spatial 和 canvas_item 两个**。

---

## 3. 第一个 Shader：让立方体随时间变色

> 🎯 目标：把一个 MeshInstance3D 立方体挂上「红 → 绿 → 蓝 → 红」循环渐变。

### 3.1 新建场景

1. 打开 Godot 4.6 → Project Manager → New Project（**3D 模板**）
2. 打开新场景：菜单 Scene → New Scene → 选 `Node3D` 根节点 → 点 **Other Node** 加两个子节点：
   - `MeshInstance3D`（命名为 `Box`）
   - `Camera3D`（摆在你看得到 `Box` 的位置，约 `(0, 2, 5)`，旋转 `-30°` 俯视）
   - `DirectionalLight3D`（默认放 `(0, 5, 0)` 即可）
3. 按 F5 运行，**你应该看到一个灰色立方体在浅色天空下**

### 3.2 写第一段 shader

1. 在 `Box` 节点上点 **Mesh → Material → New ShaderMaterial**
2. 双击新建出来的 `ShaderMaterial` 资源，**Inspector 下方会出现一个写着「Shader」的空栏**
3. 点 **Shader → New Shader** → Godot 会打开一个叫 `box_color.gdshader` 的内置编辑器
4. **把下面这段代码整段复制进去：**

```glsl
shader_type spatial;
// 3D 网格才用 spatial，UI / 2D 用 canvas_item

// render_mode 留空，用 Godot 默认的 cull_back（背面剔除）+ 世界空间光照
// 等你熟了再改成 unshaded、blend_mix、depth_draw_opaque 之类的

// 顶点阶段：拿到模型每个顶点的位置和法线
// (vertex() 不写也行，spatial 默认会用模型本身的顶点)
// void vertex() { }

// 片段阶段：每个像素都被这个函数跑一遍，决定它「最后显示什么颜色」
void fragment() {
    // TIME 是 Godot 内建变量：自游戏启动以来的秒数
    float t = TIME;

    // 让红 / 绿 / 蓝 三条通道各自按不同频率震荡，错开
    float r = 0.5 + 0.5 * sin(t * 2.0);                 // 0~1 慢速
    float g = 0.5 + 0.5 * sin(t * 2.0 + 2.094);         // 偏移 2π/3
    float b = 0.5 + 0.5 * sin(t * 2.0 + 4.188);         // 偏移 4π/3

    ALBEDO = vec3(r, g, b);
    // ALBEDO = 物体本来的颜色（漫反射色）
    // 这行是 spatial 着色器里最常用的输出之一
}
```

5. **保存**（Ctrl+S）→ 回到 3D 视口 → **你应该看到立方体在红绿蓝之间循环渐变**

> 💡 **为什么要这样写？**  
> `sin(t * 2.0)` 这一行用的是「正弦波」——它会输出一个永远在 -1~+1 之间平滑波动的数。乘 0.5 加 0.5 把它「挪」到 0~1 范围（颜色通道用的就是 0~1）。三色错开 2π/3 = 120°，**所以三个通道永远有一格是亮、一格是暗**，看起来就是平滑的彩虹色。  
> 这是游戏里写"脉冲"、"呼吸"、"循环变色"最常用的「一行公式」。

### 3.3 常见误解一：shader 改完画面没变化？

**90% 的情况是这 3 个原因之一：**

1. **没把 ShaderMaterial 拖到 Mesh 的 Surface Material Override** — 应该是点 `MeshInstance3D` → Inspector 顶部 `Mesh` 栏旁边那个 **「看起来像空心立方体」的图标** → Material → 选你的 `ShaderMaterial`  
   （或者像 3.2 那样用 `Mesh → Material → New ShaderMaterial` 自动关联）
2. **shader 写完没保存** — `Ctrl+S` 没按，Godot 不会自动保存 .gdshader
3. **GLSL 语法错** — 看右下角 Errors 面板有没有红字。**漏掉 `;`、拼错 `vec3`、忘了 `void fragment()`** 是最常见的

---

## 4. 第二个 Shader：Fresnel 边缘光（赛博朋克发光圈）

> 🎯 目标：让立方体 / 3D 模型的边缘自动发出**亮青色光**，像《赛博朋克 2077》《原神》角色立绘那种「描边发光」效果。

### 4.1 什么是 Fresnel？

Fresnel 是个物理现象：**你斜着看一个物体（比如水球），边缘会反射更多光，正面看则少**。
游戏里把它反过来用：**算「视线方向」和「表面法线」的夹角**——夹角越大（越斜 = 越边缘）→ 越亮。

### 4.2 在刚才的 shader 上加这段

把 `box_color.gdshader` 替换为：

```glsl
shader_type spatial;

uniform vec3 rim_color : source_color = vec3(0.0, 1.0, 1.0);  // 默认青色
uniform float rim_power : hint_range(0.5, 8.0) = 3.0;         // 强度
uniform float rim_strength : hint_range(0.0, 5.0) = 1.5;      // 整体亮度

void fragment() {
    // 基础色：保持一个深蓝紫
    ALBEDO = vec3(0.05, 0.05, 0.15);

    // NORMAL 是「这个像素所在表面的法线」（已经经过法线贴图 / 切线空间）
    // VIEW 是「从这一像素指向相机的方向」
    // 点积 = 1 时完全相对（= 正面看），= 0 时垂直（= 边缘看）
    float n_dot_v = max(dot(NORMAL, VIEW), 0.0);

    // 1 - n_dot_v 让「越斜 = 越大」
    // pow(..., rim_power) 调边缘光的「锐利度」，越大越像一道窄缝
    float fresnel = pow(1.0 - n_dot_v, rim_power);

    // EMISSION = 自发光（不受光照影响）
    EMISSION = rim_color * fresnel * rim_strength;
}
```

保存，**回到 3D 视口**：
- 立方体本应是深蓝紫色
- **边缘应该有明显的青色发光**——把相机绕着它转一圈，边缘一直亮

> 💡 **为什么要用 EMISSION 而不是 ALBEDO？**  
> ALBEDO 是「漫反射色」，会被灯光和阴影「覆盖」——你加了灯光，边缘光可能就看不见了。EMISSION 是「自发光」，**即使场景全黑，边缘也会亮**，是「霓虹」「魔法光」「护盾」的标准做法。

### 4.3 调参小贴士

回到 Inspector 看 ShaderMaterial 顶部：
- `Rim Power` 调到 6.0 → 边缘光变得**很细一条缝**（更有「刀锋」感）
- `Rim Power` 调到 1.5 → 边缘光**糊开成一大片**（更适合魔法球 / 能量罩）
- `Rim Color` 改成 `vec3(1.0, 0.2, 0.8)` → **品红霓虹色**
- `Rim Strength` 拉到 3.0 → 配合 Bloom 后处理（见 06-21 提的 Environment 调 Bloom），**就会真的"溢出"成"过曝"那种感觉**

---

## 5. 把 Shader 挂到 Blender 导过来的角色上

> 🎯 目标：让 06-13 做的那栋低多边形房子、或者 06-23 几何节点做的那片村庄，**也带边缘光**。

如果你本地有 .glb 模型：

1. 把模型拖进 Godot 文件系统面板（推荐放在 `res://models/`）
2. Godot 会自动导入。**关键点**：在 Inspector 顶部勾 **Import → Mesh → Generate Tangent Arrays**（Fresnel 用的 NORMAL 在某些模型上需要这个）
3. 场景里加一个 `MeshInstance3D` → Mesh 选你的 .glb
4. 重复 4.2 的代码，**保存**

> ⚠️ **如果模型分成多个 Surface（比如角色有"身体 / 头发 / 衣服"三个材质）**：  
> 你需要分别给每个 Surface 建 ShaderMaterial。Godot 没有"全局覆盖"一个模型的简单方法——可以写脚本：`for i in mesh.get_surface_count(): mesh.surface_set_material(i, my_shader_mat)`

---

## 6. 屏幕空间后处理：全屏 CRT 扫描线

> 🎯 目标：在整个 3D 画面上叠一层**老式 CRT 显示器**的横线扫描效果 + 边缘暗角 (vignette)。这是「赛博朋克 / 复古 / 恐怖游戏」里几乎必有的滤镜。

### 6.1 原理

不是去改 3D 物体，而是**在 3D 画面画完后再画一张全屏的半透明 ColorRect**——这张 ColorRect 上挂一个 **canvas_item shader**，读取下面 3D 画面的内容 `SCREEN_TEXTURE`，做点像素级计算再画上去。

**这是 Post-Processing（后处理）的核心套路**。

### 6.2 搭场景

回到 3D 场景（根 `Node3D`），新增：

```
- Node3D (主场景)
  - MeshInstance3D  (Box)            ← 上一节的立方体
  - Camera3D
  - DirectionalLight3D
  - CanvasLayer (命名为 PostFX)      ← 注意：不是 Node，是 CanvasLayer
    - ColorRect (命名为 Scanlines)   ← 覆盖整个屏幕
```

`ColorRect` 节点设置：
- **Layout → Anchors Preset** 选 `Full Rect`（Ctrl+Alt+鼠标拖 4 个角也行）
- **Color** 调成白色（不重要，会被 shader 覆盖）
- **Material → New ShaderMaterial** → 配下面这段 shader

新建 `crt.gdshader`：

```glsl
shader_type canvas_item;

// 后处理几乎一定要拿到「下面画了啥」
uniform sampler2D screen_tex : hint_screen_texture, filter_linear_mipmap;

uniform float scanline_strength : hint_range(0.0, 1.0) = 0.25;
uniform float scanline_count = 240.0;          // 老电视 240 条横线
uniform float vignette_strength : hint_range(0.0, 1.0) = 0.5;
uniform float aberration_strength : hint_range(0.0, 0.02) = 0.003;

void fragment() {
    // 1. 屏幕 UV（0~1，从左下到右上）
    vec2 uv = SCREEN_UV;

    // 2. 色差 (Chromatic Aberration): 让 R/G/B 通道稍微错开采样
    //    屏幕越靠边缘错得越厉害（模拟老镜头）
    vec2 dir = uv - 0.5;
    float dist = length(dir);
    float aberration = dist * aberration_strength;
    vec3 col;
    col.r = texture(screen_tex, uv + dir * aberration).r;
    col.g = texture(screen_tex, uv).g;
    col.b = texture(screen_tex, uv - dir * aberration).b;

    // 3. 扫描线：按 Y 坐标 sin 一道道暗线
    float scan = sin(uv.y * scanline_count * 3.14159) * 0.5 + 0.5;
    // 0~1，越靠「暗线」越接近 0
    col *= mix(1.0, scan, scanline_strength);

    // 4. 暗角 (Vignette): 屏幕四周变暗
    float vignette = smoothstep(0.7, 0.2, dist);
    col *= mix(1.0, vignette, vignette_strength);

    COLOR = vec4(col, 1.0);
}
```

按 F5，**整个画面应该蒙上一层 CRT 老电视的复古感**。

> 💡 **为什么要用 CanvasLayer？**  
> `CanvasLayer` 是**不受 Camera 影响的 2D 画层**——它永远覆盖整个屏幕。  
> 普通的 `Node2D / Control` 子节点虽然也能画，但**3D 相机移动时它们会跟着变**。**做全屏后处理用 CanvasLayer 是 Godot 4 的标准做法**。

### 6.3 进阶：把它做成全局开关

给 `Scanlines` ColorRect 加脚本 `post_fx.gd`：

```gdscript
extends ColorRect

func _input(event: InputEvent) -> void:
    if event is InputEventKey and event.pressed and event.keycode == KEY_F2:
        visible = not visible   # 按 F2 切换
        print("PostFX: ", "ON" if visible else "OFF")
```

按 F5 → 游戏中按 F2 → 扫描线消失/出现。**做开发调试 / 玩家设置里加"后处理开关"都用这一招**。

---

## 7. 常见坑点 & 解决方案

### 坑 1: shader 写了"无效果"——右下角没报错但画面没变

**原因 1**：**忘了 `ALBEDO = ...` / `EMISSION = ...` 这类"输出赋值"语句**  
spatial shader 是**白纸**——你不写 ALBEDO 赋值，模型就是 Godot 默认色。  
**解法**：先复制本篇示例，确认能跑，再改。

**原因 2**：**ColorRect 的后处理 shader `hint_screen_texture` 写错**  
必须是 `uniform sampler2D screen_tex : hint_screen_texture, filter_linear_mipmap;` ——**两件事缺一不可**：
- hint_screen_texture（让 Godot 自动注入下面画面的纹理）
- filter_linear_mipmap（缩放时不闪烁）

### 坑 2: 编译报错 "Identifier 'TIME' not declared" / "syntax error"

**原因**：在 `canvas_item` shader 里**写了 `TIME` 但用了错误的内置名**。  
**解决清单**：
- 写 spatial → 用 `TIME` / `NORMAL` / `VIEW` / `ALBEDO` / `EMISSION` / `UV`
- 写 canvas_item → 用 `TIME` / `UV` / `COLOR` / `SCREEN_UV` / `SCREEN_TEXTURE`
- **写 sky → 用 `POSITION` / `EYEDIR` / `COLOR` / `HALF_RES_COLOR`**
- **三种不能混**！混了要么编译错，要么值是空的

### 坑 3: 后处理打开后画面**整体变黑 / 整屏一个颜色**

**原因**：ColorRect 的 `Color` 设成了 (0,0,0,1) 黑色 + shader 没正确写 `COLOR = ...` 输出  
**解法**：
1. **第一步** Color → `Color(1, 1, 1, 1)` 纯白 + shader 里 `COLOR = vec4(1.0, 0.0, 0.0, 1.0);`——应该看到**全屏红**
2. **第二步** 验证完红后，再把 shader 换回正经的 `screen_tex` 采样代码

这是调试 shader 90% 的「二分定位法」。

---

## 8. 延伸阅读

1. **Godot 4 官方着色器文档（spatial）**  
   <https://docs.godotengine.org/en/stable/tutorials/shaders/shader_reference/spatial_shader.html>  
   每一行 built-in（`ALBEDO` / `NORMAL` / `LIGHT_VERTEX`）的精确语义都在这里。

2. **The Book of Shaders（着色器思维训练）**  
   <https://thebookofshaders.com/>  
   跨引擎的 shader 思维入门，**前 5 章读完基本能"读懂"本篇所有数学**。

3. **Godot Shaders 官方示例项目（GitHub）**  
   <https://github.com/godotengine/godot-demo-projects/tree/master/3d/shaders>  
   30+ 个可跑的 spatial / canvas_item / sky shader 范例，**最实用的"开箱即用 shader 库"**。

---

## 9. 今天学了啥 & 下一步

今天你搞定了：
- ✅ **着色器 = 跟 GPU 对话的小程序**，用 GDShader 写比 GLSL 简单 50%
- ✅ **spatial shader** 给 3D 模型加自发光、随时间变色、Fresnel 边缘光
- ✅ **canvas_item + ColorRect** 给整个画面做 CRT 扫描线 / 暗角 / 色差后处理
- ✅ 三种常见坑的「二分定位法」调试套路

**明天预告（06-27）**：06-23 的几何节点 + 今天 shader，**做个"程序化生成的发光水晶森林"**——用几何节点批量造 100 块水晶，每块挂上随时间变化的 Fresnel shader。

> 🎁 **下下下篇预告（07-01+ 综合实战模式）**：把 06-13 的低多边形房子从 Blender 导出 → 在 Godot 里摆场景 → 挂上今天学的 Fresnel 边缘光 → 加上 06-25 的预测 / 同步 → **一个完整的 3D 联机场景**。

---

**软件版本：** Godot 4.6-stable（2026-01-26）/ Blender 4.5 LTS（参考 06-23）。  
**数据来源时间：** 2026-06-26 04:00 (Asia/Shanghai)  
**本篇不依赖 Blender**——所有素材用 Godot 自带 PrimitiveMesh 生成。
