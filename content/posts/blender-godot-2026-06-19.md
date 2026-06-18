---
title: "Blender & Godot 3D 教程 - 2026-06-19 - Blender 灯光与渲染零基础入门:HDRI + 三点光 + Cycles vs Eevee 一次讲清"
date: 2026-06-19T04:00:00+08:00
description: "零基础打开 Blender 4.x 灯光与渲染:理解 HDRI 环境光、掌握三点照明经典布光(Key/Fill/Rim)、弄清 Cycles(真实慢) vs Eevee(实时快)的差别,把 6.13+6.17 的房子从「灰扑扑」照成「黄昏小屋」并渲出一张图。"
tags: [blender, godot, multiplayer, 3d, tutorial]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-19.jpg
  alt: "Blender 4.x 中低多边形房子被三点光照亮,HDRI 天空环境背景,电影感光影"
  relative: false
---

![cover](/images/posts/blender-godot-2026-06-19.jpg)

> 系列第八篇。前两篇我们给低多边形房子上完了 PBR 材质(6.17) + 在 Godot 里摆到了带太阳光的场景里(6.18)。今天回头看 Blender 这一头——**材质再好,没有光也是黑的**。这一篇只做一件事:把 6.13+6.17 的房子从「灰扑扑带材质的塑料块」照成「黄昏下的金色小屋」,顺手把游戏圈天天挂在嘴边的 **HDRI / 三点照明 / Cycles vs Eevee** 一次讲清。学完你渲一张图能发朋友圈,也理解 Godot 里那盏 `DirectionalLight3D` 到底在干嘛。

## 0. 本篇目标 & 前置

**学完你能做什么:**
- 理解 PBR 为什么**必须靠光**才能呈现
- 分清 4 种光源(点光 / 太阳光 / 聚光 / 面光)的差别
- 下载并挂上一张 HDRI 环境贴图,让模型立刻有"世界感"
- 用"三点照明"经典布光法把房子照得立体好看
- 在 Cycles 和 Eevee 两个渲染引擎之间切换并解释差异
- 按 F12 渲出一张图、存成 PNG
- 理解 Blender 的灯和 Godot 的灯**不是同一回事**(导出时不要依赖)

**前置:**
- Blender 4.x 已装好(6.12 讲过)
- 你手头有一个简单模型:跟着 6.13 建一个低多边形房子,或者用默认 Cube / Monkey 练手
- 跟着 6.17 至少给它上过 Base Color / Roughness(没上也行,光对默认灰材质也起作用)
- **不要求**任何美术 / 摄影基础

---

## 1. 为什么材质还不够?——光的物理直觉

先做一个**反直觉**的实验。

打开 Blender,载入你那个房子模型(或者默认 Cube)。**关掉一切光源**:

1. 视图右上角 shading 切到 **Rendered**(按 Z 选 Rendered)
2. 右下角属性面板找到 **绿色小灯泡图标**(World Properties / 世界属性)
3. 把 **Surface** 下拉框从 `Background` 改成 `Environment Texture`,但先不挂任何图片
4. 你会看到模型**几乎全黑**

为什么?因为 PBR 的「Base Color」其实是「这束光**反射回来**的颜色」——没有光进来,就什么颜色都不会出来。再红的红瓦,关灯之后也只是黑色塑料。

> 💡 **为什么要这样做**:理解了这一点,你就会明白游戏引擎里那些「好看的 3D 场景」**70% 的功劳在光,30% 在材质和模型**。一张灰模型 + 好灯光 >> 一张精雕细琢的模型 + 黑灯瞎火。

### 1.1 一句话心法

> **PBR 材质是「这个物体对光会怎么反应」,光才是「让反应发生」的能量。**

---

## 2. 三分钟认识 4 种光源

Blender 里有 4 种基础光源,先混个脸熟,后面只会用到其中 2 种。

| 光源 | 长什么样 | 用途 | 强度衰减 |
|---|---|---|---|
| **Point Light**(点光) | 像一颗小灯泡,向四面八方均匀发光 | 灯泡、烛光、篝火、魔法光球 | 有(距离平方反比) |
| **Sun Light**(太阳光) | 太阳,所有光线**互相平行** | 户外阳光、月光 | 无(无限远) |
| **Spot Light**(聚光) | 手电筒 / 舞台聚光灯,有锥角 | 车灯、路灯、舞台光、手电 | 有(距离平方反比) |
| **Area Light**(面光) | 一块发光面板(柔光箱) | 摄影棚补光、窗户透光 | 有 |

**为什么没 LED / 体积光 / IES 光源?** 都有,但 90% 的场景这 4 种就够,新手先把"够用版"打牢。

### 2.1 怎么加一盏灯

1. 视图左上角菜单 **Add → Light → Point**(或者 Sun / Spot / Area)
2. 视图中立刻出现一个光源图标 + 一根黄色虚线(能量强度示意)
3. 右下角属性面板找到**黄色小灯泡图标**(Object Data Properties / 物体数据属性)
4. 调 **Power**(能量,单位 W):默认 1000W 通常太亮,先从 50W 开始试
5. 调 **Color**(颜色):点色块,选暖黄 / 冷白 / 角色光颜色

> 💡 **为什么要这样做**:能量单位 W 看着奇怪,是因为 Blender 4.x 改用了物理正确的「瓦特」,不再是玄学的「Multiplier 1.0」。新手只要记住**户外太阳约 100~1000W,室内灯泡 5~50W**就够了。

---

## 3. HDRI 环境光:让模型"有世界"

打开 6.17 那个房子,你会发现即使加了一盏灯,**没被照到的一面还是黑的**,而且整个画面没有"氛围感"——像是 PS 里抠出来的孤立物体。

解决办法:**HDRI 环境光**。

### 3.1 HDRI 是什么

**HDRI = High Dynamic Range Image**(高动态范围图像)。普通 JPG 是 0~255 亮度,HDRI 能记录**从太阳到阴影的全部真实亮度范围**。拿它做环境,Blender 会同时拿到两样东西:

1. **环境光**:全方向照亮模型,补足阴面
2. **反射**:金属、玻璃、湿润表面会反射出 HDRI 里的天空 / 太阳

所以**加了 HDRI,模型立刻就有"它属于这个世界"的感觉**——阴面不再是死黑,而是带蓝天的反射光。

### 3.2 去哪下 HDRI(全免费)

**Poly Haven**(https://polyhaven.com/hdris)——CC0 协议,商用免费,几百张高质量 HDRI。

推荐下载:
- `kloofendal_43d_clear_puresky_1k.exr`(纯净蓝天)
- `qwantani_puresky_1k.exr`(黄昏金天)
- `studio_small_03_1k.exr`(摄影棚柔光,产品图常用)

> 💡 **为什么要这样做**:CC0 = 你用、用、改、商用都不用署名,也不用担心版权信。1k 分辨率对大多数项目够了,4k 慢且占显存,新手不要上 8k。

### 3.3 挂上 HDRI(5 步)

1. 顶部菜单 **Editor Type → Image Editor**(或者直接在 Shader Editor 里也行)
2. **Open** → 选刚才下的 `.exr` / `.hdr` 文件
3. 回到 3D 视图,右下角 **World Properties**(绿色地球图标)
4. **Surface** 下拉选 `Environment Texture`
5. 点击 **Color** 下方的小圆圈按钮 → **Image Texture** → 选刚才打开的图

视图里的房子**立刻活了**——阴面有了天空蓝的反光,整体有了"室外"氛围。

> 💡 **为什么要这样做**:Environment Texture 比单纯的 Background 多了**「作为光源参与计算」**的能力。这是 PBR 引擎都有的标准做法,Blender / Unreal / Godot 全通用。

### 3.4 调 HDRI 强度

**Strength**(强度)默认 1.0。如果场景太亮或太暗,在这里调:

- 室内场景:0.3 ~ 0.8
- 户外白天:0.8 ~ 1.5
- 黄昏 / 夜景:0.5 ~ 1.0

---

## 4. 三点照明:让模型"立体"起来的经典布光法

只挂 HDRI 不够——HDRI 给的是"全局氛围",但**主体物通常需要额外的灯来"塑形"**。

摄影 / 电影 / 3D 圈用了几百年的**三点照明**:

```
        Rim / 轮廓光(后方,逆光)
            |
            v
         [模型]
        /       \
       /         \
   Key 主光     Fill 辅光
   (左前 45°)   (右前 30°,弱)
```

### 4.1 Key Light(主光)—— 决定形状

- **位置**:模型**斜前方 45°**,略高于模型
- **作用**:照亮主体,投出**主阴影**,决定"光源从哪来"
- **强度**:三点里**最强**,占整体 60~80%
- **类型**:通常用 **Sun Light**(户外)或 **Area Light**(室内)

实操:
1. Add → Light → Sun
2. 在视图中按 **R**(旋转)把它转到模型左前方斜上方
3. Power 设 **5 W**(太阳光 5W 在 Blender 4.x 就很亮,别用 1000)
4. 按 **N** 打开右栏 Transform 面板,检查 Rotation X 约 -30° ~ -45°、Y 约 -45°

### 4.2 Fill Light(辅光)—— 减阴影

- **位置**:模型**另一侧斜前方**,与 Key 大致对称,但**更低、更远**
- **作用**:把 Key 投出的死黑阴影"抬"一点,让细节可见
- **强度**:Key 的 **30~50%**(常用低功率灯 + 大尺寸补光)
- **类型**:**Area Light** 最好(柔光),Point 也行

实操:
1. Add → Light → Area
2. 移到模型右前方,略低
3. Power 设 **100 W** 左右(面光衰减比点光慢,数值会大)
4. 旋转让它朝向模型

### 4.3 Rim Light / Back Light(轮廓光)—— 抠轮廓

- **位置**:模型**正后方偏上**,朝镜头方向打
- **作用**:在模型边缘勾出一道亮边,**把主体从背景里"抠"出来**
- **强度**:Key 的 50%~80%
- **类型**:**Sun** 或 **Spot**(更聚焦)

实操:
1. Add → Light → Spot
2. 移到模型背后,略高,指向相机方向
3. Power 设 **200 W**
4. Spot Cone:Angle 设 60° 左右

### 4.4 一张图复习

按好三盏灯之后,从相机视角看应该是这样的能量分布:

| 区域 | 亮度来源 | 占比 |
|---|---|---|
| **受光面**(Key 那一侧) | Key 主光 + HDRI | ~60% |
| **阴面**(Key 反面) | Fill 辅光 + HDRI 反射 | ~25% |
| **轮廓边**(Rim 那一侧) | Rim 轮廓光 | ~15% |
| **整体环境** | HDRI | 抬底,不主导 |

> 💡 **为什么要这样做**:这是摄影师、3D 艺术家**几百年不变**的经验公式。任何主体物——人像、产品、房子、汽车——都用这套逻辑布光。学会了就是一辈子的技能,换了 Maya / Houdini / Unreal 也照样用。

---

## 5. Cycles vs Eevee:游戏圈天天挂嘴边的两个词

切到 Rendered 视图(按 Z)看实时效果的时候,Blender 默认用 **Eevee**(实时引擎)。按 **F12** 渲图的时候,默认用 **Cycles**(离线引擎)。这俩是啥?

### 5.1 Eevee:实时、卡通、像游戏引擎

- **原理**:近似 / 模拟光线,牺牲精度换速度
- **速度**:**非常快**,60 fps 流畅
- **适合**:游戏引擎内的预览、动画、低多边形风格作品、风格化渲染
- **缺陷**:反射、折射、软阴影、玻璃、置换等高级效果**要么不准,要么需要手动补**
- **底层**:和 Godot / Unreal / Unity 的实时管线**基本同源**

### 5.2 Cycles:真实、慢、像电影 / 产品图

- **原理**:**真实路径追踪**(Path Tracing)——模拟一束光从相机出发,在场景里弹来弹去,根据物理反弹最终算出颜色
- **速度**:**非常慢**,一张图可能渲几分钟到几小时
- **适合**:产品图、建筑可视化、电影、追求"照片级真实"的静态图
- **优点**:反射、折射、焦散、玻璃、置换全部自动正确
- **缺陷**:没法实时,游戏跑不动

### 5.3 怎么切

右下角属性面板**摄像机图标**(Render Properties):
- **Render Engine** 下拉框:`Eevee` 或 `Cycles`

切换之后**别忘了调采样**:
- **Eevee → Sampling → Render**:64 ~ 256 够用
- **Cycles → Sampling → Viewport**(实时预览):32 ~ 128
- **Cycles → Sampling → Render**(最终出图):256 ~ 4096

> 💡 **为什么要这样做**:**玩 Godot 选 Eevee 出图**(快、风格化、容易上手);**做产品图 / 概念图选 Cycles**(慢但真实)。学会切换,你就知道为什么游戏引擎"长那样"——它就是 Eevee 思路。

### 5.4 一句话对比

> **Eevee 是"猜",Cycles 是"算"。猜得快但偶尔翻车,算得慢但永远对。**

---

## 6. 渲一张图:从按 F12 到保存 PNG

布完光、调完材质、切到 Cycles 之后:

### 6.1 渲图步骤

1. 视图按 **N** 打开右栏,检查相机位置(`Camera` 物体)在不在合适角度(按 **0** 切换到相机视角)
2. 确认右下角 **Render Properties → Render Engine** 是 **Cycles**
3. **Output Properties**(纸张图标)→ **File Format** 选 **PNG**,**Color** 选 **RGBA**(透明背景)或 **RGB**
4. **Output Properties → Output → File Path** 改成你想保存的位置,比如 `/tmp/render/`
5. 顶部菜单 **Render → Render Image**(快捷键 **F12**)
6. 等待采样跑完(左下角进度条)
7. 渲完直接在 Image Editor 里 **Image → Save As → 存 PNG**

> 💡 **为什么要这样做**:F12 是单帧,如果你想要动画用 Ctrl+F12 渲序列,或者 Render → Render Animation。但新手第一张图先玩 F12。

### 6.2 加快 Cycles 渲染的 4 个开关

如果渲图太慢:

1. **Sampling → Render** 降到 128(肉眼看几乎一样)
2. **Light Paths → Max Bounces** Total 从 12 降到 4~6(玻璃 / 金属多的场景才需要高 bounce)
3. **Performance → Use GPU Compute**(如果有 NVIDIA / AMD 显卡,GPU 比 CPU 快 5~20 倍)
4. 切回 **Eevee** —— 这是最快的路

---

## 7. 导出 `.glb` 时要不要把光带去 Godot?

**不带。** 这是新人 100% 会踩的坑。

### 7.1 Blender 的光 ≠ Godot 的光

- **Blender 的光**是「场景属性」的一部分,只用于 Blender 内部渲染
- **Godot 的光**(`DirectionalLight3D` / `OmniLight3D` / `SpotLight3D`)是场景里的节点,只属于 Godot

**导出 `.glb` 的时候,Blender 的光不会跟着去 Godot**。Godot 拿到的是几何 + 材质,你需要在 Godot 里**重新摆灯**(参考 6.18 那盏 DirectionalLight3D)。

### 7.2 那为什么还要在 Blender 里布光?

因为:
- **预览 / 出图**:你在 Blender 里渲的概念图要好看
- **设计参考**:布完光的角度、颜色、强度,你可以照搬到 Godot 场景里——只是换工具重摆而已
- **跨团队沟通**:把渲染图发给美术 / 策划看,大家能对齐"游戏里这个场景大概长这样"

> 💡 **为什么要这样做**:工具之间的边界感是职业素养。一个 3D 美术不会抱怨"为啥 Godot 没把我的灯带过来",他们知道 .glb 只装几何 + 材质。

---

## 8. 常见坑点(3 个)

### 坑 1:模型全黑 / 全白 / 完全没颜色

**症状**:明明挂了 HDRI + 三盏灯,模型还是一片漆黑,或者白得发光。

**原因**:
- HDRI **没挂上**(检查 World Properties → Surface 是不是 Environment Texture,而不是 Background)
- Strength 调到 0 或者一个极小值
- Render Engine 切到了 Eevee 但 Cycles-only 的材质 / 节点在工作
- 模型材质 Base Color 误设为 0,0,0(纯黑)—— 它就**真**是黑的

**解法**:
1. 把 Strength 临时调 5.0 看模型有没有反应,有就是数值问题
2. World Properties 检查 Surface 类型
3. 物体材质 → Base Color 临时设纯红,排除材质问题

### 坑 2:阴影全是锯齿 / 颗粒感很重

**症状**:Cycles 渲出来的图阴影边缘像狗啃的,或者有密密麻麻的斑点。

**原因**:
- **采样不够**:Render Samples 默认 4096 是为高质量出的;实时渲染 128 就够,最终 512~1024 也行
- **光源半径太小**:Sun / Point 默认半径 0,等于是"完美点光源",阴影会非常硬
- **Light Path 反弹太低**:玻璃 / 金属场景 Max Bounces 不够

**解法**:
1. **Light Properties → Light → Radius** 调到 0.5 ~ 2.0,阴影立刻柔
2. **Sampling → Render** 提到 512 重渲
3. **Light Paths → Clamping → Direct / Indirect** 调 0(去掉钳制,可能慢但干净)
4. 切 **Eevee + Shadow → Soft Shadow**,半径调大,也是柔和阴影

### 坑 3:渲一张图要等 10 分钟

**症状**:按 F12 之后 Blender 卡死,半天没反应。

**原因**:你大概率在用 CPU 跑 Cycles,而且采样 4096 + 反弹 12 + 复杂材质。

**解法**:
1. **Performance → Device → GPU Compute**(有 NVIDIA / AMD 显卡的话)—— 提速 5~20 倍
2. **Sampling → Render** 降到 256(肉眼差别小)
3. **Light Paths → Max Bounces → Total** 降到 4
4. **Viewport → 切 Eevee**(实时引擎,1 秒出图)
5. **Denoise 开启**:Sampling → Viewport / Render 下面勾 **Denoise**(OptiX / OpenImageDenoise),噪点瞬间消失

---

## 9. 30 秒复习路线图

如果只看一段:

> **光 = 让 PBR 材质"有颜色"的能量源**
> **HDRI = 全局环境光,让模型"有世界"**
> **三点照明 = Key + Fill + Rim,让主体"立体好看"**
> **Eevee = 游戏实时思路(快);Cycles = 离线真实思路(慢)**
> **Godot 里重摆灯,Blender 的灯带不过去**

---

## 10. 延伸阅读

1. **[Blender Manual - Lights](https://docs.blender.org/manual/en/latest/render/lights/index.html)** —— 官方灯光手册,4 种光源所有参数都讲全
2. **[Poly Haven HDRI 库](https://polyhaven.com/hdris)** —— CC0 高质量 HDR 环境贴图,免费商用
3. **[Blender Guru - 3 Point Lighting Tutorial](https://www.blenderguru.com/tutorials/lighting-techniques-blender)** —— 经典摄影式布光在 3D 里的实践(英文,YouTube 配套视频)

---

## 📌 软件版本 & 数据来源

- **Blender**:4.x(本文测试基于 4.2 LTS,4.0+ 均适用)
- **HDRI 下载**:Poly Haven(2026-06 抓取,CC0 协议)
- **数据来源时间**:2026-06-19

---

> 下期预告:系列第九篇——回到 Godot 4 一侧,把 6.18 那盏太阳光拆开看:**`DirectionalLight3D` 的 Energy / Color / Shadow 三件套**怎么对应到 Blender 今天的 Key/Fill/Rim,以及怎么用 Godot 4.6 新出的 **Volumetric Fog** 给场景加一层电影感的雾。**[Blender & Godot 3D 教程 - 2026-06-20 - Godot 4 DirectionalLight3D + Volumetric Fog:把你的 3D 场景打出电影感]**