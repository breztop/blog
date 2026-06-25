---
title: '每日开源速递 - 2026-06-25'
date: 2026-06-25T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026年6月25日

今日 GitHub Trending 榜单呈现出一种奇妙的"基础设施回归"信号。Apple 官方出品的 Swift 容器运行时 `apple/container` 单日斩获近两千星，成为榜单上半场的最大新闻；Google Labs 推出的 `design.md` 把"设计令牌"做成 Agent 可以理解的契约，是 Agent-Native 工具链拼图的关键一块；NousResearch 的 `hermes-agent` 打出"the agent that grows with you"的旗号，把自学习闭环正式产品化。视频侧 `calesthio/OpenMontage` 连续两天登顶，生态正在快速形成。下面让我们逐个深入了解。

> 数据来源：[github.com/trending](https://github.com/trending)（每日 UTC 榜单）
> 抓取时间：2026-06-25 09:14 (Asia/Shanghai) / 2026-06-25 01:14 (UTC)

---

## 1. apple/container — Apple 官方出品的 Mac 容器运行时

### 核心功能

`container` 是 Apple 开源的一款在 Mac 上创建并运行 Linux 容器（以轻量级虚拟机为载体）的工具，**完全用 Swift 编写，针对 Apple silicon 深度优化**。它底层依赖 [apple/containerization](https://github.com/apple/containerization) Swift 包做容器/镜像/进程的低层管理，对外则暴露与 OCI 规范完全兼容的接口——你可以从任何标准容器注册中心 pull 镜像，也可以把构建出来的镜像 push 回去，并在任何其他 OCI 兼容运行时中启动。

最关键的设计选择是：**每个容器跑在独立的轻量级 VM 里**，而不是和传统容器那样共享宿主机内核。这意味着：

- Linux 内核版本可以与宿主机 macOS 完全独立，避免兼容性问题
- 默认具备强隔离边界，安全模型与 macOS 沙箱自然契合
- 仍然维持 OCI 兼容，开发者无需切换工具链

`container` 依赖 macOS 26 的虚拟化和网络增强，不支持更早系统版本。它的 API 稳定性目前仅在 patch 版本之间保证（0.1.1 ↔ 0.1.2），minor 升级可能存在破坏性变更。

### 适用场景

- **macOS 上的"真 Linux"开发环境**：想要 Linux 行为、又不想装 Docker Desktop / OrbStack 的团队
- **CI on Apple Silicon**：Mac mini 集群做 iOS/macOS + Linux 双栈构建
- **企业内分发**：本地构建的 OCI 镜像可以直接 push 到 Harbor、ECR、GHCR 等任意 OCI 兼容仓库
- **安全敏感场景**：VM 级隔离比共享内核更适合多租户环境

### 安装方式

```bash
# 1. 从 GitHub Releases 下载签名安装包（pkg），双击安装到 /usr/local
# 2. 启动系统服务
container system start

# 3. 更新到最新版本
/usr/local/bin/update-container.sh

# 4. 卸载（-k 保留用户数据，-d 一并删除）
/usr/local/bin/uninstall-container.sh -k
```

### 代码示例

```bash
# 拉取一个标准 OCI 镜像
container image pull ghcr.io/oras-project/registry:latest

# 直接运行（与 docker run 体验接近）
container run --rm -p 8080:8080 ghcr.io/oras-project/registry:latest

# 查看镜像
container image list
```

### 评价

**优点**：
- Apple 官方出品，质量背书与长期维护预期明确
- 走 VM 隔离路线，安全模型与传统容器有质的差异
- 完整的 OCI 兼容，无锁定

**缺点**：
- 仅支持 Apple silicon + macOS 26，Intel Mac 和旧系统直接出局
- API 仍在快速演进（目前还是 0.x），生态（Compose 替代、日志/监控）尚需补齐
- 1,800+ 日增星中相当部分是"Apple 罕见开源"的情绪盘，长期价值要看落地

**推荐指数**：⭐⭐⭐⭐⭐（Apple 平台的"原生容器方案"，值得 Mac 团队尽早评估）

---

## 2. google-labs-code/design.md — 让 Agent 真正"读懂"你的设计系统

### 核心功能

`DESIGN.md` 是 Google Labs 推出的一份**面向编码 Agent 的视觉规范格式标准**。它把一个项目的视觉身份（visual identity）拆成两层：

- **YAML front matter**：机器可读的设计令牌（design tokens），包括 `colors`、`typography`、`rounded`、`spacing`、`components` 等
- **Markdown 正文**：人类可读的设计意图与使用规则（Overview / Colors / Typography / Layout / Elevation / Shapes / Components / Do's and Don'ts）

光给 Agent 一堆色值是不够的——它不知道 *为什么* 用这个色，*什么时候* 用，*为什么* 不用。`DESIGN.md` 同时回答"是什么"和"为什么"。

配套 npm 包 `@google/design.md` 提供了三条 CLI：

```bash
# 校验文件、查 token 引用、跑 WCAG 对比度
npx @google/design.md lint DESIGN.md

# 对比新旧版本，定位回归
npx @google/design.md diff DESIGN.md DESIGN-v2.md

# 导出为 Tailwind / DTCG / CSS 变量
npx @google/design.md export --format css-tailwind DESIGN.md > theme.css
```

### 适用场景

- **Design System 团队**：把"团队规范"变成可执行、可校验、可 diff 的工件
- **AI 编码助手集成**：在仓库根放一份 `DESIGN.md`，Claude Code / Cursor / Codex 第一次拉代码就能拿到全部视觉约束
- **设计/开发协作**：设计师改 token 就能自动 lint 出"对比度不达标"等回归
- **多品牌多主题项目**：每个品牌/主题一个 `DESIGN.md`，CI 跑 diff 防止越界

### 安装方式

```bash
# 全局安装 CLI
npm install -g @google/design.md

# 或者在项目里跑
npm install @google/design.md
```

### 代码示例

```markdown
---
name: Heritage
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral:  "#F7F5F2"
typography:
  h1:        { fontFamily: Public Sans, fontSize: 3rem }
  body-md:   { fontFamily: Public Sans, fontSize: 1rem }
  label-caps:{ fontFamily: Space Grotesk, fontSize: 0.75rem }
rounded: { sm: 4px, md: 8px }
spacing: { sm: 8px, md: 16px }
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor:       "#ffffff"
    rounded:         "{rounded.sm}"
    padding:         "12px"
---

## Overview
Architectural Minimalism meets Journalistic Gravitas. The UI evokes a
premium matte finish — a high-end broadsheet or contemporary gallery.

## Colors
- **Primary (#1A1C1E):** Deep ink for headlines and core text.
- **Tertiary (#B8422E):** "Boston Clay" — the sole driver for interaction.
```

```bash
$ npx @google/design.md lint DESIGN.md
{
  "findings": [
    { "severity": "warning",
      "path": "components.button-primary",
      "message": "textColor (#ffffff) on backgroundColor (#1A1C1E) has contrast ratio 15.42:1 — passes WCAG AA." }
  ],
  "summary": { "errors": 0, "warnings": 1, "info": 1 }
}
```

### 评价

**优点**：
- 直击"Agent 时代的设计资产"痛点：纯 token 不够，意图也得写下来
- 配套 CLI 直接产出 lint/diff/export，能立刻接到 CI
- 格式简单，Markdown + YAML，几乎零学习成本
- 导出到 Tailwind/DTCG/CSS 变量，与主流前端栈天然兼容

**缺点**：
- 仍处 alpha 阶段（`version: "alpha"`），schema 演进速度可能快
- Windows 下 `design.md` 这种带 `.md` 后缀的 bin 名会被 PowerShell 误识别为 Markdown 文件，必须用 `designmd` alias
- 想要在大型企业内部推动，设计师和前端需要重新对齐工作流

**推荐指数**：⭐⭐⭐⭐⭐（所有"Agent-Native 前端项目"的最低成本起点）

---

## 3. calesthio/OpenMontage — 把 AI 编程助手变成视频制片厂

### 核心功能

`OpenMontage` 自称"世界首个开源的 Agentic 视频制作系统"，已在昨日榜单登顶，今日继续霸榜。它把"做视频"这件事拆成 **12 条管线（pipelines）+ 52 个工具（tools）+ 500+ Agent 技能（skills）**，让任意 AI 编程助手（Claude Code / Codex / Cursor / Copilot / Windsurf）能够独立完成从概念到成片的全部流程：

1. **参考视频解析**：粘贴 YouTube/Short/Reel/TikTok，本地链接都行，Agent 自动分析转录、节奏、场景、关键帧、风格
2. **概念生成**：2-3 个差异化方案 + 诚实工具路径 + 成本预估 + 样片，先审批再开干
3. **多模态资产生产**：脚本、图像（FLUX/gpt-image-1/Recraft）、视频片段（Veo/Kling）、TTS 旁白、WhisperX 词级字幕、自动检索免版税音乐
4. **Remotion 合成**：把所有素材按时间线组装成最终视频
5. **自审机制**：ffprobe 校验、关键帧采样、音量分析、交付承诺校验、字幕检查
6. **决策可审计**：每个 provider 选择都按 7 维度打分并落日志

`OpenMontage` 还能在"零视频生成"模式下做真视频：Agent 从免费素材库检索真实动态片段并剪辑成片，规避了"把几张图交叉淡入就当视频"的常见套路。代表性作品包括 60 秒皮克斯风格动画"THE LAST BANANA"（$1.33）、"VOID — Neural Interface"产品广告（$0.69）、吉卜力风格"Candyland"/"Mori no Seishin"（$0.15，纯静态图 + Remotion 动效）。

### 适用场景

- **短视频矩阵**：自媒体、营销号、跨境电商的批量短视频生产
- **概念可视化**：把抽象 idea 变成 30-90 秒的"动态 Pitch Deck"
- **企业内部培训**：把培训文档转成动画讲解
- **影视/广告 pre-visualization**：导演/创意快速验证镜头节奏

### 安装方式

```bash
# 前置：Python 3.10+ / FFmpeg / Node.js 18+
git clone https://github.com/calesthio/OpenMontage.git
cd OpenMontage
make setup

# 然后在 Claude Code / Cursor 中输入自然语言：
# "Make a 60-second animated explainer about how neural networks learn"
```

### 代码示例

```python
# 1. 查询当前能力包络
python -c "from tools.tool_registry import registry; import json; registry.discover(); print(json.dumps(registry.support_envelope(), indent=2))"

# 2. 选定一条管线后，Agent 会按 stage director 技能执行
# 详细见 .claude/skills/pipelines/ 下每条管线对应的 SKILL.md
```

```bash
# .env — 所有 key 全部可选，按需开启
FAL_KEY=your-key          # FLUX 图像 + Google Veo/Kling/MiniMax 视频
OPENAI_API_KEY=your-key   # gpt-image-1 + TTS
GOOGLE_API_KEY=your-key   # Veo 视频 / Chirp3-HD 旁白
```

### 评价

**优点**：
- 单价极低：完整 60s 短片可以压到 $0.15-$1.33
- "真视频"路径：免费素材 + 自动剪辑，规避"动画冒充视频"的廉价感
- 自审机制把"Agent 幻觉导致的交付翻车"前置拦截
- 与任何 AI 编程助手兼容，不锁定特定供应商

**缺点**：
- 对 FFmpeg/Remotion/Node 都有要求，部署门槛高于"开箱即用"型 SaaS
- Agent 需要时间理解"管线选择"语义，新手需先读 AGENT_GUIDE.md / PROJECT_CONTEXT.md
- 闭源部分（生成式 AI 处理）与开源（编辑器/MCP/Agent Chat）边界需要明确接受

**推荐指数**：⭐⭐⭐⭐⭐（Agentic 视频赛道事实标准）

---

## 4. NousResearch/hermes-agent — 会自我成长的 Agent

### 核心功能

`hermes-agent` 是 Nous Research 推出的"自改进 Agent"，核心卖点是**唯一内置学习闭环的 Agent**：

- **技能自创建**：完成复杂任务后自动抽象出可复用 skill
- **技能在使用中自改进**：根据实际反馈迭代 skill 内容
- **主动 nudge 持久化**：定期提醒自己把知识落到长期记忆
- **跨会话检索**：FTS5 全文索引 + LLM 摘要，跨 session 召回
- **Honcho dialectic 用户建模**：[plastic-labs/honcho](https://github.com/plastic-labs/honcho) 帮你建立"用户是谁"的持续模型
- **兼容 [agentskills.io](https://agentskills.io) 开放标准**

它支持几乎所有 LLM 供应商：`hermes model` 一行切换——Nous Portal、OpenRouter（200+ 模型）、NovitaAI、NVIDIA NIM（Nemotron）、Xiaomi MiMo、z.ai/GLM、Kimi/Moonshot、MiniMax、Hugging Face、OpenAI、本地端点——**零代码改动、零锁定**。

部署形态同样灵活：

- **TUI**：完整终端界面，多行编辑、slash 自动补全、对话历史、interrupt-and-redirect
- **多端消息网关**：Telegram / Discord / Slack / WhatsApp / Signal / CLI，统一入口
- **后台运行**：6 种终端 backend（local / Docker / SSH / Singularity / Modal / Daytona），其中 Daytona 与 Modal 提供 serverless 持久化——Agent 闲时空闲，几乎零成本
- **定时自动化**：内置 cron 调度器，每日报告、每晚备份、每周审计都能用自然语言描述
- **并行委派**：spawn 隔离子 Agent 做并行任务；写 Python 脚本通过 RPC 调用工具，把多步流水线折叠成"零上下文成本"的 turn
- **训练就绪**：批量轨迹生成 + 轨迹压缩，可用于训练下一代 tool-calling 模型

### 适用场景

- **跨会话的个人 Agent**：跑在 $5 VPS 或 serverless 上，通过 Telegram 与之对话
- **24/7 自动化值守**：定时报告、夜间构建、异常告警
- **多 LLM 路由实验场**：在一个 CLI 里横向对比 200+ 模型
- **Agent 行为数据采集**：给下一代 tool-calling 模型做 trajectory 数据

### 安装方式

```bash
# macOS / Linux
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# Windows (PowerShell) — 原生支持，无需 WSL
iex (irm https://hermes-agent.nousresearch.com/install.ps1)

# 装完直接：
source ~/.bashrc
hermes
```

### 代码示例

```bash
# 切换模型
hermes model openai/gpt-5
hermes model anthropic/claude-opus-4.7
hermes model zai/glm-5
hermes model nous/hermes-4

# 在 Telegram 网关里收到通知后跟进
# （手机端 → Agent 在云端 GPU VM → 完成后 push 回手机）
```

### 评价

**优点**：
- "自改进 + 自学习"是真做出来了，不是 marketing
- 真正模型无关，6 种 backend + 200+ 模型，无锁定
- 部署形态覆盖最广：从 $5 VPS 到 GPU 集群到 serverless
- 同时支持个人用例和研究者做 trajectory 训练

**缺点**：
- 默认走 Nous Portal/OpenRouter，海外 API 配额与延迟是隐性成本
- 自学习机制意味着"短期不可复现性"——同一 prompt 不同 session 行为可能不同
- 学习曲线比 Cursor/Claude Code 纯 CLI 模式更陡，需要读 docs

**推荐指数**：⭐⭐⭐⭐⭐（"有自己想法"的 Agent，对长期用户/研究者尤其有价值）

---

## 5. stablyai/orca — 100x builder 的 AI 编排器

### 核心功能

`Orca` 把自己定位为"AI Orchestrator for 100x builders"——你可以在同一个桌面里**并行跑 Codex、Claude Code、OpenCode、Pi 等多个 CLI Agent，每个都在自己的 git worktree 里**，统一追踪、统一对比、统一合并。配套还有 iOS/Android 移动伴侣，让你随时监看 Agent 状态、收完成通知、回传追问。

核心能力一览：

- **Parallel Worktrees**：一个 prompt 扇出到五个 Agent，每个跑独立 worktree，结果横向对比，winner 一键 merge
- **Mobile Companion**：iOS / Android App，Agent 完成时推送 + 远程追问
- **Terminal Splits**：Ghostty 级 WebGL 终端，无限 split，回滚存活过重启
- **Design Mode**：在真实 Chromium 窗口点 UI 元素，HTML + CSS + 裁剪截图直接进 Agent prompt
- **GitHub & Linear 原生集成**：在 App 内浏览 PR/Issue/Project Board，从任意任务开个 worktree
- **SSH Worktrees**：把 Agent 推到远端大机器上跑，自动重连 + 端口转发
- **AI Diff 注释**：diff 行级评论回流给 Agent，review/edit/commit 一处完成
- **拖文件给 Agent**：VS Code 级编辑器，全局自动保存，拖拽图片/文件直进 prompt
- **Orca CLI**：`orca worktree create / snapshot / click / fill`，Agent 也能驱动 Orca
- **Computer Use**：让 Agent 操控真实桌面 App 处理需要 UI 互动的 workflow

兼容性几乎"包容一切"：Claude Code、Codex、Grok、Cursor、GitHub Copilot、OpenCode、MiMo Code、Amp、OpenClaude、Antigravity、Pi、oh-my-pi、Hermes Agent、Devin、Goose、Auggie、Autohand Code、Charm、Cline、Codebuff、Command Code、Continue、Droid、Kilocode、Kimi、Kiro、Mistral Vibe、Qwen Code、Rovo Dev……以及"任何 CLI Agent"。

### 适用场景

- **"我该用 Codex 还是 Claude Code 跑这个任务？"**：跑两次对比
- **大 PR 评审**：5 个 worktree 并行修 lint/test/docs，每个跑不同 Agent
- **跨设备工作流**：Mac 主机 + 手机端临场指挥
- **Agent 编排自动化**：CI 里 spawn 多个 Agent 做并行化实验

### 安装方式

```bash
# macOS (Homebrew)
brew install --cask stablyai/orca/orca

# Arch Linux (AUR)
yay -S stably-orca-bin

# 其他平台
# https://onorca.dev/download 下载
# macOS Apple Silicon / Intel / Windows .exe / Linux AppImage 都有
```

### 代码示例

```bash
# 一条 prompt 扇出到多个 Agent
orca fanout "重构这个 module 让它支持 async streaming" \
  --agents claude,codex,opencode \
  --worktree-count 3

# 横向对比 + 选择 winner
orca compare
orca merge --winner claude-2
```

### 评价

**优点**：
- 真正"AI IDE 替代品"：把 IDE、终端、Git 客户端、Issue 跟踪、移动端监控全部打通
- Agent-agnostic，不强迫用户绑定某一个供应商
- 团队形态（多 worktree + 对比 + 合并）开箱即用
- 移动端体验完整，不只是"通知中心"

**缺点**：
- 桌面客户端资源占用不轻（Chromium 引擎 + WebGL）
- 仍在 0.x，feature list 跟着 changelog 跑，没有"完整文档"
- 部分高级功能（SSH Worktree、Computer Use）有学习曲线

**推荐指数**：⭐⭐⭐⭐（"重度 Agent 用户"的生产力神器，轻度用户可观望）

---

## 6. revfactory/harness — Claude Code 的"团队架构工厂"

### 核心功能

`Harness` 是 Claude Code 的一个 plugin，定位 L3 Meta-Factory 层中的"Team-Architecture Factory"。它不是"一个 Harness"，而是**专门生成 Harness 的 Harness**——你说一句"build a harness for this project"，它就根据你的领域描述，从 6 种预定义团队架构模式里选一个，输出对应的 `.claude/agents/`（Agent 定义）和 `.claude/skills/`（技能文件）。

6 种团队架构：

| 模式 | 适用场景 |
| --- | --- |
| **Pipeline** | 顺序依赖任务（research → draft → review） |
| **Fan-out / Fan-in** | 并行独立任务（多语言翻译、跨平台适配） |
| **Expert Pool** | 上下文相关的选择调用（按需选专家） |
| **Producer-Reviewer** | 生成 + 质量评审（写代码 + 审代码） |
| **Supervisor** | 中央调度 + 动态任务分配 |
| **Hierarchical Delegation** | 递归上下级委派 |

6 阶段工作流：Domain Analysis → Team Architecture Design → Agent Definition Generation → Skill Generation → Integration & Orchestration → Validation & Testing。

两种执行模式：

- **Agent Teams**（默认）：用 TeamCreate + SendMessage + TaskCreate 协调 ≥2 个 Agent 协作
- **Subagents**：直接调 Agent tool，适合一次性任务

与同类 L3 工具的边界清晰：`coleam00/Archon` 做 Runtime-Configuration Factory（确定性运行时配置），`SaehwanPark/meta-harness` 是 Codex Runtime Port，`affaan-m/everything-claude-code` 是 L2 跨 Harness 标准化。

### 适用场景

- **新项目启动**：第一周决定"团队应该怎么搭"
- **跨项目复用团队模板**：从一次成功的 harness 抽取 pattern
- **复杂任务分解**：当 prompt 太长、L3 调度不够用时升级到团队
- **教学/咨询**：把"我搭过的 Agent 团队"沉淀成可分发模板

### 安装方式

```bash
# 在 Claude Code 里
/plugin marketplace add revfactory/harness
/plugin install harness@harness-marketplace

# 或者手动复制
cp -r skills/harness ~/.claude/skills/harness
```

### 代码示例

```text
# 在 Claude Code 里直接说
> Build a harness for this project
> Design an agent team for this domain
> Set up a harness
```

```text
# 生成的目录结构
.claude/
  agents/        # 各 Agent 的定义（角色、prompt、工具集）
  skills/        # Agent 们要用的 SKILL.md + references/
```

### 评价

**优点**：
- 明确卡位 L3 Meta-Factory，与 Archon / meta-harness / ECC 等相邻项目边界清晰
- 6 模式覆盖了 90% 团队场景，不需要从零设计
- 默认走 Agent Teams（多 Agent 协作）而非 Subagent 兜底，能力上限更高
- 完整的 6 阶段工作流含 Validation & Testing，避免"搭完不能用"

**缺点**：
- 强绑定 Claude Code，Codex/Cursor 用户要绕路
- 对 prompt 质量依赖高，"领域描述"模糊会得到模糊架构
- 目前 star 7,740 属早期，参考案例（team-examples.md）数量有限

**推荐指数**：⭐⭐⭐⭐（Claude Code 重度用户的"团队装配车间"）

---

## 7. ZhuLinsen/daily_stock_analysis — LLM 驱动的多市场股票分析系统

### 核心功能

`daily_stock_analysis` 是面向 A 股 / 港股 / 美股 / 日股 / 韩股的自选股智能分析系统，每日自动产出"决策仪表盘"并推送到企业微信 / 飞书 / Telegram / Discord / Slack / 邮箱。**5 分钟完成部署，零成本，无需服务器**——默认在 GitHub Actions 上跑。

输出能力：

- **AI 决策报告**：核心结论、评分、趋势、买卖点位、风险警报、催化因素、操作检查清单
- **多市场数据聚合**：A/H/US/ETF 行情、K 线、技术指标、资金流、筹码、新闻、公告、基本面；日股/韩股支持 YFinance 日线 + 基础技术指标（capital_flow、dragon_tiger 等高阶字段按市场边界降级）
- **Web/桌面工作台**：手动分析、任务进度、历史报告、Markdown 回测、持仓、配置管理、浅色/深色主题
- **Agent 策略问股**：多轮追问 + 15 种内置策略（均线、缠论、波浪、趋势、热点、事件、成长、预期）
- **智能导入与补全**：图片 / CSV / Excel / 剪贴板导入；股票代码 / 名称 / 拼音 / 别名补全
- **自动化与推送**：GitHub Actions / Docker / 本地 crontab / FastAPI 服务 + 多通道推送

AI 模型与数据源覆盖面也很广：Anspire / AIHubMix / Gemini / OpenAI 兼容 / DeepSeek / 通义千问 / Claude / Ollama 本地模型；行情源含 TickFlow / AkShare / Tushare / Pytdx / Baostock / YFinance / Longbridge；新闻搜索含 Anspire / SerpAPI / Tavily / Bocha / Brave / MiniMax / SearXNG；社交舆情可接 Stock Sentiment API（Reddit / X / Polymarket，仅美股）。

### 适用场景

- **A 股散户**：工作日 18:00 自动收到"自选股决策仪表盘"
- **多市场投资人**：跨 A/H/US/JP/KR 一套流水线
- **金融自媒体**：把日报自动转成公众号 / 飞书推文
- **量化研究**：策略回测、Agent 问股、舆情监测

### 安装方式

```bash
# 1. Fork 仓库
# 2. Settings → Secrets 配置 ANSPIRE_API_KEYS + STOCK_LIST 等
# 3. Actions 标签启用 I understand my workflows
# 4. 手动 Run workflow 或等工作日 18:00 自动执行
```

### 代码示例

```yaml
# .github/workflows 关键 Secret
ANSPIRE_API_KEYS: sk-xxx       # Anspire（同时启用大模型 + 中文联网搜索）
AIHUBMIX_KEY: sk-xxx           # AIHubMix（项目享 10% 优惠）
WECHAT_WEBHOOK_URL: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx
STOCK_LIST: 600519,hk00700,AAPL,7203.T,005930.KS
```

```text
# 示例推送
【决策仪表盘 | 2026-06-25 收盘】
核心结论：震荡偏弱 / 评分 6.2/10
趋势：30 分钟 MA5 下穿 MA20，短线转弱
风险警报：北向资金连续 3 日净流出
操作检查：跌破 1750 减仓 30%，回踩 1680 加仓
```

### 评价

**优点**：
- 真正的"零成本自动化"：GitHub Actions 免费额度 + 默认模型可换
- 多市场、多数据源、多推送渠道——基本覆盖散户/中小机构的全部需要
- 文档扎实：完整配置、LLM 指南、市场支持边界都明文
- 中文化做得最彻底（产品预览/推送效果/文档中心都是中文优先）

**缺点**：
- 决策报告本质是"提示 + 评分"，不是交易信号——不要拿来直接下单
- 社交舆情仅美股，其他市场仍以官方新闻/公告为主
- 默认走云端 LLM，本地化部署需要 Docker + Ollama 自行组装

**推荐指数**：⭐⭐⭐⭐⭐（A 股散户的"个人投研助理"标杆）

---

## 8. JCodesMore/ai-website-cloner-template — 一键把任意网站反推成 Next.js 代码

### 核心功能

`ai-website-cloner-template` 是一个**用 AI 编程 Agent 把任意网站反推成干净、现代的 Next.js 代码库**的可复用模板。推荐 Claude Code + Opus 4.7（效果最佳），但 Codex / OpenCode / Copilot / Cursor / Windsurf / Gemini CLI / Cline / Roo Code / Continue / Amazon Q / Augment Code / Aider 也都支持。

核心操作是 `/clone-website` 技能，背后是一套多阶段流水线：

1. **Reconnaissance**：截图 + 提取设计令牌 + 交互扫描（滚动 / 点击 / hover / 响应式）
2. **Foundation**：更新字体、颜色、全局样式，下载所有资产
3. **Component Specs**：写详细规范文件（`docs/research/components/`），含精确的 `getComputedStyle()` 取值、状态、行为、内容
4. **Parallel Build**：在 git worktree 里派发并行 builder Agent，每 section/component 一个
5. **Assembly & QA**：合并 worktree，连接页面，对原站做 visual diff

每个 builder Agent 拿到的不是"猜一猜"，而是**完整 component 规范 inline**——精确的 computed style、交互模型、多状态内容、响应式断点、资产路径，零猜测。

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
- 把"反推网站"做成了**可复用、跨 Agent 的标准流程**
- 不靠 OCR 也不靠 F12 抄样式：拿真实 getComputedStyle() 当 contract
- 并行 builder + worktree 隔离，速度比单 Agent 串行快几个数量级
- 显式列出"禁止用途"（钓鱼 / 仿冒 / 侵权）——伦理边界清晰

**缺点**：
- 强烈推荐 Claude Code Opus 4.7，其他 Agent 效果参差
- 站点有 ToS 禁止爬取/复制的需要先确认法律风险
- 设计极简、动画复杂的网站，spec 质量决定一切——仍有失败案例

**推荐指数**：⭐⭐⭐⭐（前端工程师的"网站考古工具"，善用之）

---

## 9. interviewstreet/hiring-agent — 可解释的简历评分 Agent

### 核心功能

`hiring-agent` 把"评估简历"做成了**Resume-to-Score 流水线**：PDF 解析 → 结构化抽取 → GitHub 信号增强 → 公平、可解释评分。完全可本地运行（Ollama）或用 Google Gemini。

5 步流程：

1. **PDF 抽取**：`pymupdf_rag.py` + `pdf.py` 用 PyMuPDF 把简历 PDF 转 Markdown-like 文本
2. **分节解析**：`prompts/templates/*.jinja` 严格指令每个 section（Basics / Work / Education / Skills / Projects / Awards），LLM 逐节产出 JSON
3. **GitHub 增强**：从简历抓 GitHub 用户名 → 拉 profile + repos → 分类项目 → 让 LLM 选 7 个最有代表性的（含 author commit threshold）
4. **公平评分**：`evaluator.py` 跑 strict-scored 评估，输出 `open_source` / `self_projects` / `production` / `technical_skills` + bonus + deduction + evidence
5. **输出与 CSV 导出**：`score.py` 打印可读报告；`DEVELOPMENT_MODE=True` 时追加 CSV + 缓存中间 JSON

公平性约束内嵌在 prompt template 里：评分标准对所有候选人一致，不因教育背景、品牌等做先验倾斜。所有评分都带 evidence（哪段简历内容/哪个 repo 触发了这个分数）。

### 适用场景

- **HR 初筛**：100+ 简历批量打分，按分数排序再人工看
- **校招自动化**：统一评分标准，规避面试官偏好漂移
- **面试准备**：候选人提交简历后自动给面试官一份 evidence-backed 简报
- **公平性审计**：所有评分都带 evidence，可回溯、可申诉

### 安装方式

```bash
git clone https://github.com/interviewstreet/hiring-agent
cd hiring-agent

python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 本地模型
ollama pull gemma3:4b

# 配置
cp .env.example .env
# 编辑 LLM_PROVIDER / DEFAULT_MODEL / GEMINI_API_KEY / GITHUB_TOKEN
```

### 代码示例

```bash
# 跑一次评估
python score.py /path/to/resume.pdf

# 输出样例（节选）
# 候选人：张三
# open_source:      8.5/10  (证据：3 个高 star 仓库，author commit ≥ 50)
# self_projects:    7.0/10  (证据：side project 含完整 CI/CD)
# production:       6.0/10  (证据：描述中有 "日活 10k" 但缺指标量化)
# technical_skills: 8.0/10
# bonus:  +0.5 (Kubernetes 实战) +0.3 (写过技术博客)
# deduction: -0.5 (简历与 GitHub 提交时间有 8 个月 gap)
# 总分: 7.3/10
```

### 评价

**优点**：
- 完全可本地跑（Ollama），候选人简历数据不外发
- "证据-分数"绑定，可解释性远胜于传统 ATS 黑盒评分
- fairness 约束内嵌在 prompt，避免 LLM 偏见泄露
- 评分维度（open_source / self / production / technical）+ bonus/deduction 结构合理

**缺点**：
- 对英文简历 + GitHub 强的人友好，对纯国内环境（无 GitHub、无英文项目）评分维度会失真
- PDF 解析依赖 PyMuPDF，复杂排版（双栏、图标密集）会出错
- LLM 评分本身有"中心化倾向"——同一份简历多次评估可能有 ±0.3 浮动

**推荐指数**：⭐⭐⭐⭐（HR/技术招聘团队的"第一轮过滤器"，建议人工复审兜底）

---

## 10. kunchenguid/no-mistakes — `git push` 的"质量门"代理

### 核心功能

`no-mistakes` 把自己塞在**本地 git remote 和真实 remote 之间**——你 `git push no-mistakes` 而不是 `git push origin`，它会在一个**隔离的 worktree**里跑 AI 驱动的验证流水线（review → test → docs → lint → push → PR → CI），每步都过了才把分支转推到配置的真实 remote，并自动开 PR。

核心特性：

- **Non-blocking**：流水线跑在独立 worktree，**你手头的工作不会被中断**
- **Agent-agnostic**：Claude / Codex / Rovodev / OpenCode / Pi / ACP 都能驱动（通过 acpx）
- **Agent-native**：`/no-mistakes` 技能让 coding Agent 直接"做完任务 → 自动过门"，或者"对已 commit 的工作过门"
- **Human stays in charge**：auto-fix 或 review 发现，由你决定
- **Clean PRs by default**：push、开 PR、盯 CI、auto-fix 失败，一条龙

每个 step 要么独立通过、要么停在 finding 让你处理：
- **safe mechanical fixes**（拼写、formatter、依赖升级）自动应用
- **触碰意图的修复**（API 设计、错误处理逻辑）升级给你审批

任何东西在所有 check 全绿之前**不会**到达配置好的 push target。

### 适用场景

- **个人项目副业**：怕"手抖"推到 main / 公共仓库
- **AI 编程 Agent 用户**：让 Agent 跑出来的 diff 在你审之前先过一遍 lint/test
- **团队 code review 减负**：基本 lint/test/docs 在 push 阶段就过滤掉
- **合规场景**：所有 PR 必须经过特定 pipeline，缺一不可

### 安装方式

```bash
# macOS / Linux 一行
curl -fsSL https://raw.githubusercontent.com/kunchenguid/no-mistakes/main/docs/install.sh | sh

# Windows / Go install / build-from-source 见官方安装指南
# https://kunchenguid.github.io/no-mistakes/start-here/installation/

# 初始化
cd my-repo
no-mistakes init
# 会自动：
# - 注册 no-mistakes remote
# - 安装 /no-mistakes 技能到 user-level
# - 打印 push 提示
```

### 代码示例

```bash
# 改完代码
git checkout my-branch
# ... do some work ...

# 不直接 push origin，先 push 给门
git push no-mistakes
 * Pipeline started
   Run no-mistakes to review.

# 打开 TUI 看 finding + 决定 auto-fix / approve / skip
no-mistakes

# 全部 green 后，no-mistakes 自动转发到 origin 并开 PR
# → 你再也不用手写 PR body、再也不用跑两轮 CI
```

```bash
# 让 Agent 端到端接管
> /no-mistakes 实现 OAuth2 PKCE 流程
# Agent：写代码 → 跑门 → 通过 → 推送 → 开 PR → 通知你
```

### 评价

**优点**：
- 把"AI Agent 时代 PR 质量参差"的痛点直接产品化
- 隔离 worktree 跑流水线，**不影响开发者主工作流**——这是关键设计
- 区分"safe auto-fix" vs "intrusive change"，人工始终兜底
- 跨 Agent（Claude/Codex/OpenCode/Pi/ACP）兼容，避免锁定

**缺点**：
- 仍是 0.x，finding 分类/auto-fix 规则可能在演进
- 需要在本地装一个 long-running 的门代理，对"完全 minimal 工具链"的用户是 overhead
- 强依赖 CI 在 push target 上跑得通，没 CI 的 repo 收益会打折扣

**推荐指数**：⭐⭐⭐⭐（"重度 AI 编程 + 怕手抖"用户的实用工具）

---

## 写在最后

今天榜单最值得关注的三件事：

1. **Apple 官方出容器、Google 官方出设计契约**——大厂把 AI 时代的"基础设施"一件件补齐，Agent-Native 工具链正在从"个人项目"变成"工业标准"
2. **Agent-orchestrator 工具继续分化**：Orca 做多 Agent 桌面编排、Harness 做团队架构工厂、Orca-IDE/Hermes/Rev-Archon 各自圈地——下半年值得观察"赢家"
3. **垂直场景继续出爆款**：视频（OpenMontage）、简历（hiring-agent）、股票（daily_stock_analysis）、Android Auto（headunit-revived）——LLM 越成熟，垂直 SaaS 越被开源项目"反向绞杀"

> **风险提示**：本文涉及 `daily_stock_analysis` 的内容仅为开源工具介绍，不构成任何投资建议。请独立判断、谨慎决策。
> **数据来源**：[github.com/trending](https://github.com/trending) 当日榜单 + 各项目官方 README
> **抓取时间**：2026-06-25 09:14 (Asia/Shanghai) / 2026-06-25 01:14 (UTC)
> **上一期**：[每日开源速递 - 2026-06-22](daily-digest-2026-06-22.md)
