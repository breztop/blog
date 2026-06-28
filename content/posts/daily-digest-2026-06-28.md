---
title: '每日开源速递 - 2026-06-28'
date: 2026-06-28T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026年6月28日

今日 GitHub Trending 出现一个有意思的信号：**个人云 + 桌面增强 + Agent 工具链** 三条线同时冲榜。`IceWhaleTech/CasaOS` 用 "100 美元以下打造个人云" 的口号重新激活了 NAS 圈；`microsoft/PowerToys` 作为 Windows 桌面增强的事实标准再次被 AI 工作流带火；`hugohe3/ppt-master` 则用"真可编辑 .pptx"精准切入 AI 演示工具最大的痛点；`garrytan/gstack`（YC 现任 CEO 亲推）把 "Claude Code 团队化" 做成 23 个 slash 命令；持续高热的还有 `simplex-chat`、`xbtlin/ai-berkshire`、`commaai/openpilot`、`google-labs-code/design.md`、`ripienaar/free-for-dev`、`JCodesMore/ai-website-cloner-template`。下面逐个深入。

> 数据来源：[github.com/trending](https://github.com/trending)（每日 UTC 榜单）
> 抓取时间：2026-06-28 13:02 (Asia/Shanghai) / 2026-06-28 05:02 (UTC)

---

## 1. simplex-chat/simplex-chat — 第一款"无用户标识符"的隐私通讯网络（连续两日霸榜）

### 核心功能

`SimpleX Chat` 是当前唯一一个**在协议层完全去用户标识符**的开源即时通讯网络：没有手机号、没有用户名、没有随机 ID、没有密钥可见的 userID——既不在消息中也不在元数据里。它基于 **Double Ratchet** 算法做端到端加密，并额外加一层 per-message 独立密钥协商。即便是服务器被攻破，攻击者也只能拿到无法关联到任何"用户"的密文块。

平台覆盖极其完整：

- **iOS / Android** 移动 App（App Store + Google Play + APK）
- **macOS / Windows / Linux** 桌面客户端
- **Haskell 编写的 CLI**（开发者首选）
- **TestFlight** 预览通道
- **可自托管的 SMP 服务器**（Docker 一行启动）
- **SimpleX Directory** 群组目录 + 一次性 SimpleX Address

### 适用场景

- 记者、律师、医生等高敏职业通讯
- 自托管 SMP 后做企业内部高保密对话
- 跨境通讯 + 规避审查/封锁
- 厌倦"绑定手机号"的所有人

### 安装方式

```bash
# 移动端：App Store / Google Play 搜 "SimpleX Chat"
# 桌面端：从 GitHub Releases 下载 .dmg / .exe / .AppImage

# 终端版（开发者首选）
curl -fsSL https://raw.githubusercontent.com/simplex-chat/simplex-chat/stable/scripts/cmd-zap.sh | sh

# 自托管 SMP 服务器
docker run -d --name smp-server -p 5223:5223 simplexchat/smp-server
```

### 代码示例

```text
# A 用户的终端
$ simplex-chat
> /connect
# 输出：https://smp4.simplex.im/a#<一次性密钥串>

# 把链接发给 B
# B 的终端粘贴
> /connect https://smp4.simplex.im/a#<一次性密钥串>
# 连接建立，所有消息走 E2E 加密
> /help
```

### 评价

**优点**：

- 唯一真正做到"协议层零用户标识符"的成熟方案（Signal/Matrix/Wire 仍有某种 ID）
- 客户端 + 服务器 + 协议全部开源，可独立审计
- 双层 E2E 加密 + 每条消息独立密钥
- 平台覆盖广，CLI + 桌面 + 移动 + Web 五端齐全

**缺点**：

- 用户基数仍远小于 Signal/Telegram，"朋友也在用"是当前最大门槛
- Haskell 实现的自托管对中小团队仍有门槛
- "完全隐私" = "无找回机制"

**推荐指数**：⭐⭐⭐⭐⭐（认真在乎元数据隐私的唯一成熟开源选择）

---

## 2. xbtlin/ai-berkshire — AI 时代的伯克希尔：四大师方法论 + 多 Agent 对抗（昨日第 10 → 今日第 2）

### 核心功能

`ai-berkshire` 是一个**把"价值投资四大师方法论"做成 Claude Code / Codex 可执行 Skill** 的研究框架。它把巴菲特、芒格、段永平、李录四位大师的核心方法论拆解成结构化 prompt + 工作流，让多个 Agent **并行**做研究并互相挑战。

- **方法论即代码**：每位大师的护城河、估值原则、决策模型都有独立 Skill
- **多 Agent 对抗**：让多套方法论同时跑一份研报，输出"共识"与"分歧"
- **Claude Code / Codex 双兼容**：直接放进 AI IDE 就能用

### 适用场景

- A 股 / 港股 / 美股长期价值投资者的研究提速
- AI 投研教学（把"四大师"具象化为可操作 prompt）
- 卖方/买方研究员做观点交叉验证
- 个人投资者做"自上而下 + 自下而上"快速框架搭建

### 安装方式

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
# 在 Claude Code / Codex 中打开，skills/ 目录即为可调用 Skill
claude  # 或 codex
```

### 代码示例

```text
# 让"芒格 + 巴菲特"双 Agent 跑同一份财报
> /buffett-analyze 600519.SH --vs-methodology=buffett,munger
# 输出：
# - 共识：强护城河、ROE 持续 > 20%、自由现金流稳健
# - 分歧：芒格认为当前估值已透支 3-5 年成长；巴菲特认为长期仍可持有
# - 建议持仓区间：< 1500 元
```

### 评价

**优点**：

- 把"四大师"从"心法"升级为"可执行 prompt"，对个人投资者极其实用
- 多 Agent 对抗 = 把"信息盲区"显式化
- 与 Claude Code / Codex 原生集成，零额外配置

**缺点**：

- 框架本身不输出投资建议，结果仍需人判断
- "方法论"在不同市场有效性差异大，A股/港股需自行调参
- 依赖 LLM 模型的"理解深度"，便宜模型效果明显打折

**推荐指数**：⭐⭐⭐⭐（投资研究框架的范式跃迁）

---

## 3. commaai/openpilot — 300+ 车型的开源机器人操作系统

### 核心功能

`openpilot` 是 commaai 出品的**开源驾驶辅助操作系统**。它不是一个 App，而是一套完整的 robot 操作系统，替换原车 ADAS 后可解锁 300+ 款车型的自适应巡航、车道居中、车道保持等功能。

- **硬件载体**：comma 3X（一个小型吸顶式设备，OBD-II + 双 CAN + 摄像头）
- **跨车型支持**：丰田 / 本田 / 福特 / 大众 / 现代 / 特斯拉（部分）等
- **openpilot upstream**：官方持续维护的高质量代码库
- **社区 fork**：`sunnyhaibin/openpilot-fork`（中文社区优化版）等

### 适用场景

- 通勤 + 高速场景的"半自动驾驶"刚需
- 自动驾驶研究者（最易入手的开源端到端 ADAS 平台）
- 改装 / 极客车主

### 安装方式

```bash
# 硬件：购买 comma 3X（约 $1250）
# 软件：插入设备，按提示完成车型选择 + 校准
git clone https://github.com/commaai/openpilot.git
cd openpilot
# 进入 tools/ 目录做模拟器或开发
```

### 代码示例

```text
# openpilot 的消息总线示例（简化）
from cereal import messaging

msg = messaging.new_message('carState')
msg.carState.vEgo = 25.0        # 当前车速 m/s
msg.carState.steerAngleDeg = 2.0
pub = messaging.pub_sock('carState')
pub.send(msg.to_bytes())
# 在另一进程中订阅
sub = messaging.sub_sock('carState')
for m in sub:
    print(m.carState.vEgo)
```

### 评价

**优点**：

- 唯一对消费级硬件做完整开源 ADAS 的项目
- 真车实跑数据 + 大量 corner case 测试
- 模型迭代频率比车企快一个数量级

**缺点**：

- 不是 L4 自动驾驶，仍是"辅助"范畴
- 硬件不便宜，地区法规风险
- 出现事故时责任界定模糊

**推荐指数**：⭐⭐⭐⭐（汽车 + 机器人方向必看）

---

## 4. IceWhaleTech/CasaOS — 把任意旧电脑变成"个人云"

### 核心功能

`CasaOS` 是一个**面向家庭场景的开源 Personal Cloud 系统**。IceWhale 团队在 2020 年发现：算力和存储成本持续下降、云计算向边缘迁移、消费级数据资产归属长期被忽略——于是提出"5 年内 100 美元以下就能搭一台个人云"的愿景。

核心特性：

- **零代码、零表单的友好 UI**：为非技术家庭用户设计
- **多硬件支持**：ZimaBoard、Intel NUC、Raspberry Pi、旧电脑
- **精选 App Store 一键安装**：Nextcloud、HomeAssistant、AdGuard、Jellyfin、*arr 全家桶
- **10 万+ Docker 应用**：生态直接复用 Docker Hub
- **驱动器与文件管理所见即所得**

### 适用场景

- 家用 NAS 入门（比 TrueNAS / UnRAID 友好 10 倍）
- 极客 / 创作者自建数据中台
- 家庭智能化中心（HomeAssistant + 各种 Docker 服务）
- 小型工作室 / 团队的"低预算自托管"

### 安装方式

```bash
# 一行命令安装
wget -qO- https://get.casaos.io | sudo bash

# 或
curl -fsSL https://get.casaos.io | sudo bash

# 升级
wget -qO- https://get.casaos.io/update | sudo bash

# 查看版本
casaos -v

# 卸载
casaos-uninstall
```

支持的官方系统：Debian 12 ✅、Ubuntu Server 20.04 ✅、Raspberry Pi OS ✅。社区还测过 Elementary、Armbian、Alpine、OpenWrt、ArchLinux。

### 代码示例

CasaOS 的 App Store 部署基本是"点一下"：

```text
# 浏览器打开 http://your-server-ip
# App Store -> 搜 "Jellyfin" -> Install
# 30 秒后访问 http://your-server-ip:8096
# 完成家庭媒体中心搭建
```

对于需要自定部署的进阶用户，CasaOS 底层用的就是标准 Docker Compose：

```yaml
# /var/lib/casaos/apps/jellyfin/docker-compose.yml
version: "3"
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    network_mode: host
    volumes:
      - ./config:/config
      - /mnt/media:/media
    restart: unless-stopped
```

### 评价

**优点**：

- 真正意义上的"家用 NAS 入门 OS"——比 TrueNAS 易用 10 倍
- Docker 生态零成本接入，10 万+ 应用任选
- ZimaBoard 等硬件 + CasaOS 形成完整软硬一体体验
- 社区驱动更新活跃

**缺点**：

- 大规模商用场景仍建议用 UnRAID / TrueNAS
- 没有 ZFS 原生支持（用 MergerFS + SnapRAID 代替）
- 高级配置仍需 SSH 改配置文件

**推荐指数**：⭐⭐⭐⭐⭐（家庭自托管的首选，没有之一）

---

## 5. ripienaar/free-for-dev — DevOps 人的"免费羊毛目录"（连续两日上榜）

### 核心功能

`free-for-dev` 是一份**专门给 DevOps / infradev 工程师整理的"免费 SaaS/PaaS/IaaS 羊毛清单"**。所有条目都是"有免费层 (free tier) 且对工程/运维实际有用"的服务，覆盖：

- 源代码托管 / CI/CD
- 监控 / 日志 / 告警
- 数据库 / 缓存 / 消息队列
- CDN / DNS / 域名
- 邮件 / SMS / 推送
- AI / ML API
- 安全 / 身份认证
- 等等 50+ 类别、数百个服务

每个条目都标注：免费层限制、是否需信用卡、官方文档链接。

### 适用场景

- 个人项目 / Side Project 选型（白嫖 + 合法）
- 创业公司 MVP 阶段降本
- DevOps 选型对比
- "这服务还有免费层？"的冷知识补给

### 安装方式

```bash
# 本质是 Markdown 文档，clone / 浏览 / 搜索即可
git clone https://github.com/ripienaar/free-for-dev.git
cd free-for-dev

# 或直接访问
# https://free-for.dev
```

### 代码示例

没有代码，但格式高度结构化，方便脚本化使用：

```text
# 比如你想找一个"免费的对象存储"
> grep -A 3 "S3-compatible" README.md

# 输出：
# - Backblaze B2 — 10 GB 存储 + 每日 1 GB 下载
# - Cloudflare R2 — 10 GB 存储 + 1000 万次读操作
# - Wasabi — 5 GB 试用
```

### 评价

**优点**：

- 全网最全的"免费层"清单，更新极频繁
- 标注清晰：是否需信用卡、有无时间限制、是否需域名
- 社区驱动，PR 友好

**缺点**：

- 免费层政策变化快，部分条目可能已失效（需自己验证）
- 没有自动化检测机制

**推荐指数**：⭐⭐⭐⭐⭐（DevOps / 个人开发者必 star）

---

## 6. google-labs-code/design.md — 让 Agent 真正"读懂"你的设计系统

### 核心功能

`DESIGN.md` 是 Google Labs 推出的**面向编码 Agent 的视觉规范格式标准**。一份 `DESIGN.md` 让 Agent 在生成 UI 时，**不再靠截图"猜"设计意图**，而是从结构化契约里读到颜色、字号、间距、组件层级、交互态、动效曲线等"机器可消费"的规范。

- **结构化契约**：YAML/JSON 描述 design tokens + 组件元数据
- **Agent-friendly**：能 lint、能 diff、能持续消费
- **多 IDE 兼容**：Claude Code、Cursor、Windsurf、Cody 等

### 适用场景

- Design System 团队向 AI 工作流迁移
- 跨团队设计一致性（Agent 在 PR 中自动校验设计 token）
- 老项目"AI 化"：把设计规范从 Figma 导出成 `DESIGN.md`

### 安装方式

```bash
# 把 design.md 仓库 clone 到项目里
git clone https://github.com/google-labs-code/design.md.git .design-md
# 在 AGENTS.md / CLAUDE.md 中指引 Agent 加载
echo "Read .design-md/DESIGN.md before generating UI" >> AGENTS.md
```

### 代码示例

```yaml
# DESIGN.md 节选
tokens:
  color:
    primary: "#2563eb"
    surface: "#ffffff"
  radius:
    sm: "4px"
    md: "8px"
components:
  button:
    base: "h-10 px-4 rounded-md font-medium"
    variants:
      primary: "bg-primary text-white"
      ghost: "bg-transparent text-primary"
```

### 评价

**优点**：

- 把"设计系统"从"人看"升级到"Agent 看"
- 与主流 AI IDE 兼容
- 格式简单，迁移成本低

**缺点**：

- 仍处早期，生态工具链不丰富
- 与 Figma Tokens / Style Dictionary 的边界仍模糊

**推荐指数**：⭐⭐⭐⭐（Design System + AI 团队的标配）

---

## 7. microsoft/PowerToys — Windows 桌面的"瑞士军刀"（30+ 工具集）

### 核心功能

`PowerToys` 是微软官方维护的**Windows 桌面增强工具集**，包含 30+ 实用工具，覆盖效率提升、系统定制、键盘鼠标、文件管理等高频场景。代表成员：

- **PowerToys Run**（`Alt+Space` 启动器，比 macOS Spotlight 更强）
- **FancyZones**（多窗口分屏，超越 Windows 自带分屏）
- **Command Palette**（`Ctrl+Shift+P`，类 VS Code 的命令面板）
- **Mouse Without Borders**（多电脑共用一套键鼠）
- **Advanced Paste**（剪贴板增强，支持格式转换）
- **Color Picker / Screen Ruler / Image Resizer / PowerRename**
- **Always on Top / Awake / Quick Accent / Text Extractor / ZoomIt**
- 以及 20+ 其他工具

### 适用场景

- 任何想"榨干 Windows 效率"的用户
- 开发者（PowerToys Run + Keyboard Manager 几乎可以替换掉一半 AutoHotkey 脚本）
- 设计师 / 内容创作者（Color Picker、Image Resizer、Screen Ruler）

### 安装方式

```powershell
# 方式 1：WinGet（推荐）
winget install Microsoft.PowerToys -s winget

# 方式 2：Microsoft Store
# https://aka.ms/getPowertoys

# 方式 3：GitHub Releases
# https://aka.ms/installPowerToys
```

### 代码示例

PowerToys 本身是 GUI 工具，但它的"Keyboard Manager"可以零代码做"键位重映射"：

```text
# 场景：把 CapsLock 改成 Ctrl（macOS 习惯）
# 1. 打开 PowerToys Settings -> Keyboard Manager -> Remap a key
# 2. 选 CapsLock -> Ctrl
# 3. 点 OK，立即生效
```

进阶用户可以用 `Keyboard Manager` 的"快捷键重映射"做"一键启动 PowerToys Run"等。

### 评价

**优点**：

- 微软官方维护，质量稳定
- 30+ 工具的"套件"价值，远超任何单一工具
- 100% 免费 + 开源
- 与 AI 工作流配合极好（Command Palette 启动 Claude Code 等）

**缺点**：

- 仅 Windows 平台
- 偶尔会因为系统更新引入 bug
- 部分工具需要"再点一下"才会用

**推荐指数**：⭐⭐⭐⭐⭐（Windows 用户必装）

---

## 8. hugohe3/ppt-master — AI 生成"真可编辑"的 .pptx，而不是"一坨图片"

### 核心功能

`PPT Master` 是一个**运行在 AI IDE 里的"工作流 Skill"**（不是完整 Agent），目标是让 AI 把任意文档（PDF、Word、Markdown）变成**真可编辑的 PowerPoint**——不是"一张大图塞进 .pptx"，而是**真正的 DrawingML 形状 + 文字框 + 动画 + 演讲者备注**。

核心差异化：

- **Real DrawingML**：每个元素在 PowerPoint 里都能点击修改
- **支持母版 / 模板**：可以基于你自己的 .pptx 模板生成
- **演讲者备注可配音**：用 TTS 转语音旁白
- **完全本地化运行**：除 AI 模型调用外，所有步骤都在本机
- **跨 IDE 兼容**：Claude Code、Cursor、VS Code + Copilot、Codebuddy
- **跨模型兼容**：Claude、GPT、Gemini、Kimi、DeepSeek

### 适用场景

- 商务汇报、学术答辩、培训材料的"AI 提速"
- 已有 .pptx 模板但缺内容的场景（PPT Master 填内容 + 套模板）
- 想学"AI Agent + 实际生产力"的开发者

### 安装方式

```bash
# 1. 装 Python（3.10+）+ 一个 AI IDE（Claude Code 推荐）
# 2. 准备源材料
ls ./source.pdf
# 3. 启动 Claude Code
claude
# 4. 在 IDE 中调用 skill：
# "用 ppt-master 把 ./source.pdf 做成 .pptx"
```

### 代码示例

PPT Master 的核心是"对话式"工作流：

```text
# 用户：
> 用 ppt-master 把 docs/whitepaper.pdf 做成 .pptx，
  套 templates/company-template.pptx 的样式，
  演讲者备注用英文，每页加旁白音频。

# Agent 输出：
# - 解析 PDF（用 PyMuPDF）
# - 提取章节大纲（22 页）
# - 对每页写"标题 + 关键 3 点 + 演讲者备注"
# - 套用 template 的母版样式（配色、字体）
# - 生成 .pptx（python-pptx）
# - 用 TTS 生成 22 段旁白音频
# - 交付：output/whitepaper-deck.pptx + output/narration/*.mp3
```

### 评价

**优点**：

- 精准切入"AI PPT 工具最大的痛点"——能直接编辑
- 不依赖单一平台 / 模型，最大化灵活性
- 完全开源（MIT）+ 本地化
- "透明成本"：不收订阅费，只付模型 token 费

**缺点**：

- "一次出完美"不现实，作者明确说"模型决定上限"
- 高质量结果需要 Claude Opus 4.7 + 1M 上下文 + gpt-image-2，便宜模型出片质量明显下降
- 学习曲线：对"零 Python 基础"用户偏陡

**推荐指数**：⭐⭐⭐⭐（AI PPT 工具的天花板级方案）

---

## 9. JCodesMore/ai-website-cloner-template — 一键把任意网站反推成 Next.js 代码

### 核心功能

`ai-website-cloner-template` 是一个**用 AI 编程 Agent 把任意网站反推成干净、现代的 Next.js 代码库**的可复用模板。推荐 Claude Code + Opus 4.7（效果最佳），但 Codex / OpenCode / Copilot / Cursor / Windsurf / Gemini CLI / Cline / Roo Code / Continue / Amazon Q / Augment Code / Aider 也都支持。

核心操作是 `/clone-website` 技能，背后是一套多阶段流水线：

1. **Reconnaissance**：截图 + 提取设计令牌 + 交互扫描（滚动 / 点击 / hover / 响应式）
2. **Foundation**：更新字体、颜色、全局样式，下载所有资产
3. **Component Specs**：写详细规范文件（`docs/research/components/`），含精确的 `getComputedStyle()` 取值、状态、行为、内容
4. **Parallel Build**：在 git worktree 里派发并行 builder Agent，每 section/component 一个
5. **Assembly & QA**：合并 worktree，连接页面，对原站做 visual diff

技术栈：Next.js 16（App Router）+ React 19 + TypeScript strict + shadcn/ui（Radix + Tailwind v4）+ Tailwind v4 oklch tokens + Lucide React（克隆时替换为提取的 SVG）。

### 适用场景

- **平台迁移**：把 WordPress / Webflow / Squarespace 老站重建到现代 Next.js
- **失源找回**：站点还在但仓库没了、开发者跑路了、栈太 legacy
- **学习参考**：用真代码解构"产品网站是怎么做出来的"
- **MVP 快速复用**：拿一个做得好的设计当骨架，往里填自己的内容

### 安装方式

```bash
# 1. 在 GitHub 上点 "Use this template" 创建你自己的仓库
# 2. clone 到本地
git clone https://github.com/YOUR-USERNAME/YOUR-NEW-REPOSITORY.git
cd YOUR-NEW-REPOSITORY

# 3. 安装依赖
npm install

# 4. 启动 AI 代理
claude --chrome

# 5. 运行技能
> /clone-website https://example.com
```

### 代码示例

```text
> /clone-website https://stripe.com
# Agent 自动完成：
# - 截图、提取 design token
# - 写 12 个 component spec（hero, pricing, footer, ...）
# - 派 5 个 builder Agent 并行实现
# - 合并、连接、跑 visual diff
# - 输出可立即 npm run dev 的 Next.js 代码库
```

### 评价

**优点**：

- 把"克隆网站"从"玄学"变成"流水线"
- 多 Agent 并行 + Component Spec 模板 = 几乎零猜测
- 完整工具链：Next.js 16 + React 19 + TS strict + Tailwind v4，**不是玩具栈**
- 支持 10+ AI 编程代理，不锁定单一供应商

**缺点**：

- 严重依赖"AI 编程代理 + 大模型"，便宜模型会"跑飞"
- 反推出来的代码不一定"无 bug"，需要人工 review
- 对复杂 SPA / 鉴权站效果一般

**推荐指数**：⭐⭐⭐⭐（网站重建 / 学习的杀手锏工具）

---

## 10. garrytan/gstack — Garry Tan 的"Claude Code 团队化"工作流（YC CEO 亲推）

### 核心功能

`gstack` 是 YC 现任 CEO Garry Tan 公开的**"23 个 opinionated 工具 + 8 个 power tools"** Claude Code 工作流集。它把 Claude Code 从"一个 AI 助手"变成"一支虚拟工程团队"——分别担任 CEO、设计师、工程经理、Release Manager、Doc Engineer、QA、Security Officer 等角色，全部以 **slash 命令 + Markdown** 实现，MIT 协议。

- **`/office-hours`**：产品问诊，6 个强制问题
- **`/plan-ceo-review` / `/plan-eng-review` / `/plan-design-review` / `/plan-devex-review`**：多视角 review
- **`/design-consultation` / `/design-shotgun` / `/design-html`**：设计工作流
- **`/review` / `/ship` / `/land-and-deploy` / `/canary`**：代码 review + 发布
- **`/qa` / `/qa-only`**：QA 自动化（**真打开浏览器**）
- **`/cso`**：安全审计（OWASP + STRIDE）
- **`/browse` / `/connect-chrome`**：浏览器集成
- **`/benchmark` / `/codex` / `/autoplan` / `/retro` / `/investigate` 等**：方法论工具

> Garry 自己说："过去 60 天我交付了 3 个生产服务、40+ feature，全程 Part-time，2026 年的逻辑代码产出量是 2013 年的 240 倍。"

### 适用场景

- 单兵 founder / 技术 CEO 想"以一打十"
- Claude Code 第一次使用的小白（结构化角色 > 空白 prompt）
- Tech Lead / Staff Engineer 想"每次 PR 都过 QA + 安全 + 文档"
- 团队用 `gstack-team-init` 一键把工作流同步给所有成员

### 安装方式

```bash
# 1. 一行命令安装
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack
./setup

# 2. 在项目的 CLAUDE.md 中加入 gstack 段（Claude 自动发现）
# 3. 团队版：
(cd ~/.claude/skills/gstack && ./setup --team) && ~/.claude/skills/gstack/bin/gstack-team-init required
git add .claude/ CLAUDE.md
git commit -m "require gstack for AI-assisted work"
```

前置依赖：Claude Code + Git + Bun v1.0+（Windows 还需要 Node.js）

### 代码示例

```text
# 场景 1：第一次用它
> /office-hours
# "我正在做一个帮独立开发者接外包的平台……"
# 6 个问题强制把"模糊想法"变成"清晰 spec"

# 场景 2：每个 PR 必经流程
> /plan-ceo-review   # CEO 视角：这个功能值不值得做？
> /plan-eng-review   # 工程：架构、数据流、风险
> /plan-design-review # 设计：UI / UX 一致性
> /review            # 代码 review
> /qa                # 自动化 QA（真开浏览器）
> /cso               # 安全审计
> /ship              # 发 PR
```

OpenClaw 用户更省事——装 `gstack-openclaw-*` 系列 skill，**直接在 OpenClaw agent 里跑**，无需 Claude Code：

```text
# 在 OpenClaw 里：
> Load gstack. Run /office-hours then /autoplan. Save the plan, don't implement.
> Load gstack. Run /cso
> Load gstack. Run /qa https://staging.myapp.com
```

### 评价

**优点**：

- "把 YC 总裁的工作流开源"——视角独特，价值极高
- 23 个工具全部 Markdown + MIT，免费到不可思议
- 真打通了"产品 → 架构 → 设计 → 代码 → QA → 安全 → 发布"全链路
- 与 OpenClaw / Claude Code 双向集成

**缺点**：

- 学习曲线：23 个 slash 命令全用上需要几天
- 强依赖 Claude Code（不是模型无关）
- 部分工具（`/qa` 的浏览器、`/connect-chrome`）在 Linux server 跑要配置

**推荐指数**：⭐⭐⭐⭐⭐（AI 工作流的"操作系统级"项目，必 star）

---

## 今日趋势小结

对比昨日（2026-06-27），今天 Trending 名单的关键变化：

**新进 Top 10**：`IceWhaleTech/CasaOS`（#4）、`microsoft/PowerToys`（#7）、`hugohe3/ppt-master`（#8）、`garrytan/gstack`（#10）

**排名上升**：`xbtlin/ai-berkshire`（#10 → #2）

**继续在榜**：`simplex-chat`（#1 不变）、`commaai/openpilot`（#3 不变）、`ripienaar/free-for-dev`（#6 → #5）、`google-labs-code/design.md`（#2 → #6）

**跌出 Top 10**：`kunchenguid/no-mistakes`、`grafana/grafana`、`opendatalab/MinerU`、`alchaincyf/zhangxuefeng-skill`、`mauriceboe/TREK`

**今日信号**：

1. **"个人云 + 桌面增强"回归**：CasaOS + PowerToys 同步上榜，说明开发者正在从"云依赖"回到"本地 + 桌面"的工作流
2. **AI 工作流走向"团队化"**：gstack 出现 = 单兵 founder 的"工程团队"已经成为产品
3. **"真可编辑"成为 AI 工具新标尺**：ppt-master 强调"Real DrawingML"、design.md 强调"Agent 看得懂"——都在解决 AI 工具"输出物不能直接用"的老问题

> 数据来源：[github.com/trending](https://github.com/trending)（每日 UTC 榜单）
> 抓取时间：2026-06-28 13:02 (Asia/Shanghai) / 2026-06-28 05:02 (UTC)
> 抓取方式：直接 HTTP GET + 解析 trending 页面
