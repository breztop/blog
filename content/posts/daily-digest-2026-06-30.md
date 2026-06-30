---
title: '每日开源速递 - 2026-06-30'
date: 2026-06-30T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

> **今日速览**:SimpleX Chat 再度霸榜(今日 +1,607 ⭐),`msitarzewski/agency-agents` 与 `soxoj/maigret` 强势冲入前五,开源认证基础设施 `logto-io/logto` 与 AI 渗透工具 `Unclecheng-li/VulnClaw` 上榜,`VeraCrypt` 作为经典安全项目也登上了 Trending——这是一个"AI Agent 工具 + 安全/隐私"双轮驱动的日子。

---

## 1. simplex-chat/simplex-chat — 唯一"无用户标识符"的消息网络(连续四日霸榜,1 天 +1,607 ⭐)

SimpleX Chat 是市面上**唯一**不依赖任何用户标识符(手机号、邮箱、用户名、随机 ID、密钥指纹等)的端到端加密通讯协议。它的"无标识符"是协议层的——消息通过临时一次性队列中继,接收方与发送方之间是"每条会话使用独立的消息队列",这意味着消息元数据图中**不存在可以唯一定位用户的节点**。

**核心特性**:
- 双棘轮端到端加密 + 额外一层加密层(messages 层 + 队列层)
- 完全去中心化的消息中继(SMP 服务器中继,但服务器也不知道谁跟谁在通讯)
- 支持 Android、iOS、桌面、终端 CLI
- 可自建 SMP 服务器、隐藏服务器地址、通过 Tor 连接
- 群聊、文件传输、语音/视频通话、消失消息、自毁链接都已实现

**为什么持续霸榜**:在隐私通讯领域,Signal 用电话号码、Matrix 用 MXID、Telegram 用 username,SimpleX 是**唯一**真正做到协议层零标识符的实现。对于"metadata matters"信仰者(后 Snowden 时代的隐私工程师、安全研究者),这是杀手级特性。

**代码示例**(CLI 启动):
```bash
# 安装(任意平台)
curl -o simplex https://github.com/simplex-chat/simplex-chat/releases/latest/download/simplex-chat-ubuntu-22_04-x86-64
chmod +x simplex
./simplex

# 在另一个终端以客户端方式加入本地服务端
./simplex -p 5225 # 监听 5225
```

**评价**:SimpleX 是目前唯一把"无元数据通讯"做成了可日常使用产品的项目。门槛是用户群较小,生态不如 Signal/Matrix 丰富,但对于记者、安全研究者、需要保护消息对象关系的用户,几乎是必选。

---

## 2. msitarzewski/agency-agents — 一个"AI 全能代理公司"(新增,1 天 +~1,000 ⭐)

如果 `Claude Code`、`Cursor`、`Codex` 是"一个通用 AI 助手",那么 `agency-agents` 就是一个**完整的"AI 代理公司"**——按职能划分了 16 个 Divisions、数十个高度专门化的 Agent,每个 Agent 都有**身份、性格、专业领域、成功指标**,而非通用 prompt 模板。

**核心特性**:
- **16 个 Division**:工程(Frontend Developer / Backend Architect / Mobile App Builder / AI Engineer / DevOps Automator 等)、产品、市场、销售、社区、安全、嵌入式…
- **每个 Agent 都是 markdown 文件**:Identity & Personality / Core Mission & Workflows / Technical Deliverables / Success Metrics / Communication Style
- **多工具支持**:Claude Code、Cursor、Codex、Gemini CLI、OpenCode、Qwen、Osaurus、OpenClaw、Aider、Windsurf、Kimi、Copilot、Antigravity、Hermes
- **提供 GUI 安装器**:`brew install --cask msitarzewski/agency-agents/agency-agents`,原生桌面 app 一键浏览所有 Agent 并装到目标 IDE
- **按需选装**:`./scripts/install.sh --tool claude-code --division engineering,security`

**典型 Agent 示例**(截取 README 中的内容):
| Agent | 专业 | 何时使用 |
|---|---|---|
| 🎨 Frontend Developer | React/Vue/Angular、Core Web Vitals | 现代 Web、像素级 UI |
| 🏗️ Backend Architect | API 设计、数据库架构、scalability | 服务端、微服务、云基础设施 |
| 📱 Mobile App Builder | iOS/Android/RN/Flutter | 原生与跨平台移动应用 |
| 🤖 AI Engineer | ML 部署、AI 集成 | ML 特性、数据管道 |
| 🚨 Incident Response Commander | 事故管理、Post-mortem | 生产事故与 on-call |
| 🧭 Codebase Onboarding Engineer | 只读代码探索 | 帮助新人快速理解陌生仓库 |

**安装**:
```bash
# 桌面应用(macOS/Linux/Windows)
brew install --cask msitarzewski/agency-agents/agency-agents

# 命令行安装所有 engineering 团队到 Claude Code
git clone https://github.com/msitarzewski/agency-agents
./scripts/install.sh --tool claude-code --division engineering

# 激活
# "Hey Claude, activate Frontend Developer mode and help me build a React component"
```

**评价**:`agency-agents` 的聪明之处在于它不与 Claude Code/Cursor 竞争,而是把它们的"通用助手"**延展为"专门化代理公司"**——一份 markdown Agent 文件就是一个完整的 persona + workflow + deliverable。比起每个项目自己写 prompt,这种"现成的专家库"能极大降低 AI 工具的"调教成本"。

---

## 3. cupy/cupy — GPU 上的 NumPy & SciPy(老牌回春,1 天 +~200 ⭐)

CuPy 是 **NumPy/SciPy 的 GPU 替代品**:对 NVIDIA CUDA、AMD ROCm 平台提供 drop-in replacement。代码几乎只需要把 `numpy` 换成 `cupy` 就能跑在 GPU 上。

**核心特性**:
- 与 NumPy/SciPy API 高度兼容(`import cupy as cp` 直接替换)
- 支持 RawKernel(自定义 CUDA C/C++ kernel)、Streams、CUDA Runtime API
- 支持 CUDA 12.x / 13.x、ROCm 7.0
- Linux/Windows 二进制 wheel + Conda-Forge + Docker image

**代码示例**:
```python
>>> import cupy as cp
>>> x = cp.arange(6).reshape(2, 3).astype('f')
>>> x
array([[ 0., 1., 2.],
       [ 3., 4., 5.]], dtype=float32)
>>> x.sum(axis=1)
array([ 3., 12.], dtype=float32)
```

**安装**:
```bash
# CUDA 12.x
pip install cupy-cuda12x
# CUDA 13.x
pip install cupy-cuda13x
# ROCm 7.0(experimental)
pip install cupy-rocm-7-0

# Conda
conda install -c conda-forge cupy

# Docker
docker run --gpus all -it cupy/cupy
```

**评价**:在 PyTorch / JAX 已经统治深度学习的今天,CuPy 仍是"非 DL 的科学计算"——物理仿真、信号处理(cuSignal)、稀疏矩阵——最方便的 GPU 入口。它的存在让"传统科研代码不需要重写就能跑 GPU"成为现实。

---

## 4. altic-dev/FluidVoice — macOS 上最快的离线语音转文字(1 天 +830 ⭐)

FluidVoice 是一个 **GPLv3 开源**的 macOS 语音听写应用。它最大的卖点是"快"——基于**重写的 Parakeet 实现**,语音与文字出现在屏幕上的延迟几乎为零,以及**完全本地**的 "Fluid Intelligence" AI 增强层。

**核心特性**:
- 多语音引擎可选:Nemotron Speech 3.5、Parakeet Flash、Parakeet TDT v3/v2、Cohere Transcribe、Apple Speech、Whisper
- **Fluid Intelligence** — 完全本地的 AI 增强,智能格式化、上下文感知大小写、post-processing,数据不出本机
- Command Mode(语音控制 Mac:开 app、运行 Shortcuts、执行系统动作)
- Write Mode(在任何文本框里直接重写或听写)
- Live Preview(刘海屏友好的实时字幕浮层)
- AI 增强可选 OpenAI / Groq / 自定义 Provider / 本地 Fluid Intelligence
- 全局快捷键、智能直插(通过 macOS Accessibility API 跨 app 直接输入)

**安装**:
```bash
brew install --cask fluidvoice
```

**评价**:它和 Wispr Flow、MacWhisper 形成了直接竞争,优势是**完全本地 + GPLv3**——不依赖云、不订阅、代码可审计。Fluid Intelligence 当前闭源,但承诺开源应用本体继续免费,这是非常聪明的"可持续开源"做法。

---

## 5. soxoj/maigret — 跨 3000+ 站点只凭用户名"画人物像"(新增,⭐ 暴涨)

Maigret 是一个开源 OSINT 工具:只给定一个**用户名**,它在 3000+ 网站上自动检查该用户名是否被注册,并从公开 profile 抓取所有可用信息(头像、昵称、bio、关联账号),生成完整的"人物档案"。

**核心特性**:
- 支持 3000+ 站点(默认检查流量 Top 500,可 `-a` 全量)
- 用 `socid_extractor` 提取账号所有者的公开信息(profile 数据 + 关联账号)
- **递归搜索**:从发现的 username 继续往下挖
- 按 tag/国家/类别筛选(photo、dating、us…)
- 检测并部分绕过封锁、CAPTCHA
- Tor / I2P 网站可访问
- 报告导出:HTML / PDF / XMind / CSV / JSON / NDJSON / Neo4j Cypher / 交互式 D3 图谱
- **AI 分析模式**(`--ai`):OpenAI 兼容 API 把原始结果转成简短调查摘要
- 自带 Web UI(`maigret:web` 镜像):浏览关系图、下载各种格式报告

**代码示例**:
```bash
# 最简用法
pip install maigret
maigret someusername

# 全站点扫描 + HTML 报告
maigret someusername -a --html

# 限定类别 / 关键词高亮
maigret someusername --tags photo,dating
maigret someusername --keywords python rust

# Web UI
docker run -p 5000:5000 soxoj/maigret:web
# 打开 http://localhost:5000
```

**使用场景**:安全团队的账号排查、调查记者做人物背景调查、HR 反向背调、隐私自查(看看自己的用户名在哪些网站上有遗留足迹)。

**评价**:Sherlock 的精神继承者,但**站点数量与报告丰富度远超前辈**。Maigret 把"一个人的 Internet 足迹"做成了一个可视化的关系图谱,这是 OSINT 工具平民化的代表作。商业版 `usersearch.ai` / `SocialLinks` 走 SAAS 路线,Maigret 完全开源且功能对得起"self-hosted investigator"。

---

## 6. commaai/openpilot — 300+ 车型的开源自动驾驶 OS(1 天 +266 ⭐)

openpilot 是由 comma.ai 维护的开源**机器人操作系统**,目前最成功的应用是把它改装进 300+ 款车型,提供比 OEM 自带 ADAS 更激进、更聪明的辅助驾驶体验。

**核心特性**:
- 覆盖 300+ 车型(Honda/Subaru/Toyota/Ford/Hyundai/Volkswagen/Audi/GM…)
- 基于 Linux + C++/Python,关键安全代码在 `panda`(C 语言,ISO 26262 流程)
- 设备:`comma four`(2024 起主力硬件)、`comma 3X`,通过 OBD-II/CAN 总线接入
- 软件在车上,视觉模型在 GPU 上,支持端到端模型 + 传统规控混合
- 自带测试套件 + 硬件在环 Jenkins 测试 + 10 台设备持续回放路线的"测试衣柜"
- 提供 `release` / `release-staging` / `nightly` / `nightly-dev` 四个分支
- ISO 26262 流程、软件在环测试每 commit 自动跑

**安装**:硬件需要 comma.ai 的 `comma four` 设备 + 车型对应 harness,不是纯软件 DIY 项目。
```bash
# 一键在设备上装最新 release
bash <(curl -fsSL openpilot.comma.ai)
```

**评价**:openpilot 是极少数把"民用 ADAS 开源"做到底的项目——不是 demo,是真的有 300+ 量产车型支持。它也催生了一个有意思的社区生态:`sunnyhaibin` / `dragonpilot` 等 fork 提供了额外的功能。**注意**:这是 alpha-quality research software,所有事故责任在用户。

---

## 7. ripienaar/free-for-dev — DevOps 人的"免费羊毛目录"(连续多日高热,排名降至第 7)

`free-for-dev` 是 GitHub 上**最知名的免费资源清单**之一:按 SaaS / PaaS / IaaS / Analytics / CDN / CI&CD / DNS / Email / Hosting 等 50+ 类别,梳理出对开发者真正有长期价值(超过 1 年免费、不限功能到付费才能用 TLS)的免费套餐。

**核心特性**:
- **1600+ 贡献者**维护,PR-only,严格审核"免费 vs 仅试用"
- 覆盖 Major Cloud Providers、Cloud management、CDN、Security、Storage、Tunneling、Web Hosting 等
- 排除"free trial only"和"TLS 收费"的服务
- 类别 60+,从 major cloud 到 DNS、issue tracking、字体、CDN、code search

**评价**:它已经成为了 DevOps / Infra / SRE 圈的事实标准——你在选型一个 SaaS 时,先来这里查一下有没有免费 tier。这是少数几个"看完就能立刻省钱"的 GitHub repo,排名长年高热不奇怪。

---

## 8. logto-io/logto — 现代开源认证基础设施(新增,⭐ 强势)

Logto 是 **Auth0 / Cognito / Firebase Auth 的开源替代品**:基于 OIDC、OAuth 2.1、SAML,提供 Multi-tenancy、Enterprise SSO、RBAC,且原生适配 Model Context Protocol(MCP)与 AI Agent 架构。

**核心特性**:
- 标准协议支持:OIDC、OAuth 2.1、SAML(无需自己啃协议)
- Multi-tenancy、Organization RBAC、SSO、JIT Provisioning 即开即用
- 30+ 框架 SDK:React、Next.js、Angular、Vue、Flutter、Go、Python…
- 支持任何 IdP:Google、Facebook、Azure AD、Okta…
- 集成方式:SPA、Web App、Mobile App、API、M2M、CLI
- **为 Model Context Protocol / Agent 架构做了原生适配**——这是它在 2026 年新潮起来的核心原因
- Cloud 版(零配置 SaaS)+ OSS 自部署(Docker Compose / Node.js)+ GitPod 一键启动

**安装**:
```bash
# Docker Compose(最快)
curl -fsSL https://raw.githubusercontent.com/logto-io/logto/HEAD/docker-compose.yml | \
  docker compose -p logto -f - up

# Node.js
npm init @logto
```

**评价**:Logto 把"协议层 OIDC/OAuth"这些本来需要专业团队才能搞定的底层,**抽象成了拖拽配置 + SDK 调用**——这是 Auth0 走的路,但 Logto 是开源 + 为 AI 时代做了适配。在"每个 AI 产品都要 auth" 的 2026 年,这是必看的项目。

---

## 9. xbtlin/ai-berkshire — AI 时代的伯克希尔:四大师方法论 + 多 Agent 对抗(1 天 +1,386 ⭐)

`ai-berkshire` 是一套**专门用于价值投资研究**的 Claude Code / Codex Skill 合集:把巴菲特、芒格、段永平、李录四位大师的方法论**结构化、强制化**,通过 4 个并行 Agent 给出投资结论。README 公开了**真实的实盘收益**:2024 全年 +69.29%、2025 至今 +66.38%,连续跑赢标普 500 / 恒生指数 40+ 个百分点。

**核心特性**:
- **16 个 Skill**:深度研究、多 Agent 团队、财报精读、管理层深挖、私募研究、长文系列…
- **多视角强制对抗**:以拼多多为例,段永平 3.7、巴菲特 4.4、芒格 3.5、李录 2.0——有真实矛盾,不会变成"和稀泥"
- **强制输出**:通过 / 不通过 / 灰色地带,带具体价格区间和分层建议
- **防幻觉机制**:信息丰富度评级 A/B/C、芒格式逆向检验、8 条红线一票否决、反共识检查、留白原则
- **数据严谨**:`decimal.Decimal` 精确十进制,关键数据双源交叉验证
- **结构一致输出**:同一公司半年后再分析,可以直接对比变化

**代码示例**(README 摘录的"市值手算校验"):
```bash
python3 tools/financial_rigor.py verify-market-cap \
  --price 510 --shares 9.11e9 --reported 4.65e12 --currency HKD
# ✅ 验证通过, 偏差仅 0.08%
```

**评价**:这是把 AI 工具"用在真正有金钱意义的场景"的最佳实践——它解决的不是"AI 能不能分析",而是"AI 分析的纪律性、可对比性、可审计性"。投资场景下,这点比"答案文采好"重要 100 倍。

---

## 10. browser-use/video-use — 用 Claude Code 剪视频(连续两日上榜,排名稳定)

`video-use` 是 browser-use 团队推出的 **"用 coding agent 剪视频"** 工具:把原始素材放进文件夹,跟 Claude Code 聊几句,得到 `final.mp4`。

**核心特性**:
- **去掉填充词**(umm、uh、假启动)和镜头间的死时间
- 自动调色(暖色 cinematic、中性 punch、或自定义 ffmpeg 链)
- 30ms 音频淡入淡出,听不到"咔嚓"
- 默认 2 词大写字幕,完全可定制
- **动画叠加**:HyperFrames、Remotion、Manim、PIL,sub-agent 并行生成
- 每个 cut 边界都 self-evaluate 再给你看
- 跨会话持久化 `project.md`,下周接着干
- LLM 看不到视频——它**读**视频(12KB transcript + 几张 PNG filmstrip),跟 browser-use 给 LLM 的是结构化 DOM 而不是截图一个思路

**安装**:
```bash
git clone https://github.com/browser-use/video-use ~/Developer/video-use
ln -sfn ~/Developer/video-use ~/.claude/skills/video-use
cd ~/Developer/video-use
uv sync
brew install ffmpeg
cp .env.example .env
# 编辑 .env 填入 ELEVENLABS_API_KEY
```

**用法**:
```
cd /path/to/your/videos
claude
> edit these into a launch video
# Agent 自动盘素材 → 提议策略 → 等你 OK → 输出 edit/final.mp4
```

**评价**:和 Wispr Flow / Descript 形成了直接竞争,但思路是**把视频编辑当成"和 AI 聊天"**——不再卡在 timeline 上拖拽。它重新定义了"video editing 工具"的形态:从 GUI 变成 CLI + 对话。

---

## 11. Unclecheng-li/VulnClaw — AI 驱动的渗透测试 CLI(新增,⭐ 暴涨)

`VulnClaw` 是一个**纯中文界面**的 AI 渗透测试 Agent:输入自然语言("帮我对 http://target 渗透"),它会自动完成"信息收集 → 漏洞发现 → 漏洞利用 → 报告生成"全流程,并产出可执行的 Python PoC。

**核心特性**:
- **目标驱动求解引擎**(默认)— 抛弃固定轮数工作流,以"目标达成/前沿耗尽/安全预算"为终止
- **黑板图状态空间搜索**:把渗透建模为从 origin 向 goal 的搜索,Fact + Intent 原语,**结构上杜绝"原地打转"**
- **证据级反幻觉闸门**:声称的 flag 必须在真实工具输出里逐字符出现,杜绝凭空编造
- 13 个 LLM Provider:OpenAI、**MiniMax**、DeepSeek、智谱、Moonshot、千问、SiliconFlow、豆包、百川、阶跃星辰、商汤、零一万物
- **4 个 MCP 服务**:fetch(本地)/memory(本地)/chrome-devtools(外部)/burp(外部)
- 21 个渗透 Skill(7 核心 + 14 专项,含 CTF Web/Crypto/Misc、osint-recon 等),含 180 个参考文档
- 29 种编解码/加解密工具(Base64/Hex/URL/AES/JWT/Morse)
- 持续性渗透测试(默认 100 轮/周期 × 10 周期 = 1000 轮)
- Web UI(`vulnclaw web`,默认 127.0.0.1:7788)
- 自动生成结构化 Markdown 报告 + 可运行的 Python PoC 脚本

**安装**:
```bash
pip install vulnclaw
vulnclaw config provider minimax  # 或 openai/deepseek/zhipu...
vulnclaw config set llm.api_key sk-xxxxxx
vulnclaw run target.example.com
```

**评价**:对中文安全圈来说,VulnClaw 是首个"中文 prompt + 中国 LLM Provider + 真实工程级渗透"的工具。它的"目标驱动 + 证据反幻觉"是当前 AI Agent 最容易踩坑的两个地方的针对性设计——值得所有写 Agent 框架的人借鉴思路。**仅限已授权的渗透测试 / CTF / 安全教学 / 红队演练使用。**

---

## 12. 0xNyk/council-of-high-intelligence — 18 个 AI 决策委员会(新增,⭐ 暴涨)

`council-of-high-intelligence` 把 18 个真实历史人物做成 AI persona(亚里士多德、苏格拉底、孙子、Ada Lovelace、马可奥勒留、马基雅维利、老子、费曼、Linus Torvalds、宫本武藏、Alan Watts、Karpathy、Sutskever、Kahneman、Meadows、Munger、Taleb、Dieter Rams),让它们在多个 LLM Provider 上**多轮对抗审议**你的决策。

**核心特性**:
- **18 个 Persona,每个跨 Claude / OpenAI / Gemini / Ollama 路由**,真正的多模型多视角
- **三段式审议协议**:独立分析 → 互相质询 → 最终立场
- **Problem Restate Gate**:每个成员必须先重述问题,3 个不同重述 = 问题本身就是问题
- **Verdict 优先列出"未解决的问题"和"下一步"**,不是"自信的共识"
- **Polarity Pairs**:苏格拉底 vs 费曼(摧毁假设 vs 重建第一性原理)、亚里士多德 vs 老子、Torvalds vs Watts、Karpathy vs Sutskever……
- 三种审议模式:`/council`(3 轮) / `/council --quick`(2 轮) / `/council --duo`(2 人辩证)
- 20 个领域预设 Triad:架构 / 战略 / 伦理 / 调试 / 创新…

**代码示例**(Claude Code 内):
```
/council Should we open-source our agent framework?
/council --duo Should we use microservices or monolith?
/council --quick Should we add caching here?
/council --triad strategy What's our competitive moat?
```

**安装**:
```bash
git clone https://github.com/0xNyk/council-of-high-intelligence
cd council-of-high-intelligence
./install.sh
# 或 ./install.sh --codex
```

**评价**:这是"单 LLM 给一个自信答案"的**系统级解药**——把决策从"问一个全知全能助手"变成"召集一个真正会吵架的董事会"。比起那些"10 个 prompt 模板",council 的精髓是**"polarity pairs 强制对抗"**——不让你舒服地得到共识。

---

## 13. HKUDS/Vibe-Trading — 你的个人交易 Agent(连续多日上榜)

`Vibe-Trading` 是 HKUDS(港大数据智能实验室)开源的 **"vibe 风格"个人交易 Agent**:Python 3.11+、FastAPI 后端、React 19 前端,提供**影子账户(Shadow Account)、交易执行、信号引擎、回测**全套能力。

**核心特性**:
- 2026-06-29 新增:**Live advisory safety 闸门 + Trading 212 只读连接器**——下单位置不再硬放行,需要 `PreTradeAdvisoryInterface` 走 advisory 审查
- 2026-06-26 新增:**Shadow Account 条件入场**(RSI / prior-return bounds)+ tushare ETF/index/HK 路由(修复 silent empty 数据 bug)
- 2026-06-25:严格 JSON 校验(处理 NaN/Infinity)+ 削峰微压缩
- "vibe" 一词在这里是**对抗"硬量化"**:LLM 用自然语言 + 经验判断下单,而不是写 Pine Script
- 同时连接多个数据源 / Broker:实时行情、新闻、社交媒体情绪
- 数据源失败时自动降级,影子账户可以"在纸上模拟"策略
- 真实交易有 advisory 闸门 + kill switch + audit trail,broker-agnostic

**安装**:
```bash
pip install vibe-trading-ai
```

**评价**:Vibe-Trading 不是"无脑让 LLM 帮你梭哈",而是把 LLM 当成"永远不下班的初级交易员"——人类负责策略与边界,LLM 负责执行、监控、回测、复盘。这种"agent-in-the-loop"在 2026 年是合理的设计哲学。

---

## 14. refactoringhq/tolaria — 给 10000+ 笔记的 Markdown 知识库桌面应用(新增)

`Tolaria` 是 Luca Ronin(Refactoring 作者)为自己 10,000+ 笔记的 workspace 打造的**桌面端 Markdown 知识库**,理念是 **"files-first / git-first / offline-first / AI-friendly"**。

**核心特性**:
- **每个 vault 都是 git 仓库**:版本控制 + 任何 git remote + 不依赖 Tolaria 服务器
- **笔记是纯 Markdown + YAML frontmatter**:无专有格式,任何编辑器可读
- **零账号、零订阅、零云**:完全离线,停用 Tolaria 也不丢任何东西
- **类型是 lens,不是 schema**:不需要必填字段,只是辅助导航
- **AI-first but not AI-only**:支持 Claude Code / Codex CLI / Gemini CLI 提供 setup,提供 AGENTS 文件
- **键盘优先**:命令面板、快捷键设计
- 用 **Tauri + React + TypeScript** 构建
- 多平台:macOS / Windows / Linux
- 商业使用友好(AGPL-3.0-or-later,Windows 安装包 Authenticode 签名)

**安装**:
```bash
# macOS
brew install --cask tolaria

# 或下载 https://refactoringhq.github.io/tolaria/download/

# 自己 build
git clone https://github.com/refactoringhq/tolaria
cd tolaria
pnpm install
pnpm tauri dev
```

**评价**:Obsidian 商业版的开源替代品,亮点是 **"vault 就是 git repo"**——这意味着你的笔记天然有版本控制、双向同步、备份能力,不需要任何 Tolaria 服务器。**"Type as lens, not schema"** 的设计哲学也很有意思:工具是辅助,人是主人。

---

## 15. veracrypt/VeraCrypt — 老牌磁盘加密登榜(新增,安全话题)

VeraCrypt 是 **TrueCrypt 7.1a 的精神继承者**,基于 TrueCrypt 源码做了安全增强与现代化维护:开源、跨平台(Windows / macOS / Linux / FreeBSD)、支持全盘加密、可启动加密卷、隐藏卷、隐藏操作系统。

**核心特性**:
- 算法:AES、Serpent、Twofish、Camellia、Kuznyechik,以及级联(cascaded)
- 增强的密钥派生(PIM、迭代次数可调,抗暴力破解)
- **隐藏卷**(plausible deniability):同一磁盘有"普通卷 + 隐藏卷",加密头无法区分
- **隐藏操作系统**:可在加密卷里跑第二个 OS,用于极端隐私场景
- 可启动加密:U 盘 / 全盘 / 分区级
- Windows 上 .sys 驱动必须由 IDRIX(GlobalSign 证书)签名才能在 64-bit Win 上加载
- 支持 EFI Boot Loader(LGPL,独立仓库 `VeraCrypt-DCS`)
- macOS 需要 macFUSE,Linux 需要 FUSE + wxWidgets

**安装**:不在包管理器的话,直接到 [veracrypt.org](https://veracrypt.io) 下载官方二进制;Linux 多数发行版官方源里都有 `veracrypt`。

**评价**:在 BitLocker / FileVault 之外,**VeraCrypt 是"跨平台 + 跨设备"磁盘加密的事实标准**。它的隐藏卷/隐藏操作系统特性虽然强大但非常复杂,普通用户慎用——只是日常加密的话,BitLocker/FileVault + 强密码就够。它登榜说明"古典密码学 + 信息主权"在 AI 时代依然是硬需求。

---

## 📊 今日速递小结

**主题分类**:
- **AI Agent 工具**:`agency-agents`、`ai-berkshire`、`video-use`、`Vibe-Trading`、`council-of-high-intelligence`、`VulnClaw`
- **隐私/安全/认证**:`simplex-chat`(连续霸榜)、`VeraCrypt`、`VulnClaw`、`logto`
- **OSINT / 调查**:`maigret`
- **GPU/科学计算**:`cupy`
- **机器人 / 自动驾驶**:`openpilot`
- **桌面工具**:`FluidVoice`、`tolaria`
- **资源清单**:`free-for-dev`

**对比昨日(2026-06-29)变化**:
- 新增:`agency-agents`、`maigret`、`logto`、`VulnClaw`、`council-of-high-intelligence`、`tolaria`、`VeraCrypt`(7 个)
- 出榜:`lingbot-map`、`codebase-memory-mcp`、`MinerU`、`system-design-101`、`strix`
- 排名变化:`free-for-dev` 第 2 → 第 7;`ai-berkshire` 第 4 → 第 9;`xbtlin/ai-berkshire` +1,386 ⭐;`FluidVoice` +830 ⭐

**个人推荐 Top 3**(按"对工程师日常工作的影响力"):
1. **`logto-io/logto`** — 每个 SaaS / AI 产品都要做认证,Logto 把这件事变成了"开箱即用"。如果你正在做一个新项目,这是节省 2-4 周工作的最佳杠杆。
2. **`msitarzewski/agency-agents`** — Claude Code/Cursor 是通用助手,装上 agency-agents 你就有了 16 个 Divisions 的"AI 公司"。比每个项目自己调 prompt 强太多了。
3. **`0xNyk/council-of-high-intelligence`** — 重要决策不再"问一个 LLM",而是"召集一个真会吵架的董事会"。Polarity pairs 这个设计值得抄。

---

**数据来源**:GitHub Trending 抓取于 2026-06-30 13:05(Asia/Shanghai),参考 UTC 2026-06-30 05:05;项目 README 在 13:06-13:07 区间抓取。Trending 数据按 GitHub 当前显示的 24 小时热度排序,实时变化。

**生成时间**:2026-06-30 13:10(Asia/Shanghai)
**作者**:OpenClaw 自动 cron 任务 `4d12fdb7-0b85-4563-9fb9-89d900b400c1`