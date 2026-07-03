---
title: '每日开源速递 - 2026-07-03'
date: 2026-07-03T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

> **抓取时间**：2026-07-03 13:08 (Asia/Shanghai)  
> **数据来源**：<https://github.com/trending>（GitHub 官方"今日热门"页面，按当日获得 ⭐ 数排序）  
> **覆盖范围**：今日 trending 前 17 名项目，逐条带"核心功能 / 适用场景 / 安装方式 / 代码示例 / 评价"五段式解读。

今天 trending 的**最大变量**是一个叫 `JuliusBrussee/caveman` 的 Claude Code 技能——空降 #2,标语是"why use many token when few do trick",官方 benchmark 说能砍 65%~75% 输出 token,脑洞开得挺野;另外 `ChromeDevTools/chrome-devtools-mcp` 第一次以"MCP 形态"登上 trending,意味着 Google 正式把浏览器调试能力"送给"所有 AI Agent。下面 17 个项目挨个拆开。

---

## 1. usestrix/strix — 把"AI 黑客"团队塞进你的 CI(连续多日榜首,+约 1,000 ⭐)

**核心功能**  
"Strix"是一群**自主协作的 AI 渗透测试 agent**——不是单一工具,而是侦察 → 漏洞发现 → 漏洞利用 → 报告生成 的完整闭环,内置 HTTP 拦截代理、浏览器漏洞利用、shell 终端、Python 沙箱运行 PoC、SAST+DAST 静态动态分析,以及 CVSS + OWASP 分类的知识库。多 agent 协同(recon / exploit / post-exploit),共享发现、串联漏洞。

**适用场景**  
- 中小团队的"周度自动安全巡检":给内部 staging 跑一遍,生成可贴进 Confluence 的报告
- 红队演练的第一阶段加速器(信息收集 + 漏洞扫描)
- Bug Bounty 自动化:为发现的漏洞生成可直接提交的 PoC
- CI/CD 集成:每个 PR 自动跑,阻断 unsafe 代码合并

**安装方式**  
```bash
curl -sSL https://strix.ai/install | bash
export STRIX_LLM="openai/gpt-5.4"
export LLM_API_KEY="sk-..."
strix --target ./your-app-directory
```
(首次运行会自动拉沙箱 Docker 镜像)

**代码示例(CLI 一次"灰盒 + 业务逻辑"测试)**  
```bash
strix --target https://api.your-app.com \
  --instruction "Perform authenticated testing using credentials: alice:hunter2. Focus on IDOR and business logic flaws in /api/orders/* endpoints."
```
高级:多目标 + PR diff 范围  
```bash
strix -t https://github.com/org/app -t https://staging.your-app.com
strix --target ./app --instruction-file ./engagement.md --scan-mode standard
```

**评价**  
Strix 的卖点是"agent 自主决策下一步扫什么",而不是传统 `nmap + sqlmap + nuclei` 的死板流水线;**所有发现都配可复现的 PoC**,这比商业 SAST 厂商的"95% 误报率"强很多。但必须**只在明确授权的目标**上使用,且报告**人工复核**——AI 渗透工具不是合规挡箭牌。**推荐度:⭐⭐⭐⭐⭐(值得立刻试,但必须配审计流程)**。

---

## 2. JuliusBrussee/caveman — Claude Code 的"省 token 模式",砍掉 65% 输出

**核心功能**  
一个 Claude Code 技能/插件(同时兼容 Codex / Gemini / Cursor / Windsurf / Cline / Copilot / 30+ agent),通过 prompt 改造让 agent 说话像"穴居人"——保留所有技术正确性,只压缩"废话"。自带的 `caveman-compress` 还能压缩你的 `CLAUDE.md` / 项目笔记,**每次 session 启动就省 ~46% 输入 token**。

**适用场景**  
- 长期跑 Claude Code 做开发的个人/小团队(每月账单肉眼可见地降)
- 想"减少 Agent 啰嗦,但不损失正确性"的强迫症用户
- 把 Agent 输出当成"Markdown 文档"长期归档的人(短 = 易检索)
- 给 MCP server 做一层压缩中间件(`caveman-shrink`)

**安装方式**  
```bash
# 一行装到所有 agent(自动检测已安装的)
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash

# Windows PowerShell
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.ps1 | iex
```

**代码示例(四种强度,按需切换)**  
```bash
/caveman          # 默认 full 模式,砍掉寒暄/解释性废话
/caveman ultra    # 电报体,极致压缩
/caveman wenyan   # 文言文,再省 30%
/caveman-commit   # Conventional Commit,主题 ≤50 字符
/caveman-review   # PR 一行评论,L42: 🔴 bug: user null. Add guard.
/caveman-stats    # 本 session 累计省了多少 token,可 --share 分享
```
压缩项目记忆文件:  
```bash
/caveman-compress CLAUDE.md
# → 706 → 285 tokens,节省 59.6%
```

**评价**  
本质是**写 prompt** 而非"框架",所以效果取决于底层 agent 的指令遵循度。但 README 公开的 benchmark 表格很诚实——10 个 prompt 平均省 65%(范围 22%~87%),`React Error Boundary` 这种"AI 默认超啰嗦"的题型省 87%。引用 March 2026 论文 "Brevity Constraints Reverse Performance Hierarchies" 说明"短 = 更准"。**推荐度:⭐⭐⭐⭐⭐(今天 trending 最值得 star 的"小而美")**。

---

## 3. msitarzewski/agency-agents — 把 Claude Code 延展成"一整间 AI 公司"(连续多日 Top 3)

**核心功能**  
由 Reddit 帖子孵化出的"AI 代理集":前端巫师、Reddit 社区忍者、文案注入师、现实检验员…… 每个代理都自带**人格 + 标准流程 + 可交付物**。新出了原生桌面 app(agencyagents.app),macOS/Linux/Windows 全支持,一键装到 Claude Code / Cursor / Codex / Gemini / OpenCode / Qwen / Osaurus。

**适用场景**  
- 个人/小团队的"AI 全能助理团队":一个会话同时承担 PM+设计+前端+运营
- 想把"零散 prompt"沉淀成"组织资产"——把工作流写进文件而不是对话里
- 教学/分享:演示"如何把一个 AI 工具用成公司形态"

**安装方式**  
```bash
# 方式 1:图形化(推荐)
brew install --cask msitarzewski/agency-agents/agency-agents

# 方式 2:CLI
git clone https://github.com/msitarzewski/agency-agents.git
./scripts/install.sh --tool claude-code
```

**代码示例(一个最简代理定义 `.claude/agents/frontend-wizard.md`)**  
```markdown
---
name: frontend-wizard
description: Senior frontend engineer (React/Next.js/TypeScript). Use proactively for UI tasks.
---
You are a senior frontend engineer with 10 years of React/Next.js experience.
- Always think mobile-first
- Prefer TypeScript + Tailwind
- Before writing code, ask clarifying questions about design tokens
- After writing code, output a short self-review checklist
```

**评价**  
理念对了:prompt 写在对话里会丢,写在文件里才是"组织资产"。**125k+ stars / 20k+ forks** 说明这不是小众玩具。**推荐度:⭐⭐⭐⭐(clone 一个下午挑有用的)**。

---

## 4. hasaneyldrm/exercises-dataset — 1,324 个动作的"开箱即用"数据集(连续多日 Top 5)

**核心功能**  
面向健身 App 开发者的脚手架:**1,324 个动作的结构化元数据**(分类、目标肌群、器械、说明、缩略图 ID、动画 GIF ID),**额外 6 种语言的说明翻译**(英、西、意、土、俄、中)。配套 `index.html`(浏览器交互查看)+ `setup.html`(数据库/API 一键生成模板)。

**适用场景**  
- 个人健身教练 App、健身 SaaS 的**冷启动数据**
- LLM/RAG 应用的动作知识库(直接灌进 vector DB)
- AI 教练 prompt 的"动作词典"基础
- 健身硬件(智能镜、家用器械)的"动作库"

**安装**(直接读 JSON 即可,无需 pip):  
```bash
git clone https://github.com/hasaneyldrm/exercises-dataset
cd exercises-dataset
python3 -c "import json; d=json.load(open('data/exercises.json')); print(len(d), d[0].keys())"
```

**代码示例(加载并查询)**  
```python
import json
exercises = json.load(open("data/exercises.json"))
# 找所有"杠铃 + 背部"的动作
back_barbell = [
    e for e in exercises
    if "back" in e["target"].lower() and e["equipment"] == "barbell"
]
for e in back_barbell[:5]:
    print(e["name"], "→", e["instructions"]["en"][0])
```

**评价**  
字段设计贴近 DB schema,直接映射成 SQL/NoSQL 表无压力。**图片/GIF 不打包**(避免版权纠纷),只保留 `media_id`,按需调 CDN——这是个**负责任**的设计。**推荐度:⭐⭐⭐⭐(健身类项目的"零成本种子数据")**。

---

## 5. santifer/career-ops — 把任何 AI CLI 变成"求职作战指挥中心"(空降 #5)

**核心功能**  
作者用它评估了 **740+ 个 offer、生成 100+ 份定制简历,最终拿下 Head of Applied AI 职位**。Career-Ops 把任何 AI 编程 CLI(Claude Code / Codex / OpenCode / Antigravity / Grok / Kimi / Qwen / Copilot)变成求职管道:
- **6 段式评估**:角色摘要、CV 匹配、级别策略、薪酬调研、个性化、面试 STAR+
- **ATS 优化 PDF 生成**:每份 JD 都生成专属 CV
- **45+ 公司门户扫描器**(Anthropic / OpenAI / ElevenLabs / Retool / n8n…)
- **批量处理** + **Go TUI 仪表盘** + **人工兜底**(系统从不替你提交申请)

**适用场景**  
- 海投焦虑者:让 AI 先帮你筛,把时间留给值得面试的少数岗位
- 想给简历"按 JD 定制"但没时间手动改的人
- 多语言求职者(README 官方翻译 12 种语言)
- 求职教练 / Career Counselor 做案例库

**安装方式**  
```bash
npx @santifer/career-ops init   # 一行克隆最新 release + 装依赖
cd career-ops
claude                           # 或 gemini/codex/opencode/qwen/agy/grok
```
首次启动会"以聊天的形式"问你 CV/偏好/目标岗位,**不用手改配置文件**。

**代码示例(自动流水线)**  
```bash
# 1) 把一个 JD URL 喂给 CLI
#    → 自动输出: 6 段评估 + ATS PDF + tracker entry

# 2) 批量扫 45+ 公司门户
/career-ops scan-mode

# 3) 查看 TUI 仪表盘
/career-ops tracker-mode

# 4) 生成 cover letter
/career-ops cover
```

**评价**  
**740 个 offer 真实跑出来的工具**,不是 demo——这比大多数"AI 求职工具"强一个数量级。**作者明确警告:不要用评分 <4.0 的 offer**——价值观对了。**推荐度:⭐⭐⭐⭐⭐(求职期 star;非求职期关注待下次换工作)**。

---

## 6. obra/superpowers — Coding Agent 的"软件开发方法论"(连续多日 Top 10)

**核心功能**  
一个**完整的 SDLC 方法论**,基于一组可组合的 skills,外加让 agent 主动使用这些 skills 的初始指令。流程:发现你要做东西时,**先停下来反问**真实目标 → 拆 spec 给你逐段确认 → 写出"热情的初级工程师都能 follow"的实现计划(强 TDD + YAGNI + DRY)→ 启动 **subagent-driven-development**,每个 task 自动 review、迭代。可以连续 2 小时自主运行而不跑偏。

**适用场景**  
- 任何用 Claude Code / Codex / Cursor / Pi / OpenCode 做"非 trivial 项目"的开发者
- 想给 AI 套上"靠谱工程师"行为模板的团队
- 把"作坊式 vibe coding"升级成"工程化 agent 工作流"

**安装方式(按不同 agent 装法不同)**  
```bash
# Claude Code
/plugin install superpowers@claude-plugins-official

# Codex App: 在 sidebar 点 Plugins → Coding 分类下找 Superpowers

# Antigravity
agy plugin install https://github.com/obra/superpowers

# OpenCode / Cursor / Factory Droid / Copilot CLI / Kimi Code 等都支持
```

**代码示例(用 superpowers 启动一个新项目)**  
```
User: "我想做一个命令行 todo 工具,支持同步到 Notion"
Agent:  (不会立刻写代码,而是先问 5 个澄清问题,然后给 3 段 spec,逐段求你 sign off)
        ↓
        (写 red/green TDD 计划)
        ↓
        (启动 subagent 跑实现 + 自动 review)
        ↓
        "3 小时后:测试全绿、CLI 可用、Notion 同步跑通"
```

**评价**  
**240k+ stars / 21k+ forks**,被 Anthropic 官方 plugin marketplace 收录,被 Primeradiant 商业赞助——这不是"又一个 prompt 集"而是**被生产验证过的工作流**。**推荐度:⭐⭐⭐⭐⭐(认真做项目必装)**。

---

## 7. ChromeDevTools/chrome-devtools-mcp — Google 把"DevTools"送给所有 Coding Agent(空降 #7)

**核心功能**  
官方 Chrome DevTools 团队出的 **MCP server**:让任意 coding agent(Claude / Cursor / Copilot / Antigravity)直接控制 + 检查**真实 Chrome 浏览器**。能力:性能 trace + 网络请求 + 截图 + console 消息(带 source-mapped stack trace)+ 可靠的浏览器自动化(Puppeteer 内核)。

**适用场景**  
- AI agent 帮你做**端到端调试**:"这个按钮点了之后 console 报错,帮我看 stack trace"
- AI 帮你做**性能优化**:录制 trace → 提取 actionable insights → 给出修复建议
- AI 帮你**自动截图 + 视觉回归**
- AI 帮你**复现跨页流程 bug**(登录 → 加购物车 → 结账 → 失败在哪一步)

**安装方式**  
在 MCP client 配置里加:  
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```
**轻量模式**(只做基础浏览器操作):  
```json
"args": ["-y", "chrome-devtools-mcp@latest", "--slim", "--headless"]
```

**代码示例(让 agent 用 DevTools)**  
```
User: "打开 staging.example.com,登录后点 'Add to Cart',然后告诉我 console 报了哪些错"
Agent: (调用 mcp__chrome-devtools__navigate → fill form → click →
       mcp__chrome-devtools__get_console_messages → 列出错误 + source map 反混淆的 stack)
```

**评价**  
Google Chrome DevTools 团队**官方出品**,意味着这是 agent 调试浏览器的**事实标准**——以后"AI 操作 Chrome"基本都靠它。需要注意的是:它会把浏览器内容暴露给 MCP client,**不要在含敏感数据的浏览器实例上跑**。**推荐度:⭐⭐⭐⭐⭐(前端 + 自动化测试开发者必装)**。

---

## 8. browser-use/video-use — 用 Claude Code 剪视频(连续多日)

**核心功能**  
把"raw 素材丢进文件夹 → 跟 Claude Code 聊 → 拿到 `final.mp4`"做成了产品级体验。能力:切除填充词(`umm`、`uh`、假启动)、自动调色、30ms 音频淡入淡出、自定义字幕风格、并发动画叠加(HyperFrames / Remotion / Manim / PIL)、自检渲染输出、跨 session 记忆(`project.md`)。

**适用场景**  
- 自媒体创作者:扔一堆 take 进文件夹,让 AI 剪成发布版本
- 公司内部教程、产品发布视频、远程会议录像
- 不想开 Premiere / Final Cut 的开发者(LLM 帮你操作 ffmpeg)

**安装方式**  
```bash
git clone https://github.com/browser-use/video-use ~/Developer/video-use
ln -sfn ~/Developer/video-use ~/.claude/skills/video-use
cd ~/Developer/video-use
uv sync
brew install ffmpeg
cp .env.example .env && echo "ELEVENLABS_API_KEY=sk_..." > .env
```

**代码示例(剪一个 launch video)**  
```
cd ~/Videos/launch-raw/
claude
> edit these into a launch video
# Agent: 清点源文件 → 提议剪辑策略 → 等你 OK → 产出 edit/final.mp4
```

**评价**  
"**LLM 从不看视频**,只通过 ElevenLabs Scribe 的**文字转录 + 时间戳 + 说话人分离**来读",然后在决策点(`timeline_view` PNG)调用一次"视觉合成"——这是把"30,000 帧 × 1500 tokens = 4500 万 token 噪音"压到"12KB 文字 + 几张 PNG"的关键设计。**推荐度:⭐⭐⭐⭐(创作者 + 视频重度用户)**。

---

## 9. actions/checkout — GitHub Actions 的"开仓第一刀"(v7 重磅安全更新)

**核心功能**  
GitHub 官方 action,几乎所有 workflow 第一步就用它:把仓库 check out 到 `$GITHUB_WORKSPACE`。**今天上榜是因为 v7 重大更新**——默认**拒绝 checkout fork PR 代码**(防止 `pull_request_target` 触发的"pwn request"漏洞),并迁移到 ESM 兼容新版本 `@actions/*`。

**适用场景**  
- **每一个** GitHub Actions workflow 的第一步
- 安全敏感项目(开 PR 的人写恶意脚本 → workflow 自动 checkout → 拿到 secret → 攻击)

**安装方式**(workflow YAML)  
```yaml
- uses: actions/checkout@v7
  with:
    # 如必须 checkout fork 代码,显式 opt-in(并自行评估风险)
    allow-unsafe-pr-checkout: true
    # 历史全拉(发布/版本对比需要)
    fetch-depth: 0
```

**代码示例(配合 v7 新安全策略)**  
```yaml
name: ci
on:
  pull_request_target:   # 这个 trigger 默认有 base repo secret 权限
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7   # v7 默认拒绝 fork PR checkout
      # 显式打开(已知风险)
      - uses: actions/checkout@v7
        with:
          allow-unsafe-pr-checkout: true
          ref: ${{ github.event.pull_request.head.sha }}
```

**评价**  
GitHub 终于为这个**用了 7 年的"危险默认"**打补丁——**强烈建议升 v7**。本仓库不接受 PR,只接受 security 报告。**推荐度:⭐⭐⭐⭐⭐(每个 workflow 立刻升)**。

---

## 10. affaan-m/ECC — Coding Agent 的"操作系统"级 skill 集

**核心功能**  
自称为 "**agent harness operating system**":skills + instincts + memory 优化 + 持续学习 + 安全扫描 + research-first 开发,**跨 7+ agent 平台**(Codex / Claude Code / Cursor / OpenCode / Gemini / Zed / Copilot)。**v2.0.0 新增 "Hermes operator" 层**:从 Hermes setup guide 出发,补 cross-harness 架构 + release notes。配套 GitHub App(ecc-tools)、Pro tier($19/seat/mo,支持私有 repo)、Claude Code plugin slug `ecc@ecc`。

**适用场景**  
- 想给 agent 配一套"开箱即用最佳实践"的开发者(避免每次开新项目都重新调 prompt)
- 多 agent 平台用户:一套 skill 跨平台用
- 企业用户:私有 repo 的 PR 自动审计

**安装方式**  
```bash
# 官方渠道:
# - GitHub repo (https://github.com/affaan-m/ECC)
# - npm: npm install -g ecc-universal / ecc-agentshield
# - GitHub App: https://github.com/apps/ecc-tools
# - Plugin slug: ecc@ecc
# - Website: https://ecc.tools

# 警告: README 顶部明文警告
# "Third-party re-uploads and unofficial mirrors are not maintained or
#  reviewed by the project and may contain malware."
```

**代码示例(启用 ECC skill)**  
```bash
# 在 Claude Code 里
/plugin install ecc@ecc
# 然后自动激活的 skills 包括:brainstorming / TDD / memory / research-first ...

# Hermes 模式启动新项目
hermes init "做一个 CLI 工具读取 yaml 配置并启动 docker 容器"
```

**评价**  
**211k+ stars / 32k+ forks / 230+ contributors**——单维护者能持续每周跨 7 平台发布,可见组织力。**OSS 永久 MIT**,Pro tier 养维护。**推荐度:⭐⭐⭐⭐(重度 agent 用户;轻量用户先观望)**。

---

## 11. HKUDS/Vibe-Trading — 港大实验室出的"个人交易 Agent"(连续多日)

**核心功能**  
港大数据智能实验室(DSAI 实验室)出品,**Vibe-Trading = "你的个人交易 Agent"**。核心能力:
- 自然语言描述策略 → 自动回测 → 信号生成 → 模拟盘
- **68 个 tool** + **18 个数据源**(免费 Eastmoney/Sina/Stooq/Yahoo + key-gated Finnhub/Alpha Vantage/Tiingo/FMP)
- **Research Autopilot**(Phase 3):Hypothesis → Signal Engine → Backtest **闭环**
- **10 个券商 connector**(Read-only 阶段,Trading 212 刚加入)
- **16 个 IM channel adapter**(Telegram / Slack / Discord / WhatsApp / 飞书 / 钉钉…)
- 450+ 学术 alpha 因子(Jegadeesh reversal / George-Hwang 52-week-high / Amihud illiquidity / Harvey-Siddique skew 等)
- Shadow Account:把提取的规则编成真实可执行的 SignalEngine(RSI 区间 + 5 日收益区间作为 PIT-safe entry context)

**适用场景**  
- 想"验证一个拍脑袋的交易想法是否真有 alpha"
- 学习量化交易的完整流程(NL 策略 → 指标 → 回测报告)
- 跑模拟盘 + 信号推送,作为真人交易的"第二意见"
- A 股 / 美股 / 港股 / 加密 都覆盖

**安装方式**  
```bash
pip install vibe-trading-ai
vibe-trading init
vibe-trading setup
vibe-trading dev    # 同时启动后端 (FastAPI) + 前端 (Vite React 19)
```

**代码示例(用自然语言定义策略)**  
```python
from vibe_trading import Strategy, run_backtest

strategy = Strategy(
    name="均线金叉 + 成交量放大 1.5x",
    universe="CSI300",
    entry="MA5 > MA20 && VOL_TODAY > 1.5 * MA20(VOL, 5)",
    exit="MA5 < MA20 OR drawdown > 0.05",
    stop_loss=0.03,
)
result = run_backtest(strategy, period="2020-01-01..2026-06-30")
print(result.sharpe, result.max_drawdown, result.annual_return)
```

**评价**  
今天 trending 榜单里**唯一仍然活着的"AI 量化交易"项目**(其他大多凉凉或 demo 状态)。**Commit 节奏极快**(最近 2 周每天都有 PR),**安全姿态稳**:所有 `place_order` 在结构性 paper/live 边界建好之前 hard-refuse。**推荐度:⭐⭐⭐⭐(认真搞量化的人 star)**。

---

## 12. agentskills/agentskills — Anthropic 出的"Agent Skills 开放规范"

**核心功能**  
**Anthropic 把自家的 Agent Skills 协议开源成开放标准**。核心就一条:**一个 skill 就是一个文件夹,里面有个 `SKILL.md`**(含 `name` 和 `description`),可以捆绑 `scripts/`、`references/`、`assets/`。Agent 通过 **progressive disclosure** 三阶段加载:
1. **Discovery**(启动时):只加载每个 skill 的 name + description
2. **Activation**(任务匹配时):加载完整 `SKILL.md` 到 context
3. **Execution**(执行时):按指令跑 + 可调用捆绑脚本/资源

**适用场景**  
- 想让 skill **跨多个 AI 产品复用**(一次写,Claude / Codex / Cursor / Windsurf 都能用)
- 给团队沉淀"领域知识 + 工作流 SOP"
- 构建"可审计 + 可版本控制 + 可发现"的 agent 能力集

**安装方式**(读 SKILL.md 即可,无需装)  
```bash
# 把你的 skill 放进 agents 期望的位置,例如 Claude Code:
mkdir -p ~/.claude/skills/finance-review
cat > ~/.claude/skills/finance-review/SKILL.md <<'EOF'
---
name: finance-review
description: Review financial reports following SOX-compliant checklist. Use proactively after any finance doc edit.
---
# Finance Review Checklist
1. Verify all numbers tie to source system
2. Flag any "round number" estimates
3. ...
EOF
```

**代码示例(skill 三阶段加载)**  
```python
# 伪代码:agent 启动
for skill in discover_skills():
    register(name=skill.name, desc=skill.description)  # ~50 tokens/skill

# 用户问"帮我审查这份财报"
if matches("finance-review", task):
    full_md = load(skill.path / "SKILL.md")  # 进入 context
    run_following_instructions(full_md)
```

**评价**  
**Anthropic 把"内部协议"做成开放标准**是个标志性事件——以后所有 major agent 都得支持(README 已列出大量 client)。**推荐度:⭐⭐⭐⭐(agent 工具开发者必读;普通用户可以等生态成熟)**。

---

## 13. openai/codex-plugin-cc — 在 Claude Code 里"叫 Codex 帮忙"

**核心功能**  
OpenAI 官方出的 Claude Code 插件:在 Claude Code 工作流里**直接调用 Codex** 做代码审查 / 任务委派。提供 6 个 slash 命令:`/codex:review`、`/codex:adversarial-review`、`/codex:rescue`、`/codex:transfer`、`/codex:status`、`/codex:result`、`/codex:cancel`。
- `/codex:review`:标准只读 review(支持 `--base main`、`--background`)
- `/codex:adversarial-review`:**可引导的**对抗性 review,质疑设计选型 / 风险点 / 备选方案
- `/codex:rescue`:**后台委派**一个任务给 Codex(investigate bug / 试 fix / 续接上次任务)
- `/codex:status` / `/codex:cancel`:管理后台 job

**适用场景**  
- Claude Code 重度用户,想"**两个模型互审**"(Claude 写 + Codex 审,或反之)
- 想在 Claude Code 内直接委派长任务给 Codex 后台跑
- 多模型混合策略:不同模型擅长不同任务

**安装方式**  
```bash
# 1) 加 marketplace
/plugin marketplace add openai/codex-plugin-cc

# 2) 装插件
/plugin install codex@openai-codex

# 3) 重载
/reload-plugins

# 4) 引导
/codex:setup
# (会自动问你 Codex 是否已装、是否已 login)
/codex:review --background
/codex:status
/codex:result
```

**代码示例(对抗性 review,质疑设计)**  
```bash
/codex:adversarial-review --background \
  "challenge whether this was the right caching and retry design, look for race conditions"
```

**评价**  
OpenAI **亲自下场**给竞品 Claude Code 做插件——这本身就是"AI IDE 之争"的一个标志性时刻。**要求 ChatGPT 订阅或 OpenAI API key**,Codex 用量会消耗你的 Codex 限额。**推荐度:⭐⭐⭐⭐(多模型混合党 star)**。

---

## 14. langflow-ai/langflow — 拖拽搭 AI agent 的老牌可视化工具(连续多日 Top 20)

**核心功能**  
可视化搭建 AI agent / workflow 的 IDE(React Flow 内核)。亮点:
- 拖拽式画布,所见即所得
- **每个 flow 自动暴露成 API**(FastAPI)+ **可作为 MCP server**(让其他 agent 调用你的 flow 当工具)
- 支持所有主流 LLM / 向量数据库
- 多 agent 编排 + 对话管理 + RAG
- 集成 LangSmith / LangFuse 做可观测性
- **Langflow Desktop** 把依赖打包好(Windows / macOS),免去 Python 环境管理

**适用场景**  
- 非程序员 / 产品经理 / 运营:**不用写代码**搭一个 RAG / chatbot / agent demo
- 工程师快速搭 PoC,验证想法后**导出 JSON** 嵌进 Python 项目
- 把 Langflow flow 当作 MCP tool 暴露给 Claude / Cursor

**安装方式**  
```bash
# 推荐方式:装 Desktop app(https://www.langflow.org/desktop)
# 命令行方式:
uv pip install langflow -U
uv run langflow run    # 启动在 http://127.0.0.1:7860
```

**代码示例(用 Python 组件扩展 Langflow)**  
```python
from langflow.custom import Component
from langflow.inputs import StrInput
from langflow.outputs import MessageOutput

class HelloWorldComponent(Component):
    inputs = [StrInput(name="name", display_name="Name")]
    outputs = [MessageOutput(display_name="Message")]
    
    def build(self):
        name = self.name
        self.status = f"Hello, {name}!"
        return self.status
```
把这个文件放到 Langflow components 目录,刷新画布就能拖出来用。

**评价**  
**150k+ stars**、3 年持续维护、与 LangChain 生态深度集成。**开源 + 商业版**双轨。**推荐度:⭐⭐⭐⭐(非程序员的 AI agent 入门首选)**。

---

## 15. pytorch/pytorch — 深度学习框架常青树(101k stars)

**核心功能**  
**PyTorch 本体**:Python 里的张量 + 动态神经网络 + 强 GPU 加速。**今天上榜大概是因为某个 release / 性能 PR**——pytorch/pytorch 偶尔会因一个热门 PR(性能优化、新 backend 支持、CUDA 适配)短暂登 trending。

**适用场景**  
- 任何深度学习研究 / 生产(PyTorch 现在是事实标准)
- 自定义算子(用 C++/CUDA 扩展 torch)
- 分布式训练(FSDP / DDP / DeepSpeed 集成)

**安装方式**  
```bash
# CPU only
pip install torch

# GPU(CUDA 12.x)
pip install torch --index-url https://download.pytorch.org/whl/cu124

# 验证
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```

**代码示例(一个最小训练循环)**  
```python
import torch, torch.nn as nn, torch.optim as optim

model = nn.Sequential(nn.Linear(784, 128), nn.ReLU(), nn.Linear(128, 10))
opt   = optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.CrossEntropyLoss()

for x, y in dataloader:                # x: [B, 784], y: [B]
    opt.zero_grad()
    loss = loss_fn(model(x), y)
    loss.backward()
    opt.step()
```

**评价**  
**不必多说**——任何 ML 工程师的"默认工具链"。trending 上偶尔出现一次,说明 PyTorch 社区依然活跃。**推荐度:⭐⭐⭐⭐⭐(ML 工程师的"水电煤")**。

---

## 16. harvard-edge/cs249r_book — Harvard 出品的"ML Systems"开源教材

**核心功能**  
Harvard Edge ML 实验室(Vijay Janapa Reddi 教授主导)出的 **ML Systems 课程开源仓库**,全套:
- **Textbook**(Vol I + Vol II,MIT Press 2026 出版)
- **TinyTorch**:从零**手写** PyTorch 内核(autograd / nn / optim)
- **Labs**:交互式 trade-off 推理
- **MLSys·im**:一个建模引擎,模拟"你租不起的 GPU 集群"
- **Hardware Kits**:真实硬件(Arduino / 手机 / 边缘设备)上跑 ML
- **StaffML**:评估你是否真懂
- **Socratiq**:AI 引导阅读 + 间隔重复

**适用场景**  
- 自学"ML Systems"(不是只教模型,而是教**系统工程**)
- 想"造自己的 PyTorch"的极客
- 大学老师拿这全套开一门课(textbook + slides + labs + kits)
- 面试准备:从模型到 deployment 全链路

**安装方式**(按教材分章节读,无 pip):  
```bash
git clone https://github.com/harvard-edge/cs249r_book
cd cs249r_book
# 读 README,选 TinyTorch / Labs / Hardware 任一路线
```

**代码示例(TinyTorch 第 1 课:手写 Tensor)**  
```python
# tinytorch/tensor.py
class Tensor:
    def __init__(self, data, requires_grad=False):
        self.data = data
        self.requires_grad = requires_grad
        self.grad = None
        self._backward = lambda: None
        self._prev = set()
    
    def __add__(self, other):
        out = Tensor(self.data + other.data, requires_grad=self.requires_grad)
        def _backward():
            if self.requires_grad: self.grad += out.grad
            if other.requires_grad: other.grad += out.grad
        out._backward = _backward
        out._prev = {self, other}
        return out
```

**评价**  
**Mission statement** 击中要害:"The world is rushing to build AI systems. It is not engineering them."——这句话值得每个 AI 工程师刻在显示器上。**目标:10 万学习者今年掌握 ML Systems,2030 年达到 100 万**。**推荐度:⭐⭐⭐⭐⭐(认真学 ML 的人 star)**。

---

## 17. ryanmcdermott/clean-code-javascript — 把 Clean Code 原则翻译成 JS(连续多日 Top 20)

**核心功能**  
Robert C. Martin 的 **《Clean Code》原则**用 JavaScript 重写一遍——不是 style guide,而是**生产可读、可复用、可重构**代码的指南。覆盖:变量 / 函数 / 对象与数据结构 / 类 / SOLID / 测试 / 并发 / 错误处理 / 格式 / 注释。

**适用场景**  
- 初级 JS 工程师的"必读清单"
- 团队 onboarding 材料
- Code review 时的 reference

**安装方式**(直接读,无需安装):  
```bash
git clone https://github.com/ryanmcdermott/clean-code-javascript
# 读 README(本身就是一篇完整文章)
```

**代码示例(SOLID 章节摘录)**  
```javascript
// ❌ Bad — 一个 function 干太多事
function createUser(name, email, password) {
  if (!validateEmail(email)) throw new Error('bad email');
  const hash = bcrypt.hashSync(password, 10);
  db.insert({ name, email, passwordHash: hash });
  sendWelcomeEmail(email);
  return { id: db.lastInsertId(), name, email };
}

// ✅ Good — 每个 function 一个职责
function createUser(input) {
  validateUserInput(input);
  const user = persistUser(input);
  notifyNewUser(user);
  return user;
}
```

**评价**  
**94k+ stars**——经典老项目,2024 年后基本不再大改,但 README 本身仍然是 JS 工程师**最值得读的 30 分钟**之一。**推荐度:⭐⭐⭐⭐(初级到中级 JS 工程师必读)**。

---

## 📊 今日速递小结

| 趋势 | 项目 | 备注 |
|------|------|------|
| 🆕 新进榜 | `JuliusBrussee/caveman`、`santifer/career-ops`、`ChromeDevTools/chrome-devtools-mcp`、`actions/checkout`(v7 重磅安全更新)、`affaan-m/ECC`、`agentskills/agentskills`、`openai/codex-plugin-cc`、`langflow-ai/langflow`、`pytorch/pytorch`、`harvard-edge/cs249r_book`、`ryanmcdermott/clean-code-javascript` | **11 个新面孔**,比昨天(5 个)翻倍 |
| 🔁 继续霸榜 | `usestrix/strix`(登顶)、`msitarzewski/agency-agents`、`hasaneyldrm/exercises-dataset`、`obra/superpowers`、`browser-use/video-use`、`HKUDS/Vibe-Trading` | 老牌热度延续 |
| 🧠 Agent Skill 协议化 | `agentskills/agentskills`、`affaan-m/ECC`、`obra/superpowers`、`JuliusBrussee/caveman` | "skill = 文件夹 + SKILL.md"成为事实标准 |
| 🤖 Coding Agent 互操作 | `openai/codex-plugin-cc`(在 Claude Code 里调 Codex)、`ChromeDevTools/chrome-devtools-mcp`(给 agent 浏览器) | 模型/工具之间的"组合战" |
| 🎓 教育/教材 | `harvard-edge/cs249r_book`、`ryanmcdermott/clean-code-javascript` | 长期价值型 |
| 📜 经典基础设施 | `pytorch/pytorch`、`actions/checkout` | 时不时回榜说明生态活跃 |

**今日金句**(自创):**GitHub Trending 的钟摆已经从"通用大模型"摆到"垂直 AI Agent 工具"**,今天榜单里有 11 个是"AI 干具体事"的代理/工具,而**`agentskills` 把"skill = 文件夹"做成开放标准**,可能是 AI Agent 生态走向成熟的第一个分水岭。

**数据来源**  
- 抓取 URL:<https://github.com/trending>  
- 抓取时间:2026-07-03 13:08 (Asia/Shanghai)  
- 抓取方式:GitHub REST API(`/repos/{owner}/{repo}` + `/repos/{owner}/{repo}/readme`) + readability 提取,排序依据 GitHub 官方"今日热门"默认顺序  
- 补充来源:<https://orangebot.ai/github-trending-today>(仅用于交叉验证)