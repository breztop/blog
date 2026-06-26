---
title: 'Blender & Godot 3D 教程 - 2026-06-27 - Blender 场景整理零基础入门:Collections / Outliner / 命名规范 / 资产库四件套'
date: 2026-06-27T04:00:00+08:00
description: '零基础打开 Blender 4.x 场景整理:理解 Outliner(场景大纲)、Collections(集合分组)、命名规范(Object/Mesh/Material/Collection)、资产库(Asset Library)四件套,把一个乱七八糟的默认场景整理成「半年后再打开还能 5 分钟找到任何东西」的工程化项目。'
tags: [blender, godot, multiplayer, 3d, tutorial, scene-management, asset-library]
categories: [game-dev]
draft: false
cover:
  image: /images/posts/blender-godot-2026-06-27.png
  alt: 'Blender 编辑器里 Outliner 面板按 Collections 分层组织,左边一列资产库缩略图,深色专业 UI,场景文件以分层结构展示'
---

![cover](/images/posts/blender-godot-2026-06-27.png)

# Blender 场景整理零基础入门:Collections / Outliner / 命名规范 / 资产库四件套

> 本文写给"模型建得出来,但 .blend 文件半年后打开像屎山"的 Blender 初学者。

## 为什么这篇必须看

很多 Blender 入门教程停在"我会挤出、我会打灯、我会 Cycles 渲染一张图"。然后你兴冲冲做游戏、做动画,做到第 30 个物件时——

- Outliner 里 200 个物体叫 `Cube.001` `Cube.042` `Plane.017`,找一个东西要翻 5 分钟
- 材质球全是 `Material.001` 到 `Material.156`,改个玻璃参数不知道哪个是哪个
- 上次做的"那棵树"想复用,翻遍 30 个 .blend 文件找不到
- 想隐藏 UI 只渲染角色,发现所有东西都堆在 `Collection` 根目录下,得一个一个点眼睛

**这不是建模问题,是工程问题。** Blender 是 DCC 软件里场景管理能力最强的(没有之一),但默认设置故意留白,逼你建立自己的规范。今天这篇把规范讲透。

读完你能做到:
1. 用 Collections 把场景分成可切换的层(角色/环境/灯光/特效)
2. 用命名规范让 6 个月后的自己一眼看懂每个物体
3. 用资产库 (Asset Library) 把常用物件存成"组件",拖出来就用
4. 把项目分成 3 类 .blend 文件:**项目文件** / **资产文件** / **启动模板**

---

## 第一步:认识 Outliner(场景大纲)

打开 Blender 默认场景,在右上角找到这个面板:

```
┌─ Outliner ─────────────────┐
│  ▾ Scene Collection        │
│    ▸ Collection            │
│      Cube                  │
│      Cube.001              │
│      Plane                 │
│      Light                 │
│    ▸ Collection.001        │
│      Camera                │
└─────────────────────────────┘
```

**Outliner = 场景的"目录树"**,跟文件资源管理器一样。任何你在 3D 视图里建的物体,都会在这里出现一行。

### 小贴士:Outliner 才是你真正的"项目浏览器"

3D 视图只适合看空间关系,**找东西 / 改关系 / 批量操作都要在 Outliner 里完成**。养成习惯:每次新建物体先在 Outliner 里看一眼它落在哪个 Collection。

### 5 个 Outliner 关键操作

1. **眼睛图标 👁** :隐藏/显示物体(选中多个按 `H` 也行,但 Outliner 更精确)
2. **箭头图标 ➡** :在 3D 视图里框选该物体(找不到东西时的救命功能)
3. **右键 → Rename** :重命名(不要在物体上 F2,Outliner 里改更可靠)
4. **拖拽** :把物体从一个 Collection 拖到另一个(松手前会高亮目标 Collection)
5. **搜索框** :Outliner 顶部有放大镜,输入 `tree` 立刻过滤所有名字含 tree 的物体

---

## 第二步:用 Collections 把场景分层

**Collection(集合)是 Blender 的"文件夹"**,可以嵌套。它的设计目的只有一个:**让一个场景能拆成可独立开关的逻辑模块**。

### 实战:从默认场景开始整理

**操作 1**:在 Outliner 顶部右键 `Scene Collection` → **New Collection**,命名为 `Hero`(主角)。

**操作 2**:选中场景里那个默认 Cube,鼠标按住拖到 `Hero` 上(等高亮再松手)。这个 Cube 现在属于 `Hero`。

**操作 3**:继续建几个 Collection,目标结构如下:

```
Scene Collection
├── 01_Hero          ← 主角 / 玩家角色
├── 02_Environment   ← 地形 / 建筑 / 天空
├── 03_Props         ← 道具(桌椅 / 树 / 石头)
├── 04_Lighting      ← 所有灯光
├── 05_VFX           ← 粒子 / 特效
└── 06_Render_Only   ← 只用于最终渲染的辅助物件(背景板 / 体积云)
```

### 数字前缀的意义

`01_` `02_` 这种前缀让 Outliner 永远按"工作流顺序"排序。建模 → 环境 → 道具 → 灯光 → 特效 → 渲染辅助,这就是 3D 场景的标准流水线。

### Collection 的 3 个开关

每个 Collection 旁边都有两个图标:

- **眼睛 👁**:3D 视图里显示/隐藏(渲染也会跟着)
- **监视器 📺**:仅渲染时显示,3D 视图不显示(打灯时让场景简洁)
- **箭头 ➡**(最右边的):选中该 Collection 内所有物体

**小贴士**:`04_Lighting` 可以把眼睛关掉(建模时不想被灯光干扰),但渲染前要打开。`06_Render_Only` 反过来:建模时为了找参考可以打开,最终渲染时再用监视器图标单独控制。

### Collection 嵌套

Collection 里可以再建 Collection。比如 `02_Environment` 下可以拆:

```
02_Environment
├── 02.1_Terrain
├── 02.2_Buildings
└── 02.3_Skybox
```

**但不要超过 3 层**——超过就开始反人类了。如果你的场景需要 4 层以上,说明应该拆成多个 .blend 文件了(见第五步)。

---

## 第三步:命名规范

Blender 的默认命名是 `Cube` `Cube.001` `Material` `Material.001`,**这是地狱的开始**。一个好的命名规范要让名字自带信息。

### 4 类核心命名规则

| 类型 | 规则 | 例子 |
|------|------|------|
| **Object(物体)** | `<类别>_<具体名>` | `Hero_Knight` `Prop_Tree_Oak` `Light_Key_Fill` |
| **Mesh(网格数据)** | `<物体名>.<部件>_Geo` | `Hero_Knight.Body_Geo` `Hero_Knight.Sword_Geo` |
| **Material(材质)** | `<物体名>_<材质类型>` | `Hero_Knight.Armor_Metal` `Prop_Tree.Bark_Rough` |
| **Collection** | `<流水号>_<类别>` | `01_Hero` `02_Environment` `03_Props` |

### 实战:把默认 Cube 改名为 `Hero_Knight`

**操作 1**:在 Outliner 右键 `Cube` → Rename → 输入 `Hero_Knight`。

**操作 2**:注意 Mesh 数据块(Outliner 右上角切换到第二个图标"Blender File"视图才能看到),原本叫 `Cube`,现在跟着物体变成 `Hero_Knight.Body_Geo`(Blender 4.x 自动用物体名加 `.001` 后缀,你需要手动重命名)。

**操作 3**:材质同理——选中物体,Material 标签页下把 `Material` 改名为 `Hero_Knight.Armor_Metal`。

### 大驼峰 vs 下划线,选一个并坚持

两种风格都可以,关键是**整个项目只用一个**:
- 大驼峰 `HeroKnight`(读起来像英文,但键盘要按 Shift)
- 下划线 `hero_knight`(输入快,但读起来不连贯)

我个人推荐 **下划线 + 全小写 + 下划线分隔语义**,理由:
1. Godot 的节点命名规范就是 `snake_case`,跨软件迁移不用再改
2. 终端 / 路径里不用处理大小写
3. 搜索 `knight` 能一次性匹配所有相关物件

### 命名规范的 3 条铁律

1. **禁用空格**:`Hero Knight` 是给自己挖坑——脚本处理、URL 编码、命令行都会出事
2. **禁用中文**:Blender 支持但下游工具(Godot、Unity、Substance)经常乱码
3. **不用 `.001` 后缀做"版本"**:`Cube.001` 是 Blender 复制的标记,不是你的版本号。版本管理用文件名 `Hero_v01.blend` `Hero_v02.blend`

---

## 第四步:资产库(Asset Library)——Blender 4.x 的杀手锏

这是 Blender 3.x 引入、4.x 大幅增强的功能:**把常用物体拖进一个特殊 .blend 文件,以后在任何项目里都能像调色板一样拖出来用**。

### 实战:把 `Hero_Knight` 存进资产库

**操作 1**:新建一个 .blend 文件,保存为 `~/blender_assets/characters.blend`(资产文件,不是项目文件)。

**操作 2**:把 `Hero_Knight`(包含模型、材质、动画)复制到这个文件里。

**操作 3**:在 Outliner 右键 `Hero_Knight` → **Mark as Asset**(标记为资产)。物体前会出现一个 🟢 绿色书签图标。

**操作 4**:**Edit → Preferences → File Paths → Asset Libraries** → 加上面的文件夹路径 `~/blender_assets/`。

**操作 5**:回到项目文件,右上角编辑器类型切换到 **Asset Browser**(资产浏览器,一个文件夹图标),左侧会列出 `characters.blend` 里所有标记为 Asset 的物件。直接把 `Hero_Knight` 拖进 3D 视图——实例化完成。

### 资产 vs 链接(Link)的区别

- **Asset(Append)**:把物体**复制**到当前 .blend,改一个不影响另一个。适合需要本地修改的物件。
- **Linked Library(Link)**:物体**引用**自外部 .blend,改原始文件所有项目同步更新。适合"全局唯一的角色"(主角 Player、UI 图标)。

**操作**:Asset Browser 里每个资产右上角有个数字角标,**点它切换 Append / Link 模式**。Link 模式下,原始文件改了拖一个新版本过来,所有引用自动更新。

### 小贴士:资产库不是"素材市场"

网上那些 `blend swap` 上的模型是给非商业项目参考的,**自己项目里做出来的高复用物件**才是资产库的价值所在。一个 5 人小团队有个 200 个内部资产的资产库,产能可以翻倍。

### 资产文件的组织结构

```
~/blender_assets/
├── characters.blend       ← 所有角色
├── environment.blend      ← 所有环境(树、石头、建筑模块)
├── props.blend            ← 所有道具
├── materials.blend        ← 所有"纯材质"球(木材、金属、玻璃等)
└── hdri.blend             ← 所有 HDRI 环境贴图
```

每个 .blend 文件内部的 Collection 结构按上面的命名规范走。

---

## 第五步:三类 .blend 文件的工程化拆分

大多数初学者把一个 .blend 文件从建模用到渲染,文件膨胀到 500MB+ 还卡。这是错的。

### 推荐的项目结构

```
~/projects/my_game/
├── project.blend                  ← 当前主项目(只有引用,几乎不含模型)
├── project_backup_v01.blend       ← 备份
├── assets/                        ← 资产文件(每个类目一个 .blend)
│   ├── characters.blend
│   ├── environment.blend
│   └── props.blend
└── renders/                       ← 输出图/视频
```

### 实战:用 Link 模式搭建一个"小屋"场景

**操作 1**:打开 `project.blend`,在 Asset Browser 里从 `environment.blend` 拖一棵 `Tree_Oak`(Link 模式)进 `02_Environment` Collection。

**操作 2**:复制粘贴 5 次(用 `Shift+D` 不是 `Ctrl+C`),让场景里有 6 棵树。**它们都是同一个原始物件的引用**——改原始文件的树形状,场景里 6 棵全部跟着变。

**操作 3**:再从 `props.blend` 拖一张 `Table_Wood` 进 `03_Props` Collection。Append 模式(因为这张桌子你可能要在项目里调大小)。

### 启动模板 .blend

最后,做一个 `template.blend` 保存你所有常用设置(渲染引擎偏好、单位、快捷键、Output 路径),每个新项目用 `File → New from template` 启动。

---

## 常见坑点 + 解决方案

### 坑 1:`Cube.001` 满天飞,删一个物体另一个消失

**原因**:Blender 里 Object(物体)和 Mesh(网格数据)是分开的。删除物体时如果选了 "Orphan Data",网格数据还在;但如果两个物体共享同一份网格数据,删一个会带掉另一个。

**解决**:
1. Outliner 切换到 **Blender File 视图**(右上角第二个图标),这是"数据块浏览器"
2. 检查哪些 Mesh 数据被多少 Object 引用——数字 > 1 就是被共享
3. 选中物体 → `F3` 搜索 `Make Single User` → Object & Data & Material,把物体变成完全独立

### 坑 2:命名改了,但 Outliner 里还是旧名字

**原因**:Blender 的命名缓存有时候没刷新,特别是刚从其他软件导入 FBX/OBJ。

**解决**:
1. 选中物体 → `F2` 直接在 3D 视图里改名(不依赖 Outliner 缓存)
2. 或者 `File → Save` 后 `File → Revert`,强制重载

### 坑 3:资产库拖出来是 Link 模式,改不了材质

**原因**:Link 模式下原始文件是只读的,你不能直接改。

**解决**:
1. 在 Project 视图里选中那个 Link 进来的物体
2. `F3` → `Make Local` → 选择 `Selected Objects` 或 `All`,把引用变成本地副本
3. 改完存盘。注意:**改完后这个物件就和原始资产脱钩了**,原始文件再更新也不会同步过来。

---

## 实战 Demo:30 分钟搭一个"可维护的小屋场景"

下面给个完整的最小工作流,你可以直接照着做:

### Step 1(2 分钟):建项目结构
- 桌面建文件夹 `~/projects/cabin/`
- 新建 `cabin.blend`,保存进去
- Outliner 里建 `01_Cabin` `02_Environment` `03_Lighting` `04_Render_Only` 四个 Collection

### Step 2(10 分钟):在 `01_Cabin` 里建模
- `Shift+A → Mesh → Cube` 改名为 `Cabin_Wall`,缩放 `(4, 0.2, 3)`,位置 `(0, 0, 1.5)`
- `Shift+A → Mesh → Cube` 改名为 `Cabin_Roof`,缩放 `(4.5, 0.2, 3)`,旋转 `45°` Z 轴,位置 `(0, 0, 3)`
- `Shift+A → Mesh → Cube` 改名为 `Cabin_Door`,缩放 `(0.8, 0.1, 1.6)`,位置 `(0, 2, 0.8)`
- Mesh 数据全部按命名规则重命名,例如 `Cabin_Wall.Wall_Geo`

### Step 3(5 分钟):造环境
- 在 `02_Environment` 里 `Shift+A → Mesh → Plane` 改名为 `Terrain_Ground`,缩放 `(10, 10, 1)`
- 复制 3 棵树(用 Asset Library 拖,Append 模式)改名为 `Tree_Oak_01` 到 `Tree_Oak_03`

### Step 4(5 分钟):打灯
- `04_Lighting` 里 `Shift+A → Light → Point`,改名为 `Light_Key`(主光),能量 1000W,位置 `(4, -3, 5)`
- 再来一盏 `Light_Fill`(补光),能量 200W,位置 `(-4, -3, 2)`
- HDRI 的话用 `World Properties → Surface → Environment Texture`,选一张免费的 Poly Haven HDRI

### Step 5(8 分钟):打材质并保存资产
- `Cabin_Wall` 材质改名为 `Cabin_Wall.Wood_Mat`,Base Color 选棕色,糙度 0.8
- `Cabin_Roof` 材质改名为 `Cabin_Roof.Shingle_Mat`,Base Color 选深红
- `File → Save As`,另存为 `assets/environment.blend`,把 `01_Cabin` 整个 Collection 右键 **Mark as Asset**

### 验证
- `File → New`(新项目)→ Asset Browser → 从 `environment.blend` 拖 `01_Cabin` 进新场景
- 你应该看到刚才的小屋完整出现在新场景里,材质灯光全对

---

## 延伸阅读

1. **Blender 官方手册 — Asset Library**: <https://docs.blender.org/manual/en/latest/files/asset_libraries.html> — 最权威的资产库配置文档,看完能解决 90% 的 Link 问题
2. **Blender Studio — Project Organization**: <https://studio.blender.org/pipeline/> — Blender 官方动画工作室的实际项目组织流程,看完能理解"为什么这么设计"
3. **CG Cookie — Scene Management in Blender**: <https://cgcookie.com/articles/blender-tips-how-to-organize-your-scene> — 老牌 Blender 教程站,Outliner 和 Collections 的实战演示特别清楚

---

## 软件版本与数据来源

- **Blender 版本**:Blender 4.2 LTS(本文操作在 4.x 全系列通用,资产库功能从 3.0 开始引入,4.x 增强最大)
- **数据采集时间**:2026-06-27 04:00 (Asia/Shanghai)
- **配套阅读**:昨天的 Godot 着色器教程(2026-06-26)和前天的客户端预测教程(2026-06-25)
