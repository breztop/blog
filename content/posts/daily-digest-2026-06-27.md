---
title: '每日开源速递 - 2026-06-27'
date: 2026-06-27T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026年6月27日

今日 GitHub Trending 呈现出一种罕见的"硬核基建回归"信号：`simplex-chat` 把"无用户标识符"的隐私通讯做到 iOS/Android/桌面三端齐全，单日再涨 432 星；`commaai/openpilot` 作为面向 300+ 款车型的开源驾驶辅助操作系统持续高热；`opendatalab/MinerU` 的 3.4 版本升级 PP-OCRv6 让 OCR 处理速度直接翻倍，是 RAG/Agent 工程师的必看工具；`grafana` 老牌项目依然稳坐观测性领域头牌；`xbtlin/ai-berkshire` 则是 AI 投研领域少见的、把"四大师方法论 + 多 Agent 对抗"做成可执行 Skill 的实战项目。下面让我们逐个深入了解。

> 数据来源：[github.com/trending](https://github.com/trending)（每日 UTC 榜单）
> 抓取时间：2026-06-27 14:14 (Asia/Shanghai) / 2026-06-27 06:14 (UTC)

---

## 1. simplex-chat/simplex-chat — 第一款"无用户标识符"的隐私通讯网络

### 核心功能

`SimpleX Chat` 是目前唯一一个**完全不依赖任何用户标识符**（手机号、用户名、随机 ID、密钥、邮箱全部没有）的开源即时通讯网络。它的核心思想是：用户身份本身就不应该出现在协议里——既不在消息中，也不在元数据里。它使用 **Double Ratchet 算法**做端到端加密，并在此之上再加一层额外的加密层（out-of-band 密钥协商 + 每条消息独立密钥）。即便服务器被攻破，攻击者也只能看到一堆加密块，无法把消息关联到任何"用户"。

平台覆盖极其完整：

- **iOS / Android** 移动 App（App Store / Google Play + APK）
- **macOS / Windows / Linux** 桌面客户端
- **终端 CLI**（也叫 `simplex-chat`，Haskell 编写，跨平台）
- **TestFlight** 预览通道（领先正式版 1-2 周收到新功能）
- **自托管 SMP 服务器**：你可以完全自己跑消息中继

值得一提的还有 **SimpleX Directory** —— 用户共建的群组目录，以及 **SimpleX Address**（一种可重置的一次性链接），让分享联系方式这件事本身也不再成为泄漏面。

### 适用场景

- **记者 / 律师 / 医生** 等需要为客户身份保密的职业通讯
- **隐私极客 / 安全研究者** 评估去中心化通讯协议
- **企业内部高敏对话**：自托管 SMP 后，元数据无法被任何第三方（包括运营商）看到
- **跨境通讯**：协议不依赖任何国家身份基础设施，规避审查/封锁风险
- **Signal 替代品**：厌倦了"绑定手机号"的所有人

### 安装方式

```bash
# 桌面端（macOS / Linux / Windows）从 GitHub Releases 下载
# 移动端：App Store / Google Play 搜 "SimpleX Chat"

# 终端版（开发者首选，Haskell 实现）
curl -fsSL https://raw.githubusercontent.com/simplex-chat/simplex-chat/stable/scripts/cmd-zap.sh | sh

# 或 Docker 自托管 SMP 服务器
docker run -d --name smp-server \
  -p 5223:5223 \
  simplexchat/smp-server
```

### 代码示例

SimpleX 的"邀请链接"设计是其精髓——一个 URL 就是完整的"添加联系人"动作：

```
# 在 A 用户的终端
$ simplex-chat
> /connect
# 输出：https://smp4.simplex.im/a#<一次性密钥串>

# 把这个链接发给 B 用户
# B 用户在终端粘贴：
> /connect https://smp4.simplex.im/a#<一次性密钥串>
# 连接建立，所有消息走 E2E 加密
> /help
```

群组、文件传输、语音/视频通话、可撤回消息、消息反应、移动端推送通知全部开箱即用。

### 评价

**优点**：
- 唯一真正做到"协议层零用户标识符"的成熟方案（Signal/Matrix/Wire 都有某种 ID）
- 客户端 + 服务器 + 协议全部开源，可独立审计
- 双层 E2E 加密 + 每条消息独立密钥，密码学设计非常严谨
- 平台覆盖广，CLI + 桌面 + 移动 + Web 五端齐全

**缺点**：
- 用户基数仍远小于 Signal/Telegram，发现"朋友也在用"是当前最大门槛
- Haskell 编写的服务端，自托管对中小团队仍有门槛
- 群组体验相比 Telegram 仍有差距（话题、置顶、频道生态）
- "完全隐私"也意味着"无找回机制"，丢了链接就等于失联

**推荐指数**：⭐⭐⭐⭐⭐（如果你认真在乎元数据隐私，这是当前唯一成熟的开源选择）

---

## 2. google-labs-code/design.md — 给 Agent 一份"看得懂"的设计契约

### 核心功能

`DESIGN.md` 是 Google Labs 推出的**面向编码 Agent 的视觉规范格式标准**——把"设计系统"这件事从 PNG/Style Guide 这种"人看的文档"升级成"Agent 能 lint、能 diff、能持续消费"的结构化契约。一份 `DESIGN.md` 包含两层：

- **YAML front matter**（机器可读）：`colors` / `typography` / `rounded` / `spacing` / `components` 等设计令牌（design tokens），每条都标注类型（Color / Dimension / Typography / Token Reference）
- **Markdown 正文**（人类可读）：Overview / Colors / Typography / Layout / Elevation / Shapes / Components / Do's and Don'ts 八个标准段落，回答"为什么用这个色"和"什么时候不用"

配套的 npm CLI `@google/design.md` 提供三条子命令：

```bash
# 校验 + WCAG 对比度检查 + 结构错误
npx @google/design.md lint DESIGN.md

# 对比新旧版本，找出 token 级回归
npx @google/design.md diff DESIGN.md DESIGN-v2.md

# 把 DESIGN.md 作为新组件/页面的 prompt 上下文喂给 Agent
```

规范本身是"宽松"的：未知段落会保留而非报错，未知颜色 token 只要值合法就接受，重复段落才报错。这让它能平滑适配各公司已有的设计系统。

### 适用场景

- **AI Coding 工作流**：Cursor / Claude Code / Codex 长期项目，需要让 Agent 持续遵循设计规范
- **设计-开发协作**：把 Figma 里散落的色值/字号沉淀为可版本控制、可 review 的文本文件
- **设计系统迁移**：用 `diff` 命令对比新旧 DESIGN.md，找出 token 级回归
- **WCAG 无障碍合规**：CLI 内置对比度检查，把无障碍验证从人工 review 变成 CI 步骤
- **多品牌/白标产品**：每个品牌一份 DESIGN.md，Agent 按上下文切换

### 安装方式

```bash
# 方式一：CLI 工具（npx 即用即跑）
npx @google/design.md lint ./DESIGN.md

# 方式二：项目级安装
npm install --save-dev "@google/design.md"

# Windows 上注意：npx @google/design.md 在 PowerShell 里可能没输出，
# 因为 .md 后缀跟 Windows Markdown 文件关联冲突。绕过办法：
npx -p @google/design.md designmd lint DESIGN.md
```

### 代码示例

最小可用的 DESIGN.md：

```markdown
---
name: Heritage
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 3rem
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
rounded:
  sm: 4px
  md: 8px
spacing:
  sm: 8px
  md: 16px
---

## Overview
Architectural Minimalism meets Journalistic Gravitas.

## Colors
- **Primary (#1A1CDE)**: Deep ink for headlines.
- **Tertiary (#B8422E)**: "Boston Clay" — the sole driver for interaction.

## Components
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.neutral}"
    rounded: "{rounded.sm}"
    padding: 12px
```

喂给 Agent 后，Claude Code/Cursor 会自动按这套 token 渲染 UI，不会"灵感一来给你换个蓝色"。

### 评价

**优点**：
- 把"设计规范"从 PDF/Style Guide 升维成**机器可消费的结构化契约**，这是 Agent 时代的关键基础设施
- 规范设计非常务实：宽松解析 + 严格段落顺序，适配各种已有体系
- CLI 内置 lint / diff / WCAG，开箱即用
- 跟 `@modelcontextprotocol/specification` 一样，是 Google Labs 在 Agent-Native 工具链上的关键拼图

**缺点**：
- 现阶段组件系统还比较简单，复杂的"响应式/状态/主题切换"规则仍需要靠 Markdown 正文补充
- 生态还在早期，VSCode/Figma 插件、CI 模板需要社区补齐
- 不解决"团队成员不写 DESIGN.md"这个社会工程问题

**推荐指数**：⭐⭐⭐⭐⭐（Agent 时代设计系统的"事实标准候选"，尽早落地不吃亏）

> 本项目昨日（06-25）已入选过 Trending 速递，但今天凭 2,407 stars/day 的成绩依然稳坐第二。再次上榜说明 Google 的"Agent-Native 工具链"叙事正在被开发者广泛接受。

---

## 3. commaai/openpilot — 300+ 车型的开源机器人操作系统

### 核心功能

`openpilot` 不是"另一个 ADAS 库"，而是 comma.ai 把它定位为 **"operating system for robotics"**——一个面向自动驾驶的开源操作系统。它能把 300+ 款车型的辅助驾驶系统升级到接近 L2+ 级别：

- **自适应巡航 + 车道居中**（基础）
- **自动变道**（基于导航/盲区感知）
- **驾驶员监控**（face/eye tracking + 注意力分级）
- **路径预测与可视化**（社区 fork 的"experimental"分支甚至支持视觉化路径规划）
- **数据闭环**：默认上传驾驶数据训练模型（可在设置中关闭）

硬件上需要 **comma four / comma 3X**（自研设备，类似手机+摄像头的形态）+ **车款专属 harness**（OBD-II/CAN 总线接入线束）。软件本身跨平台 C++/Python，**遵循 ISO26262 流程指南**，安全关键代码 (`panda`) 用 C 写并经过严格的 code rigor 规范。

值得一提的是 `nightly-dev` 分支包含了一些实验性功能（比如部分车型的城市 NOA），以及一个比较有趣的运营策略：comma.ai 把仓库放 GitHub + 设备走"用户自己填 URL 安装 release/staging/nightly"——把"灰度发布"做到了硬件级别。

### 适用场景

- **现役车主** 想给爱车升级辅助驾驶能力（前提：车型在支持列表）
- **机器人/自动驾驶研究者** 研究真实大规模行驶数据（comma connect 开放查询）
- **嵌入式/安全关键** 工程师学习"如何把安全代码做到 ISO26262 合规"
- **硬件爱好者** 通过 self-driving-car-for-free 教程让旧设备复活

### 安装方式

```bash
# 一行命令在支持的硬件上安装
bash <(curl -fsSL openpilot.comma.ai)

# 或者选 staging / nightly / nightly-dev
# 在 comma four / comma 3X 的 setup URL 里填：
#   openpilot.comma.ai             → 稳定版
#   openpilot-test.comma.ai        → staging（提前 1-2 周体验新功能）
#   openpilot-nightly.comma.ai     → nightly（最新不稳定代码）
```

支持的车型列表见 [CARS.md](https://github.com/commaai/openpilot/blob/master/docs/CARS.md)，硬件在 [comma.ai/shop](https://comma.ai/shop) 购买。

### 代码示例

openpilot 主要用 Python (C++)，想参与开发可以直接跑测试套件：

```bash
git clone https://github.com/commaai/openpilot
cd openpilot

# 跑 SIL（software-in-the-loop）测试
./tools/scripts/run_tests.sh

# 跑 panda 硬件安全测试（需要硬件）
cd panda && make test_safety

# 看 supported cars 文档并验证你的车型
cat docs/CARS.md | grep -i "your_car"
```

数据查询可以用 comma connect：

```python
# 通过 comma connect API 查询某次行程的数据
import requests

route_id = "..."
url = f"https://api.comma.ai/v1/route/{route_id}/log"
headers = {"Authorization": "JWT <your_token>"}
resp = requests.get(url, headers=headers)
for event in resp.json()["events"]:
    print(event["type"], event["time"])
```

### 评价

**优点**：
- 真正的"用户级"开源 ADAS，覆盖 300+ 车型，远超同类项目
- 严谨的 ISO26262 流程 + SIL/HIL 多层测试 + 硬件 closet 持续回归
- 代码质量高（`panda` 的安全代码风格堪称嵌入式教科书）
- 商业模式清晰（卖硬件 + 收数据），开源协议允许长期独立 fork

**缺点**：
- **硬件锁定**：必须买 comma 的设备才能跑，普通开发者基本只能"读代码"
- "ALPHA QUALITY SOFTWARE FOR RESEARCH PURPOSES ONLY"——法律免责声明明确说了"不是产品"
- 默认上传驾驶数据（虽然可关闭），对隐私敏感用户是减分项
- 中文社区资料较少，新手上手成本高

**推荐指数**：⭐⭐⭐⭐（如果你是车主且车型支持，体验会非常惊艳；如果是开发者，关注 `panda` 的安全代码风格就值回票价）

---

## 4. kunchenguid/no-mistakes — 给 `git push` 加一道 AI 质量门

### 核心功能

`no-mistakes` 是一个**本地 Git 代理 + AI 质量门**：在 `git push origin` 之前，它先把你的代码扔进一个独立的 worktree，依次跑 review → test → docs → lint → push → PR → CI 这一整套流水线，**所有检查都绿了再帮你自动开 PR**。形象地说，它把你的"远程仓库"前面加了一道"门"：

```
你的分支 ── git push no-mistakes ──► 临时 worktree
                                         │
                                  跑 review/test/docs/lint
                                         │
                                  全绿 ─► 自动转发到 origin + 开 PR
                                  有问题 ─► 暂停给你处理
```

最关键的设计选择是**隔离**：你的本地工作目录不会被流水线触碰，所有动作都在一次性 worktree 里完成。

而且它是 **Agent-Native** 的：内置 `/no-mistakes` Skill，可以被 Claude Code / Codex / Cursor / opencode / pi / Copilot 等 50+ runtime 通过 acpx 调用。让 Agent 自动完成"写代码 + 自检 + 提 PR"全链路。

### 适用场景

- **个人项目**：每次 push 前自动跑 review/test，避免把 break 推到 main
- **团队项目**：把"质量门"做成代码而非流程，新人入职第一天就能跑
- **AI Coding Agent 工作流**：让 Agent 写完代码后自动过质量门再 PR，减少人工 review
- **Fork 贡献上游**：用 `--fork-url` 让 no-mistakes 帮你把代码推到自己的 fork 再提 PR
- **CI 不完善的外部项目**：自己先在本地跑一遍高质量检查再 push

### 安装方式

```bash
# macOS / Linux 一键安装
curl -fsSL https://raw.githubusercontent.com/kunchenguid/no-mistakes/main/docs/install.sh | sh

# Windows / Go install / 从源码编译：见官方文档
# https://kunchenguid.github.io/no-mistakes/start-here/installation/

# 在仓库里初始化
cd my-project
no-mistakes init

# 输出示例：
# ✓ Gate initialized
#   repo        /Users/you/src/my-repo
#   gate        no-mistakes → /Users/you/.no-mistakes/repos/abc123.git
#   remote      git@github.com:you/my-repo.git
#   skill       /no-mistakes installed for agents at user level
```

### 代码示例

```bash
# 初始化后，你的工作流变成：
$ git checkout -b feat/new-api
# ... 改代码 ...
$ git push no-mistakes
 * Pipeline started
   Run no-mistakes to review.

$ no-mistakes
# 打开 TUI，你可以：
#   - 让 Agent 自动修复 safe fixes
#   - 手动 approve ask-user 类问题
#   - 选择 skip / fix 任意 finding
# 全绿后自动 push + 开 PR

# 配合 Claude Code / Codex：
# 在对话里说："用 /no-mistakes 跑一下这个改动"
# Agent 会自动跑 pipeline，告诉你哪里需要决策
```

如果你是 Fork 上游贡献者：

```bash
no-mistakes init --fork-url git@github.com:you/awesome-upstream.git
# origin 保持指向 upstream，no-mistakes 会把代码推到你的 fork
# 然后用 no-mistakes 自动开 PR 到 upstream
```

### 评价

**优点**：
- 概念清晰：**让 push 永远是有质量的 push**，把"代码质量"变成代码而非流程
- Agent-Native：`/no-mistakes` Skill 让 Agent 写完代码自动过门
- 隔离设计：流水线跑在临时 worktree，**不打扰你的当前工作**
- Agent-agnostic：claude / codex / copilot / opencode / pi 都能驱动
- Fork 场景友好：--fork-url 设计非常贴心

**缺点**：
- 比较新的项目，CI 模板、规则集生态还在早期
- 默认行为要"过门才能 push"，对追求极致速度的小改动可能繁琐（可关）
- macOS / Linux 友好，Windows 支持文档偏少
- 自托管 `.no-mistakes/repos/` 目录会占额外磁盘空间（每个仓库一份 bare repo）

**推荐指数**：⭐⭐⭐⭐⭐（把"代码质量"做成自动化产物的代表性工具，建议所有认真做项目的开发者试试）

> 本项目昨日（06-25）已入选过 Trending 速递，今天凭 398 stars/day 继续稳坐第 4 位。持续上榜说明开发者社区对"AI-Native 质量门"的需求是真实的。

---

## 5. grafana/grafana — 观测性领域的事实标准

### 核心功能

`Grafana` 是开源、可组合的**观测性与数据可视化平台**。它的核心定位是"指标/日志/追踪的统一可视化层"——无论数据来自 Prometheus、Loki、Elasticsearch、InfluxDB、Postgres 还是 ClickHouse，Grafana 都能用同一套查询语言 + 同一套面板把它们画出来。

10.x 版本的核心能力包括：

- **数据源无关的可视化**：内置 30+ 数据源（Prometheus / Loki / Tempo / Elasticsearch / InfluxDB / Postgres / MySQL / BigQuery / Snowflake / Datadog / CloudWatch / ...）
- **仪表盘即代码**：JSON 模型 + provisioning 文件 + Terraform provider
- **告警与路由**：Unified Alerting 引擎，支持 webhook / Slack / PagerDuty / OpsGenie / 邮件
- **Explore 模式**：临时查询 + 日志关联 + trace 跳转
- **Grafana Loki / Tempo / Mimir / Pyroscope**：从可视化延展到日志/追踪/持续剖析/指标的完整存储栈
- **插件生态**：1300+ 官方/社区面板、App、数据源插件
- **Grafana 11/12 新方向**：Scenes 框架、Drilldown 体验、LLM 驱动的自然语言查询（实验性）

对于企业用户还有 **Grafana Enterprise** 特性（LDAP、Reporting、Adaptive Logging 等）和 **Grafana Cloud**（托管服务）。

### 适用场景

- **SRE / DevOps**：搭建 Prometheus + Grafana 的事实标准监控栈
- **后端开发**：可视化业务指标（QPS、错误率、P99 延迟）
- **业务分析**：把 SQL 数据源连进来，替代轻量 BI 工具
- **物联网 / 嵌入式**：连 InfluxDB / TimescaleDB 画时序面板
- **企业统一可观测性**：整合多云、多数据源到一个统一界面

### 安装方式

```bash
# macOS
brew install grafana
brew services start grafana

# Docker（最常用）
docker run -d --name=grafana \
  -p 3000:3000 \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana-oss:latest

# Ubuntu / Debian
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_11.2.0_amd64.deb
sudo dpkg -i grafana_11.2.0_amd64.deb
sudo systemctl enable --now grafana-server

# 默认账号：admin / admin（首次登录强制改密码）
# 访问 http://localhost:3000
```

### 代码示例

Grafana 最强的是"用代码描述仪表盘"，下面是 provisioning 一个完整 dashboard 的最小例子：

```yaml
# /etc/grafana/provisioning/dashboards/main.yaml
apiVersion: 1
providers:
  - name: 'default'
    folder: 'MyApp'
    type: file
    options:
      path: /var/lib/grafana/dashboards
```

```yaml
# /var/lib/grafana/dashboards/orders.json（片段）
{
  "title": "Orders QPS",
  "panels": [
    {
      "type": "timeseries",
      "title": "QPS by region",
      "targets": [
        {
          "expr": "sum by (region) (rate(orders_total[1m]))",
          "datasource": "Prometheus"
        }
      ]
    }
  ]
}
```

自然语言查询（实验性）：

```
# 在 Explore 面板输入：
"过去 1 小时错误率最高的服务是哪个？"
# Grafana 11+ 会自动转成 PromQL：
topk(3, sum by (service) (rate(http_requests_total{status=~"5.."}[5m])))
```

### 评价

**优点**：
- **生态王者**：30+ 官方数据源 + 1300+ 插件，几乎能想到的 backend 都能接
- 仪表盘即代码友好，Terraform / Ansible / GitOps 全流程支持
- 性能优秀：10.x 在大数据量下查询速度比 8.x 提升明显
- 社区活跃，企业采用率高（NASDAQ、eBay、PayPal、Stack Overflow 等都用）
- Grafana 11+ 的 Scenes 框架让"动态可交互"仪表盘开发成本大幅下降

**缺点**：
- **配置复杂度高**：第一次接入多数据源 + 告警路由的学习曲线陡峭
- 仪表盘 JSON 体积大、难以 review，团队协作时建议用 Grafonnet / Jsonnet
- 部分企业版特性（Reporting、LDAP 完整集成）需要付费
- 中文文档相对英文滞后，新功能往往要等几个月才有翻译

**推荐指数**：⭐⭐⭐⭐⭐（观测性领域的"基础水电煤"，从个人项目到企业级都绕不开）

---

## 6. ripienaar/free-for-dev — DevOps 人的"免费羊毛目录"

### 核心功能

`free-for-dev` 是一个**面向 devops / infradev 的 SaaS/PaaS/IaaS 免费层清单**。它不像 awesome-* 那样宽泛，而是专注回答一个问题：**"我作为开发者/团队，哪些云服务是免费可以白嫖的，免费层具体限制是什么？"**

按类别组织：

- **PaaS**：Heroku / Render / Railway / Fly.io / Vercel / Netlify / Cloudflare Pages
- **数据库**：Supabase / Neon / PlanetScale / Turso / Upstash / MongoDB Atlas
- **CI/CD**：GitHub Actions / CircleCI / Drone / Travis（仍部分免费）
- **监控 / 日志**：Sentry / Logtail / Better Stack / Highlight
- **邮件 / 通讯**：Mailgun / SendGrid / Resend / Postmark 免费层
- **存储**：Cloudflare R2 / Backblaze B2 / Wasabi
- **身份认证**：Auth0 / Clerk / Supabase Auth / Firebase Auth
- **AI / LLM**：OpenAI / Anthropic / Google AI Studio / Groq / Cohere 免费额度
- **DNS / 域名**：Freenom / Porkbun / Cloudflare Registrar
- **CDN / 安全**：Cloudflare / Fastly 免费层

每条目都标注 **免费层限制**（如 "Supabase 免费层：500MB 数据库 + 1GB 存储 + 50k MAU"），是 devops 选型时的"白嫖地图"。

### 适用场景

- **个人开发者 / 独立开发者**：把所有 SaaS 都换成免费层，每月节省 $50-200
- **初创团队 MVP 阶段**：零成本搭建完整技术栈
- **学习 / 实验项目**：用免费层搭 demo / 教学示例 / PoC
- **架构师选型**：快速对比同类服务的免费层限制
- **预算评审**：团队扩规模前先看哪些免费额度会被"撞穿"

### 安装方式

这不是软件，是一个 Markdown 列表——直接读就行：

```bash
# 在线读
open https://github.com/ripienaar/free-for-dev

# 克隆本地离线查阅
git clone https://github.com/ripienaar/free-for-dev.git
cd free-for-dev
grep -i "ai" README.md   # 找 AI 相关的免费服务
grep -B 1 "1000.*month" README.md  # 找月免费量 1000 的服务
```

### 代码示例

利用这个仓库的常见姿势：

```bash
# 1. 选 AI Coding 平台时
grep -A 3 -i "cursor\|copilot\|codeium" README.md

# 2. 给 side project 选数据库时
grep -B 1 -A 5 -i "postgres\|mysql\|sqlite" README.md | head -50

# 3. 给 newsletter 选邮件服务商时
grep -B 1 -A 5 -i "mailgun\|sendgrid\|resend" README.md

# 4. 用 ripienaar 的"精选"标记
# README 中带 ⭐ 的条目是 ripienaar 个人推荐
grep -B 1 "⭐" README.md
```

进阶用法：把这份清单导入团队 wiki，每季度 review 一次"我们还在用哪些免费额度 + 下季度预算"。

### 评价

**优点**：
- **领域第一**：devops 免费层清单的事实标准，被无数公司 reference
- 持续维护，PR 活跃（每月都有几十个新条目/更新）
- 每条都标注限制 + 链接，不用你自己去每个官网挖
- 按类别组织清晰，搜索友好
- 配套的 `free-for.dev` 站点体验更佳

**缺点**：
- **英文为主**：对中文开发者筛选"国内服务"有限（需配合 `free-for-dev-cn` 之类 fork）
- 免费层变动频繁：有些条目半年后悄悄改限制，要记得定期 review
- 不收录太新的服务（审核较严），冷门但好用的可能错过
- 没有"按使用场景推荐"的智能——你得自己挑

**推荐指数**：⭐⭐⭐⭐⭐（每个 devops 工程师都该 star 一次的项目；月度 review 一次能省下不少订阅费）

---

## 7. opendatalab/MinerU — 把 PDF/Office 文档变成 Agent 可消费的 LLM-ready 文本

### 核心功能

`MinerU` 是 OpenDataLab 出品的**高精度文档解析引擎**，专为 LLM/RAG/Agent 工作流设计。它能把复杂文档（PDF、DOCX、PPTX、XLSX、图片、网页）转成**结构化的 Markdown / JSON**，输出**遵循人类阅读顺序**，自动去除页眉页脚，跨页表格合并。

最新 **3.4 版本（2026-06-18）** 的关键升级：

- **OCR 模型升级到 PP-OCRv6**：OmniDocBench v1.6 上准确率提升约 11%
- **OCR 推理管线优化**：处理速度提升约 100%（批量文档/扫描件场景尤其明显）
- **模型下载逻辑优化**：自动选择最优模型源、本地缓存复用、自动回写配置
- 移除日文/繁中/英文/拉丁的 OCR 选项（统一路由到 ch 模型），简化配置

核心能力矩阵：

- **多格式支持**：PDF / DOCX / PPTX / XLSX / 图片 / 网页，原生解析（非 OCR）
- **公式 → LaTeX · 表格 → HTML**：数学公式、复杂表格完美还原
- **版面还原**：扫描件、手写体、多栏排版、跨页表格合并、阅读顺序识别
- **VLM + OCR 双引擎**：109 种语言 OCR
- **三种推理后端**：pipeline（CPU/GPU 都能跑、稳定）/ vlm-engine（高准确率，支持 vLLM/LMDeploy/mlx）/ hybrid-engine（高准确率 + 原生文本提取 + 低幻觉）
- **国产 AI 芯片适配**：昇腾 / 寒武纪 / 燧原 / 沐曦 / 摩尔线程 / 昆仑芯 / 壁仞 / 海光 / 平头哥
- **集成方式**：MCP Server（Cursor/Claude Desktop/Windsurf）、LangChain / LlamaIndex / RAGFlow / RAG-Anything / Flowise / Dify / FastGPT、Python/Go/TypeScript SDK、CLI、REST API、Docker

### 适用场景

- **RAG 系统**：高质量文档解析是 RAG 效果的天花板，MinerU 帮你把"脏 PDF"变干净
- **企业知识库**：合同、财报、手册、扫描件统一入库
- **Agent 工作流**：把 PDF 报告喂给 Claude / GPT 前先过 MinerU
- **学术研究**：论文 PDF → Markdown，公式保留 LaTeX，表格保留结构
- **多语言场景**：109 种 OCR 满足跨境业务
- **国产化部署**：10+ 国产 AI 芯片支持，政企合规友好

### 安装方式

```bash
# Python（推荐用 uv）
uv pip install -U "mineru[core]"

# 或 pip
pip install -U "mineru[all]"

# 第一次跑会自动下载模型（首次可指定 model_source）
mineru --source "https://..." -o ./output

# Docker
docker pull opendatalab/mineru:latest
docker run -v $(pwd)/output:/app/output opendatalab/mineru:latest \
  mineru -p /app/input/test.pdf -o /app/output
```

MCP Server 模式：

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "mineru": {
      "command": "uvx",
      "args": ["mineru-mcp"],
      "env": {
        "MINERU_MODEL_SOURCE": "modelscope"
      }
    }
  }
}
```

### 代码示例

```python
from mineru import MinerUPipeline

# Pipeline 后端（CPU/GPU 都行，稳定）
pipeline = MinerUPipeline(backend="pipeline")
result = pipeline.parse("report.pdf")
print(result.markdown)  # 结构化 Markdown
print(result.tables)    # 表格列表（HTML 格式）
print(result.formulas)  # 公式列表（LaTeX 格式）

# VLM 后端（高准确率，需要 GPU）
from mineru import VLMEngine
engine = VLMEngine(model="MinerU2.5-Pro-2605-1.2B", backend="vllm")
result = engine.parse("complex_paper.pdf", effort="high")
```

CLI 用法：

```bash
# 解析 PDF，输出 Markdown + JSON
mineru parse -p ./docs/annual_report.pdf -o ./output

# 批量处理整个目录
mineru batch -i ./docs/ -o ./output --workers 4

# 查看版本 + 模型信息
mineru --version
mineru model list
```

### 评价

**优点**：
- 文档解析准确率在 OmniDocBench v1.6 等公开基准上**领先同类**
- 3.4 版本 OCR 速度翻倍 + 模型缓存优化，工程化做得很好
- **支持国产 AI 芯片**：政企/国产化部署场景的重要选择
- MCP / LangChain / Dify / FastGPT 集成开箱即用
- 109 种语言 OCR，跨境业务友好

**缺点**：
- VLM 模式需要较强的 GPU（默认推荐 24GB+ 显存）
- 模型首次下载体积较大（多个 GB），需要稳定的 model source
- 商用许可证需要注意：从 AGPLv3 切换到 MinerU Open Source License 后，商业使用前最好 review 条款
- 对超复杂版面（艺术字、海报、杂志跨页）的还原仍有边界情况

**推荐指数**：⭐⭐⭐⭐⭐（RAG/Agent 工程师的"必装工具"，3.4 升级后 OCR 速度翻倍，强烈推荐升级）

---

## 8. alchaincyf/zhangxuefeng-skill — 把"张雪峰思维"做成 Agent Skill

### 核心功能

`zhangxuefeng-skill` 是一个**结构化的"张雪峰认知操作系统"**——不是简单的 ChatGPT 套个名人面具，而是基于：

- **5 本著作**（张雪峰已出版的书）
- **15+ 篇权威媒体深度采访**
- **30+ 条一手语录**
- **11 个关键决策记录**
- **完整人生时间线**

提炼出 **5 个核心心智模型 + 8 条决策启发式 + 完整表达 DNA**，再把这些方法论封装成符合 [Agent Skills](https://agentskills.io) 协议的 SKILL.md，让任何支持 Agent Skills 的 runtime（Claude Code / Codex / Cursor / OpenClaw / Hermes Agent / CodeBuddy / Workbuddy / Gemini CLI / OpenCode 等 50+）都能加载。

核心心智模型：

- **社会筛子论**：社会就是大筛子，学历筛孩子、房子筛父母、工作筛家庭
- **选择 > 努力**：方向错误的努力是浪费
- **就业倒推法**：从毕业后的就业数据倒推今天的专业选择
- **阶层现实主义**：家里没矿别谈理想，先谋生再谋爱
- **争议即传播**：把观点推到极端才有传播力

它专门解决"高考志愿 / 考研 / 职业规划"这类**中国家庭最焦虑的决策场景**，并明确演示了"用张雪峰的框架分析"和"普通 ChatGPT 回答"的差异：

> 普通 AI："拼多多有增长潜力但也面临竞争压力，投资者需要权衡..."
> 张雪峰 Skill：激进型 $95-105 / 稳健型 $85-95 / 保守型 观望

按 [agentskills.io](https://agentskills.io) 协议设计，意味着即使 runtime 不支持 Skill 自动加载，你也可以把 SKILL.md 内容粘贴进对话——它本质就是 markdown + YAML frontmatter。

### 适用场景

- **高考志愿填报**：分数 × 省份 × 家庭背景 → 推荐专业 + 院校
- **考研决策**：是否考研、考哪里、考什么专业、考几次
- **职业规划**：转行、考公、留学、直接工作的对比
- **AI 教育应用**：把"经验型建议"做成可复用的 Agent 知识
- **Skill 协议学习**：作为开源 Skill 范本，学习如何把"人类专家的方法论"蒸馏成 Agent 资产

### 安装方式

```bash
# 通用 CLI 安装器（自动识别 runtime，55+ runtime 支持）
npx skills add alchaincyf/zhangxuefeng-skill

# 或指定 runtime
npx skills add alchaincyf/zhangxuefeng-skill -a claude-code
npx skills add alchaincyf/zhangxuefeng-skill -a codex
npx skills add alchaincyf/zhangxuefeng-skill -a cursor
npx skills add alchaincyf/zhangxuefeng-skill -a openclaw

# 安装位置（部分）：
#   ~/.claude/skills/zhangxuefeng-skill/    Claude Code
#   ~/.codex/skills/zhangxuefeng-skill/     Codex CLI
#   ~/.cursor/skills/zhangxuefeng-skill/    Cursor
#   ~/.openclaw/workspace/skills/zhangxuefeng-skill/  OpenClaw

# 手动 git clone
git clone https://github.com/alchaincyf/zhangxuefeng-skill <对应 runtime 的 skills 目录>
```

### 代码示例

装好后跟 Agent 对话：

```
我： 我孩子 560 分，河南的，家里没金融背景，该报什么专业？

Claude Code (加载 Skill 后)：
> 张雪峰 ❯ 停停停，你先别急着说金融。我问你几个问题。
>   家里是做金融的吗？爸妈在银行、证券公司、基金公司？
>   没有？那我跟你说，金融这个行业，千万别碰。
>   560 分河南考生，大概率去个双非一本学金融，
>   毕业出来跟 985 的抢岗位，你拿什么抢？脸吗？
> 
> 我的建议：
>   560 分河南，我给你一个方向：计算机或者电气工程。
>   理工科选专业，文科选学校，记住这句话。
>   你孩子这个分数，去个好点的一本学计算机，
>   毕业出来至少有一门手艺，不用求人。
```

完整对话记录见 [examples/](https://github.com/alchaincyf/zhangxuefeng-skill/tree/main/examples)。

### 评价

**优点**：
- **不是"套壳"，是"蒸馏"**：5 本著作 + 30+ 语录 + 11 个决策 → 真实方法论
- **agent skills 协议标准**：跨 55+ runtime 可用，未来 Agent 生态的"事实标准候选"
- 解决中国家庭的真实痛点（高考/考研/职业），需求极其刚性
- 自带完整 examples，新人可直接理解"为什么这是 Skill 不是 prompt"
- 项目作者 alchaincyf 还有 `nuwa-skill` / `huashu-design` 等多个 Skill 系列，生态连贯

**缺点**：
- "张雪峰 IP"本身有争议，部分用户对方法论的态度两极化
- Skill 内容基于公开资料整理，**不等于张雪峰本人背书**
- 对国外用户/不熟悉中国教育体系的人，价值有限
- 启发式偏"实用主义/阶层现实主义"，缺少对"理想主义路径"的平衡视角

**推荐指数**：⭐⭐⭐⭐（对中国家庭刚需场景非常有用；作为"Agent Skills 协议"的代码范本也值得学习）

---

## 9. mauriceboe/TREK — 自托管的"全能旅行规划 + 协作平台"

### 核心功能

`TREK` 是一个**自托管的全能旅行/行程规划平台**，把传统旅游 App（TripAdvisor、Google Trips、Polarsteps）和协作工具（Notion、Figma、Maps）的能力整合到一个开源项目里。功能矩阵极其庞大：

**规划能力**：

- 拖拽式 day planner：把地点组织成多日行程，跨日移动
- 交互式地图：Leaflet 或 Mapbox GL，支持 3D 建筑、地形、照片标记、聚类、路径可视化
- 地点搜索：Google Places（照片/评分/营业时间）或 OpenStreetMap（免费、无 API key）
- 地点导入：Google Maps / Naver Maps 分享列表、GPX / KML / KMZ / GeoJSON
- 时间戳笔记：拖拽重排、图标标签
- 路线优化：自动排序地点并导出到 Google Maps
- 天气预报：Open-Meteo 16 天预报 + 历史气候 fallback
- 分类筛选：地图上只看匹配类别的 pin

**资源管理**：

- 预订：航班、酒店、餐厅，状态、确认号、附件管理；从预订邮件/PDF 自动导入（用 KDE Itinerary）
- 费用：Splitwise 风格多人分账、按人/按日分解、settle-up、多币种
- 行李清单：分类、模板、成员分配、进度跟踪
- 行李追踪：iOS 风格重量分布
- 文档管理：行程/地点/预订附件（PDF/票/扫描件），单文件 ≤ 50MB
- PDF 导出：完整行程带封面、图片、笔记

**协作能力**：

- WebSocket 实时同步：所有连接用户即时看到变化
- 多用户行程：邀请成员，角色权限
- 邀请链接：一次性 / 复用 / 过期
- SSO（OIDC）：Google / Apple / Authentik / Keycloak 或任何 OIDC provider
- 2FA：TOTP + 备份码
- Passkeys：WebAuthn 免密登录（指纹 / 人脸 / PIN / 安全密钥），管理员可关

**PWA & 离线**：

- iOS / Android 浏览器直接安装，无需 App Store
- Service Worker 缓存瓦片、API、上传（Workbox）
- 完整全屏 / 主题化状态栏 / 启动画面
- 移动端触控优化 + 安全区域处理

**其他模块**：

- Atlas：访问过国家地图 + bucket list + 旅行统计 + streak + liquid-glass UI
- Journey：杂志风格旅行日志 + 照片（可接 Immich / Synology）+ 地图 + 心情
- Vacay：个人假期规划 + 日历 + 100+ 国家假日 + carry-over
- AirTrail：接自托管 AirTrail 实例同步航班到预订

**AI 集成（MCP Server）**：

- 内置 MCP Server（OAuth 2.1 认证）
- 150+ 工具 / 30 资源
- 27 个 OAuth scopes × 13 个权限组
- AI 可全自动：创建行程、规划 days、打包清单、预算、标记访问国家
- 内置 prompt：trip-summary / packing-list / budget-overview
- addon-aware：暴露 Atlas / Collab / Vacay 等模块给 AI

**国际化 & 管理**：

- 20 种语言：EN/DE/ES/FR/IT/NL/HU/RU/ZH/ZH-TW/PL/CS/AR(RTL)/BR/ID/TR/JA/KO/UK/GR
- 管理面板：用户、邀请、打包模板、分类、addon、API key、备份、GitHub 历史
- 通知：邮件 (SMTP) / webhook / ntfy / 应用内
- 自动备份：定期 + 手动触发

### 适用场景

- **家庭/朋友团体旅行**：多人协作 + 共享费用 + 共享文档
- **数字游民 / 慢旅行**：长期行程、跨多国、跨币种管理
- **隐私敏感用户**：自托管，数据不出机器
- **旅行社/差旅团队**：企业级 SSO + 角色权限
- **AI 实验**：MCP Server 让 AI 帮你规划行程
- **不喜欢 Google/Apple 生态的人**：可彻底替换 Google Trips / Apple Wallet 行程功能

### 安装方式

TREK 是 Docker 友好的自托管应用：

```bash
# 方式一：Docker Compose（推荐）
git clone https://github.com/mauriceboe/TREK.git
cd TREK
docker compose up -d

# 默认监听 http://localhost:8080
# 首次访问会引导创建管理员账号

# 方式二：手动配置
# 1. 克隆仓库
# 2. 配置 .env（数据库、SMTP、SSO provider 等）
# 3. 运行迁移 + 启动 backend
npm install
npm run migrate
npm run start

# 详细安装文档见 /docs/install.md
```

启用 MCP Server 后，可被 Claude Desktop / Cursor / Windsurf 等客户端发现：

```bash
# 在 .env 中配置 MCP
ENABLE_MCP=true
OAUTH_CLIENTS=claude-desktop,...

# Claude Desktop config
{
  "mcpServers": {
    "trek": {
      "url": "http://your-trek-host/mcp",
      "auth": "oauth-2.1"
    }
  }
}
```

### 代码示例

TREK 主要面向终端用户而非开发者，下面是几个"AI + TREK"协作的示例：

```
# 你：我要规划 7 月冰岛环岛 10 日游

Claude（已连 TREK MCP）：
> 我已经创建了 trip "Iceland Ring Road 2026"，并自动：
> - 添加了 10 天日程模板
> - 调用 Open-Meteo 拉了雷克雅未克/维克/阿克雷里未来 16 天天气预报
> - 从 OpenStreetMap 找到了 25 个 POI（瀑布、黑沙滩、冰川、温泉）
> - 按地理位置自动排序了每日路线
> - 生成了 4 人打包清单（极光摄影、温泉、徒步）
> - 创建了多币种费用预算（ISK/EUR/USD）
>
> 你想先看地图，还是先调每日行程？
```

费用结算：

```
你：这次旅行最终花了多少钱？

Claude：调取了你的费用记录：
  - 住宿（7 晚）：185,400 ISK / 4 人分 = 46,350 ISK/人
  - 餐饮：58,200 ISK / 4 人分 = 14,550 ISK/人
  - 租车 + 油费：92,000 ISK / 4 人分 = 23,000 ISK/人
  Settle-up：Alice 还欠 Bob 8,200 ISK
```

### 评价

**优点**：
- **功能极其完整**：从规划到协作到结算到 AI，一条龙
- 自托管 + SSO + Passkey + 2FA，企业级安全
- MCP Server + 150+ 工具，把 AI 当成"超级旅行助理"用
- 20 种语言 + 多币种，国际化做得非常好
- PWA + 离线缓存，移动端体验出色
- 活跃开发中（看截图迭代速度），社区在 Discord

**缺点**：
- **学习曲线陡**：功能多 = 配置项多，第一次部署需要花时间
- 自托管对个人用户门槛较高（要懂 Docker / 反代 / 备份）
- 移动 App 仍是 PWA，对纯 native 体验要求高的用户可能觉得"差一口气"
- 中文 UI 完整但社区案例较少，踩坑靠自己

**推荐指数**：⭐⭐⭐⭐⭐（如果你认真组织多人/多国旅行，TREK 是目前自托管领域功能最完整的方案；尤其推荐给重视隐私 + 想要 AI 协作的旅行者）

---

## 10. xbtlin/ai-berkshire — 用 Claude Code 做"价值投资研究"的 4 大师对抗框架

### 核心功能

`AI Berkshire` 是一个**基于 Claude Code 的价值投资研究 Skill 合集**，把**巴菲特、芒格、段永平、李录**四位价值投资大师的方法论系统化、结构化，再通过 **AI Agent** 实现专业级投研。它解决的不是"AI 能不能分析"，而是"AI 分析能不能拿来做决策"。

**核心定位差异**：

> 你直接问 Claude："帮我分析拼多多值不值得买"，你得到的是"一方面...另一方面..."的平衡分析，最后以"投资有风险，请自行判断"收尾。
> 这种分析看起来对，但没法拿来做决策。

AI Berkshire 强制输出"通过 / 不通过 / 灰色地带"，带具体**价格区间 + 分层建议**（激进型 / 稳健型 / 保守型）。

**真实战绩**：

- 2024 全年：**+69.29%**（标普 500 +23.31%，跑赢 46 个百分点）
- 2025 至今：**+66.38%**（标普 500 +16.39%，跑赢 50 个百分点）
- 恒生指数 / 沪深 300 / 纳斯达克同期全部跑赢
- 两年累计实盘收益超 **146 万元**

**4 大师视角对抗**（以拼多多为例）：

- **段永平（商业模式）**：好生意，C2M 模式难以复制 → 3.7/5
- **巴菲特（财务估值）**：扣现金 PE 仅 6.3x，印钞机 → 4.4/5
- **芒格（逆向思考）**：护城河比想象中浅，抖音 3 年做到 4 万亿 GMV → 3.5/5
- **李录（长期确定性）**：管理层文化有隐患，10 年后不确定 → 2.0/5

> 巴菲特说"真便宜"，李录说"不确定就不买"——这种冲突才是投资决策的真实状态。单一 prompt 无法制造这种多视角对抗，而这恰恰是避免盲点的关键。

**多层"防骗"机制**：

- **信息丰富度评级**（A/B/C）：防止"资料多 = 确定性高"的幻觉
- **芒格式逆向检验**：强制思考失败场景，列 5 大情景 + 概率
- **快速否决清单**：8 条红线一票否决（管理层诚信污点 → 直接否决）
- **反共识检查**：避免和市场想法一样
- **留白原则**：数据不足时标注"灰色地带"

**金融工程严谨性**：

- 所有计算用 Python `decimal.Decimal`（精确十进制，不用 float）
- 关键数据至少 2 个独立来源交叉验证
- 市值手算校验：股价 × 总股本 vs 报告数据，偏差必须 < 0.5%

**架构**：

- **Skill 层**：16 个明确入口（深度研究、财报分析、行业筛选、持仓管理、思维工具）
- **Agent 层**：每个 skill 内部 4 个 Agent 并行（独立搜索、独立判断、互相挑战）
- **工具层**：精确计算、实时检索、报告抽检

**核心 Skill**：

- `/investment-research`：4 大师综合深度分析
- `/investment-team`：4 Agent 并行投研团队
- `/management-deep-dive`：管理层纵深
- `/private-company-research`：未上市公司（蚂蚁、SpaceX 等）
- `/deep-company-series`：8 篇长文系列拆一家公司
- `/earnings-review`：财报精读（一手资料）
- `/earnings-team`：财报精读多 Agent 版

### 适用场景

- **个人投资者**：把"问问 AI"升级成"系统化研究"
- **专业投资人 / 私募**：作为分析师助理的"预审 + 校验"工具
- **商学院 / 金融教育**：把大师方法论蒸馏成可复用的 Agent 资产
- **AI 工程师**：学习"如何把领域专家的方法论做成 Agent Skill"
- **多 Agent 系统研究者**：参考其 Agent 分工 / 综合架构

### 安装方式

```bash
# 假设你已经装好 Claude Code
# 1. 克隆到 Claude Code skills 目录
git clone https://github.com/xbtlin/ai-berkshire \
  ~/.claude/skills/ai-berkshire

# 2. 安装金融计算依赖
pip install -r ~/.claude/skills/ai-berkshire/requirements.txt

# 3. 在 Claude Code 中调用
> /investment-research 拼多多
> /investment-team 英伟达
> /management-deep-dive 腾讯管理层
> /earnings-review 阿里最新季报
```

### 代码示例

```bash
# 启动 4 Agent 并行投研团队
> /investment-team 拼多多

Claude Code（加载 Skill 后）：
> 已启动 /investment-team，启动 4 个 Agent 并行研究：
>   Agent 1 (段永平视角 - 商业模式)：...
>   Agent 2 (巴菲特视角 - 财务估值)：...
>   Agent 3 (芒格视角 - 行业竞争)：...
>   Agent 4 (李录视角 - 风险管理)：...
> 
> [3 分钟后]
> 综合报告：
>   能力圈    ★★★★☆
>   好生意    ★★★★☆
>   护城河    ★★★★☆
>   管理层    ★★★★☆
>   安全边际  ★★★★★
>   综合      4.4 / 5
> 
> 决策：
>   激进型   当前价位可建仓 20%   $95-105
>   稳健型   等回购政策明确后建仓  $85-95
>   保守型   不符合 10 年确定性标准，观望
```

财务验证示例：

```bash
# 校验市值手算
> 帮我验证腾讯市值：股价 510 HKD，总股本 91.1 亿，报告 4.65 万亿 HKD

Claude（自动调用工具）：
$ python3 tools/financial_rigor.py verify-market-cap \
    --price 510 --shares 9.11e9 --reported 4.65e12 --currency HKD
✅ 验证通过, 偏差仅 0.08%
```

### 评价

**优点**：
- **罕见的"实盘验证"项目**：2024/2025 两年实盘 +69%/+66%，跑赢所有主流指数
- **4 大师对抗而非单一视角**：真实模拟"投资委员会"决策过程
- **防骗机制扎实**：信息评级、逆向检验、否决清单、反共识、留白——5 道护栏
- **金融工程严谨**：Decimal 精确计算 + 双源交叉验证
- **架构清晰**：Skill / Agent / 工具三层，**是学习 Agent 工程的优秀范例**
- 中文 README + 真实战绩图，可信度高

**缺点**：
- **依赖 Claude Code**：不直接支持其他 Agent runtime（截至当前版本）
- **业绩归因难独立验证**：截图来自富途账户，但无法确认是否"全部来自该框架"还是叠加主观操作
- **历史收益不代表未来**：2024/2025 价值投资环境相对有利，AI Agent 时代趋势变化后是否依然有效待观察
- **适用标的有限**：深度价值投资框架，对科技股泡沫 / 题材股 / 期权策略覆盖较少
- **法律合规**：投资建议的合规边界需要用户自行判断

**推荐指数**：⭐⭐⭐⭐⭐（个人投资者：可作为分析辅助；Agent 工程师：作为"领域专家方法论如何做成 Skill"的顶级范例；金融从业者：可参考其多 Agent 对抗 + 防骗机制）

---

## 今日速递小结

今天榜单的几个信号值得注意：

1. **隐私 / 基建回归**：`simplex-chat` 和 `commaai/openpilot` 都在做"看起来不性感但极其重要"的基础设施——隐私通讯和开源机器人 OS
2. **AI Agent 工作流成熟化**：`no-mistakes`（AI 质量门）、`design.md`（Agent 设计契约）、`MinerU`（Agent 文档解析）、`ai-berkshire`（Agent 投研框架）—— 一整条"Agent-Native 工具链"已经成形
3. **中文项目集体爆发**：`alchaincyf/zhangxuefeng-skill`、`xbtlin/ai-berkshire` 都属于"中文场景 + AI Skill 协议"的代表作，反映中国开发者在 Agent 时代的"本土化优势"
4. **Self-hosted 复兴**：`TREK`（自托管旅行规划）、`simplex-chat`（自托管 SMP）—— 大厂云服务的对冲方案正在被更多开发者选择

下期见！👋

---

> **数据来源**：[github.com/trending](https://github.com/trending)  
> **抓取时间**：2026-06-27 14:14 (Asia/Shanghai) / 2026-06-27 06:14 (UTC)  
> **覆盖项目数**：10  
> **说明**：每个项目的"评价"部分为基于公开资料的客观分析，仅供参考，投资/技术决策请结合自身情况独立判断。