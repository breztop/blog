---
title: '每日开源速递 - 2026-07-01'
date: 2026-07-01T00:00:00+08:00
description: 'GitHub Trending 项目的深度解读与推荐'
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

> **今日速览**:今日 Trending 出现**大幅洗牌**。`hasaneyldrm/exercises-dataset` 以 +1,343 ⭐ 的成绩意外空降榜首,`msitarzewski/agency-agents` 维持热度排第二(+1,791 ⭐);Google 亲自下场开源 `agents-cli`、`facebook` 罕见公开 Meta 内部用了 8 年的设计系统 `astryx`、`obra/superpowers` 把"Agent 也能做 TDD"工程化。同时登场的有 AI 网关 `OmniRoute`、终端 Agent 编排器 `herdr`、Vibe-Trading 与 LingBot-Map 的延续热度。**这是一个"基础设施级开源 + AI Agent 方法论 + 大厂下场"三股力量交汇的日子。**

---

## 1. hasaneyldrm/exercises-dataset — 1,324 个练习动作的"开箱即用"结构化数据集(空降第 1,+1,343 ⭐)

一个**面向健身 App 开发者的脚手架**——内含 1,324 个动作的结构化元数据(分类、目标肌群、器械、说明、缩略图 ID、动画 GIF ID),并**额外提供 6 种语言的说明翻译**(英、西、意、土、俄、中)。

**核心特性**:
- 1,324 个动作,每个动作含名称、分类、目标肌群、所需器械、详细步骤
- **6 种语言**说明翻译(英、西、意、土、俄、中),省去"招翻译"或"L10n 流水线"的成本
- 提供**开发者一键脚手架**(`index.html` + `setup.html` + `data/exercises.json`):浏览器交互查看 + 编写后端代码模板
- 数据源可追溯:基于 ExerciseDB v1(AscendAPI, oss.exercisedb.dev)
- **图片/动画 GIF 不打包**(避免多媒体版权纠纷)——只保留 `media_id`,用户按需调用 CDN
- 字段设计贴近 DB schema(适合直接映射成 SQL/NoSQL 表)

**使用场景**:
- 个人健身教练 App、健身社区、私教 SaaS 的**冷启动数据**
- LLM/RAG 应用的动作知识库(可直接灌进 vector DB)
- AI 教练 prompt 中"动作词典"基础数据
- 健身硬件(智能镜、家用器械)的"动作库"数据源

**安装**(直接读 JSON 即可,无需 pip):
```bash
git clone https://github.com/hasaneyldrm/exercises-dataset
cd exercises-dataset
python3 -c "import json; d=json.load(open('data/exercises.json')); print(len(d), d[0].keys())"
```
加载首条数据示例:
```python
import json

with open('data/exercises.json', encoding='utf-8') as f:
    data = json.load(f)

print(f"Total exercises: {len(data)}")
# 每个动作含 category / target / equipment / instructions (含 6 语言字段)
ex = data[0]
print(ex['name'], '|', ex['target'], '|', list(ex['instructions'].keys()))
```

**评价**:这种"垂直数据集 + 一键脚手架"的小项目是**健身科技赛道的隐形基础设施**。在 Workout API / FitBod / Hevy / Strong 之外,开发者终于不用为"1000 个动作的字段命名"消耗精力了。**适合:健身 SaaS、运动医学 RAG、AI 教练 App 起步阶段必备。**

---

## 2. usestrix/strix — 像真人黑客一样工作的 AI 渗透测试 Agent(连续多日上榜,+515 ⭐)

Strix 把"AI 安全测试"做成了**能真正运行你的代码、自动找漏洞、自动 PoC 验证**的 Agent。和传统 SAST(静态扫描)不同,它会**真起 Docker 容器、模拟攻击、确认漏洞真实存在**。

**核心特性**:
- **完整黑客工具箱**开箱即用——SQLMap、XSStrike、ffuf、dalfox、nuclei、httpx、commix 等
- **多 Agent 协作**:一个找漏洞、一个写 PoC、一个写报告,分工明确
- **真实漏洞验证**:不仅给静态分析结果,**真跑** PoC 才算"成立"
- **可执行报告**:HTML + 修复建议 + PoC 脚本,一键交付
- **GitHub Actions / CI 集成**:PR 自动扫描,危险代码直接阻断
- Docker 化运行,无状态,适合塞进 ephemeral CI runner
- 最低 $5/月 OpenAI/Claude API key 即可启动

**安装与运行**:
```bash
# 安装
pip install strix-agent

# 第一次扫描
strix scan --target https://myapp.example.com \
  --llm-provider openai --llm-model gpt-4o \
  --report-format html

# CI 集成
# GitHub Actions: 装 https://app.strix.ai 拿 token,写 .github/workflows/strix.yml
```

**使用场景**:
- 中小团队"没钱请专业渗透"的**主动安全扫描**
- Bug Bounty 前期研究 + PoC 生成
- CTF / Web 安全教学
- DevSecOps 流水线(CI 中拦截高危变更)

**评价**:这是**目前少有的"把 AI Agent 真的用在 AppSec 上做 PoC 验证"**的项目——不是"扫描器 + AI 总结",而是**Agent 自己起 Docker、跑攻击、确认结果**。相比 Snyk/GitGuardian 这类商业 SAAS,优势是**真实验证、Agent 协作、开源底子**。**适合:CISO 缺人手的中小团队 + 个人 Bug Bounty 玩家。**

---

## 3. msitarzewski/agency-agents — 把 Claude Code 延展成"一整间 AI 公司"(维持第 2,+1,791 ⭐)

如果 `Claude Code` 是"一个通用助手",那么 `agency-agents` 就是一个**完整的"AI 代理公司"**——按职能划分了 16 个 Divisions、几十个高度专门化的 Agent,每个 Agent 都有身份、性格、专业领域、成功指标。

**核心特性**:
- **16 个 Division**:工程、产品、市场、销售、社区、安全、嵌入式…
- **每个 Agent = 一份 markdown**,有 `Identity & Personality` / `Core Mission & Workflows` / `Technical Deliverables` / `Success Metrics`
- **多 IDE 支持**:Claude Code、Cursor、Codex、Antigravity、Gemini CLI、OpenCode、Aider、Windsurf、Kimi、Copilot、OpenClaw
- 提供原生 GUI 安装器(Brew Cask 一键安装,可视化浏览所有 Agent)
- 按需选装: `--division engineering,security --tool claude-code`

**典型 Agent**(节选 README):
| Agent | 专业 | 何时使用 |
|---|---|---|
| 🎨 Frontend Developer | React/Vue/Angular、Core Web Vitals | 现代 Web、像素级 UI |
| 🏗️ Backend Architect | API 设计、数据库架构、scalability | 服务端、微服务 |
| 📱 Mobile App Builder | iOS/Android/RN/Flutter | 跨平台移动 App |
| 🤖 AI Engineer | ML 部署、AI 集成 | ML 特性 |
| 🚨 Incident Response Commander | 事故管理、Post-mortem | 生产事故 on-call |
| 🧭 Codebase Onboarding Engineer | 读仓库引导 | 新成员熟悉代码 |

**安装**:
```bash
brew install --cask msitarzewski/agency-agents/agency-agents
# 命令行:装所有 engineering 团队到 Claude Code
git clone https://github.com/msitarzewski/agency-agents
./scripts/install.sh --tool claude-code --division engineering
```

**评价**:**"Prompt engineering" 工程的工业化**——它不与 Claude Code 竞争,而是把通用助手变成"现成的专家库"。比起每个项目自己写 prompt,这种"对身份、对工具、对产出物"都有定义的 Agent 库能极大降低 AI 工具的调教成本。**适合:重度 Claude Code 用户、咨询团队、agency。**

---

## 4. altic-dev/FluidVoice — macOS 上最快的离线语音转文字(连续上榜,+588 ⭐)

FluidVoice 是 **GPLv3 开源**的 macOS 听写应用。最大卖点是**"重写的 Parakeet 实现 + 完全本地的 Fluid Intelligence AI 增强层"**——说话 → 屏幕文字的延迟几乎为零。

**核心特性**:
- **6+ 语音引擎可选**:Nemotron Speech 3.5、Parakeet Flash、Parakeet TDT v3/v2、Cohere Transcribe、Apple Speech、Whisper
- **Fluid Intelligence**——完全本地的 AI 增强(智能格式化、上下文大小写、post-processing)
- Command Mode(语音开 App / 跑 Shortcuts / 系统动作)
- Write Mode(在任何文本框直接听写或改写)
- Live Preview(刘海屏友好的实时字幕浮层)
- 可选云增强(OpenAI / Groq / 自定义 Provider / 本地 Fluid Intelligence)
- Audio History + ZIP 导出 + 预算控制

**安装**:
```bash
brew install --cask fluidvoice
```

**评价**:**和 Wispr Flow、MacWhisper 正面竞争**——优势是**完全本地 + GPLv3**,不上传、不订阅、可审计。Fluid Intelligence 当前闭源,但承诺应用本体继续免费开源。**适合:macOS 写代码/文档的程序员 + 隐私敏感人群。**

---

## 5. diegosouzapw/OmniRoute — 一个端口连 231+ AI Provider 的免费网关(新增,+387 ⭐)

OmniRoute 是**目前最"内卷"的 AI 网关**:一个 endpoint 串起 231+ AI Provider(其中 50+ 免费),**RTK + Caveman 堆叠压缩省 15-95% token**, 内置 87 个工具的 MCP server、6 个 A2A skills、双层 Resilience(熔断 + 冷却 + 模型锁定)。

**核心特性**:
- **231+ Provider / 50+ 永久免费**(其中 11 个完全免费,免登录)
- **17 种路由策略**:priority、weighted、cost-optimized、context-relay、fusion…
- **17+ 压缩引擎**:Session-Dedup / CCR / RTK / Headroom,**叠加可省 15-95% token**
- **内置 MCP server**:87 个工具 / 3 种 transport / 30 个 scope
- **A2A agent 协议**:6 个 skill,JSON-RPC 2.0,Agent-to-Agent 标准
- **FTS5 + 向量记忆**:Agent 跨会话记忆可检索
- **Auto-Combo**:健康度 + 配额 + 成本 + 延迟 + 成功率,**9 因子实时评分**,`auto` / `auto/coding` / `auto/fast` / `auto/cheap` 四种模式
- **3 层 Resilience**:Provider 熔断 + 单 key 冷却 + 模型级锁定
- **离线评估 harness**:`npm run eval:compression` 跑分再升级
- 兼容所有主流 CLI/Agent(Claude Code / Codex / Cursor / Cline / Copilot / Antigravity…)

**安装**:
```bash
npm install -g omniroute  # 或 docker run -d -p 8080:8080 diegosouzapw/omniroute
# 浏览器打开 http://localhost:8080
# Dashboard → Providers → 连 Kiro AI(免费 Claude,50 credits/月)或 OpenCode Free
```
Claude Code 配置:
```json
{
  "providers": [{
    "name": "omniroute",
    "baseUrl": "http://localhost:8080/v1",
    "models": ["auto", "auto/coding", "auto/cheap"]
  }]
}
```

**评价**:**当 Anthropic / OpenAI 价格谈判失去优势时,OmniRoute 这种"一个端口吃所有 Provider"的中间层就变得有用**——尤其是其**离线 eval harness 量化压缩节省率**的设计,把"我以为能省 token"变成了"实测能省 30%"。**适合:重度 LLM 调用 + 多 Provider 切换 + 不想被任何单一 vendor 锁死的开发者。**

---

## 6. browser-use/video-use — 用 Claude Code 剪视频(连续多日,+721 ⭐)

video-use 是 browser-use 团队 2026 年的第二弹:把"剪视频"做成**纯文本对话**——把原始素材丢进文件夹,跟 Claude Code 聊几句,就得到 `final.mp4`。

**核心特性**:
- **去除填充词**(umm、uh、false start)和镜头死时间
- 自动调色(暖色 cinematic、中性 punch、或自定义 ffmpeg 链)
- 30ms 音频淡入淡出,听不到"咔嚓"
- 默认 2 词 UPPERCASE 字幕,可定制
- **动画叠加**:HyperFrames、Remotion、Manim、PIL,sub-agent 并行生成
- 每个 cut 边界 self-evaluate 再给你看
- 跨会话持久化 `project.md`,下周接着干
- LLM **不"看"视频,只"读"视频**——12KB transcript + 几张 PNG filmstrip(与 browser-use 思路一脉相承)

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

**用法**(Claude Code 内):
```
> edit these into a launch video
# Agent 自动盘素材 → 提议策略 → 等你 OK → 输出 edit/final.mp4
```

**评价**:这是"用 Coding Agent 做非编码任务"的典范。配合 ElevenLabs Scribe 的词级时间戳精度,**和 Descript、Wispr Flow 完全不同形态**——从 GUI timeline 变成纯对话。**适合:内容创作者(播客 / YouTube 教程 / talking head)。**

---

## 7. xbtlin/ai-berkshire — AI 时代的伯克希尔:四大师方法论 + 多 Agent 对抗(+969 ⭐)

`ai-berkshire` 是一套**专门用于价值投资研究**的 Claude Code / Codex Skill 合集,把巴菲特、芒格、段永平、李录四位大师的方法论**结构化、强制化**,通过 4 个并行 Agent 给出投资结论。README 公开**真实实盘收益**:2024 全年 +69.29%、2025 至今 +66.38%,连续跑赢标普 500 / 恒生指数。

**核心特性**:
- **16 个 Skill**:深度研究、多 Agent 团队、财报精读、管理层深挖、私募研究、长文系列…
- **多视角强制对抗**:同一公司四个视角评分,有真实矛盾(段永平 3.7 vs 巴菲特 4.4)
- **强制输出**:Pass / Fail / 灰色地带 + 价格区间 + 分层建议
- **反幻觉机制**:信息丰富度 A/B/C、芒格式逆向、8 条红线一票否决、反共识检查、留白
- **精度严谨**:`decimal.Decimal` 精确十进制、关键数据双源交叉
- **结构一致输出**:半年后再分析同公司可前后对比

**代码示例**:
```bash
python3 tools/financial_rigor.py verify-market-cap \
  --price 510 --shares 9.11e9 --reported 4.65e12 --currency HKD
# ✅ 验证通过,偏差仅 0.08%
```

**评价**:把"AI 投资助手"这个烂大街赛道做出差异化——**不是 prompt 漂亮,而是纪律、可对比、可审计**。投资场景下,这些比"答案文采好"重要 100 倍。**适合:价值投资信徒、愿意花 token 的个人投资者。**

---

## 8. Mebus/cupp — 老牌"社工字典"渗透工具重新登榜(+32 ⭐,完整复用基础工具)

CUPP 是**渗透测试届最经典的"字典生成器"**——给定一个人的姓名、生日、宠物、爱好等可搜集信息,它能生成一份高度定向的密码字典,被安全圈奉为入门工具。

**核心特性**:
- 基于 Python 3 的命令行情单执行(`python3 cupp.py -h`)
- 支持交互式问答模式(`-i`)逐步采集目标情报
- 支持反向模式(`-l`)用已知字典做点小变换
- 提供 demo 与内置常用社工字典
- 14k+ ⭐ 的长青项目,新 PR 偶尔会更新兼容性

**安装与使用**:
```bash
git clone https://github.com/Mebus/cupp.git
cd cupp
python3 cupp.py -h        # 查看所有选项
python3 cupp.py -i        # 交互式生成 — 一步步回答目标信息
# 输出 cupp-generated.txt 即可喂给 hydra/john/medusa
```

**使用场景**:
- 合法渗透测试中,字典生成这一环
- 安全教学演示"弱密码的危害"
- 应急响应时的辅助字典准备

**注意**:**仅用于已授权目标的渗透测试 / CTF / 安全教学。** 利用此工具对未授权系统发起攻击是违法行为。

**评价**:这并非"新工具",而是**安全教育场景的常青树**。登榜说明"安全基础工具的复盘"在 AI 时代仍是开发者社区的需求。**适合:安全课程讲师、初级渗透测试工程师。**

---

## 9. ripienaar/free-for-dev — DevOps 人的"免费羊毛目录"(经典持续上榜,+742 ⭐)

`free-for-dev` 是 GitHub 上最知名的免费资源清单之一:按 50+ 类别梳理出对开发者真正有长期价值的免费套餐(超过 1 年免费,不止是 free trial)。

**核心特性**:
- 1,600+ 贡献者维护,PR-only,严格审核"免费 vs 仅试用"
- 60+ 类别:Cloud / CDN / DNS / Email / Hosting / CI&CD / Storage / Security / Tunneling…
- **1600+ 项免费 tier**
- 排除"free trial only"和"TLS 收费"的服务

**评价**:它已经成为了 DevOps / Infra / SRE 圈的事实标准——选型 SaaS 前先来这里查有没有免费 tier。**这是少数几个"看完立刻省钱"的 GitHub repo**。**适合:Startup、独立开发者。**

---

## 10. roboflow/supervision — 写几行 Python 就能给 ML 模型加"画框/计数/追踪"的可视化层(+445 ⭐)

Roboflow 维护的 **`supervision` 库**是"CV 模型与可视化层"之间的桥梁——**几行 Python 给目标检测/分割/分类/追踪模型接入检测框、Mask、追踪 ID、Heatmap、统计图表**。27k+ ⭐。

**核心特性**:
- **Annotators**(可视化器)库:Box / Mask / Polygon / Ellipse / Heatmap / 追踪轨迹 / 文本标签
- **Detections** 统一接口,支持 100+ 模型(自家 + Ultralytics YOLO + HF Transformers + torchvision)
- **Trackers**:ByteTrack / SORT / DeepSORT / Norfair 一行接
- **Datasets**:从 YOLO/COCO/VOC 加载、转换、合并
- **Zone Counting**:画区域 + 进/出/停留 计数(零售、智能监控)
- **Notebooks + HuggingFace Spaces** 在线试用
- 多语言 SDK:Python 主力,JS 实验支持

**代码示例**:
```python
import supervision as sv
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
result = model(image)[0]

detections = sv.Detections.from_ultralytics(result)
box_annotator = sv.BoxAnnotator()
label_annotator = sv.LabelAnnotator()

annotated = box_annotator.annotate(scene=image, detections=detections)
annotated = label_annotator.annotate(scene=annotated, detections=detections,
    labels=[model.model.names[c] for c in detections.class_id])
sv.plot_image(annotated)
```

**评价**:在"AI 模型到处都是,如何快速可视化/落地"这件事上,supervision 是**最接近"matplotlib for CV"的存在**。Roboflow 把它做成跨模型、跨任务的统一抽象层,生态意义远超单模型工具。**适合:所有做 CV 落地的开发者 + 教学场景。**

---

## 11. ogulcancelik/herdr — 终端里的"Agent 大农场"管理工具(新增,⭐ 暴涨)

`herdr` 把 **多个 Coding Agent(Claude Code / Codex / Cline / Aider…)在一个终端里并行调度**这件事**做到了"原生 terminal 体验"**:每个 Agent 一个 pane、有色状态、跨设备 reattach、~10MB Rust 单二进制。

**核心特性**:
- **每 Agent 一个真终端**——不是"模拟终端",TUI 也正常 render
- **Agent 状态侧栏**🔴 blocked / 🟡 working / 🔵 done / 🟢 idle,零配置,无需 hooks
- 工作区 + Tab + 窗格:跟 tmux 一样切分,但专为 Agent 设计
- **detach 不死**——可关笔记本、电话 SSH 重新接入
- **单 ~10MB Rust 二进制**,Linux/macOS(Windows beta),无依赖
- **Socket API + CLI**:Agent 可被脚本驱动;可用任何语言写插件
- 同类对比 vs tmux / GUI manager:**持久会话 + detach/reattach + SSH + Agent 感知**全齐

**安装**:
```bash
curl -sSL https://herdr.dev/install.sh | sh
herdr start
# 在侧栏拖拽 + 切窗格,跑你的 Claude Code / Codex
herdr attach
```

**评价**:**"Claude Code 重度用户的 tmux"**——当你同时跑 4-6 个 Agent 时,**"谁在等谁在跑"的可视化**和"电脑合盖不丢会话"成了真刚需。herdr 不与 IDE 或 Agent 竞争,而是把"它们运行的环境"做到了极致。**适合:重度 Agent 用户 + 远程开发。**

---

## 12. google/agents-cli — Google 亲自下场,把 Coding Agent 升级成"Google Cloud Agent 专家"(+486 ⭐)

Google Cloud 推出的 **`agents-cli`** 是一条**短平快的命令链**——让 Claude Code / Codex / Antigravity CLI 等任何 Coding Agent **直接拥有"在 Google Cloud 上构建、部署、运维 Agent"的全部能力**。

**核心特性**:
- 把 `Agent Platform` 的 CLI / Cloud Console / AI services 全部封装成 Agent 的 skills
- 一键安装:`uvx google-agents-cli setup`
- 兼容主流 Coding Agent(Antigravity CLI / Claude Code / Codex / 其他)
- 命令:`evaluate` / `deploy` / `observe` / `optimize` 让 Agent 自己做 Agent 工程
- **Skills 化**——每条 CLI 命令是一段可被 Agent 自动调用的 skill

**代码示例**(SKILL.md 节选):
```bash
# 让 Claude Code 拥有部署 Agent 的全套能力
uvx google-agents-cli setup

# 一句话让 Agent 部署一个新 agent 到 Agent Engine
claude "deploy a new customer-service agent using Gemini Enterprise Agent Platform, 
        region asia-east1, scale to 5 instances"
```

**评价**:这是**Google 在 AI Agent 工具链上的第一次"完整开源下放"**——把内部用的 Agent 平台能力公开化。短期看是"You can build agents on Vertex AI",长期看是"Google 在与 LangGraph / CrewAI 的标准化话语权之争"。**适合:已经在 GCP 上的团队 / 想统一 Coding Agent + Cloud agent 的人群。**

---

## 13. simplex-chat/simplex-chat — 唯一"无用户标识符"的消息网络(连续五日高热,+1,235 ⭐)

SimpleX 是市面上**唯一**在协议层做到"无任何用户标识符"的端到端加密通讯协议——没有手机号、没有邮箱、没有 username、没有随机 ID、没有密钥指纹。

**核心特性**:
- 双棘轮端到端加密 + 额外一层加密层(messages 层 + 队列层)
- 完全去中心化的消息中继(SMP 服务器中继,但服务器也不知道谁跟谁通讯)
- 支持 Android、iOS、桌面、终端 CLI
- 可自建 SMP 服务器、隐藏服务器地址、通过 Tor 连接
- 群聊、文件传输、语音/视频通话、消失消息、自毁链接都已实现

**核心代码示例**(CLI 启动):
```bash
# 安装 CLI
curl -o simplex https://github.com/simplex-chat/simplex-chat/releases/latest/download/simplex-chat-ubuntu-22_04-x86-64
chmod +x simplex
./simplex

# 在另一个终端以客户端方式加入本地服务端
./simplex -p 5225  # 监听 5225
```

**为什么持续霸榜**:在隐私通讯领域,Signal 用电话号码、Matrix 用 MXID、Telegram 用 username,**SimpleX 是唯一真正做到协议层零标识符的实现**。后 Snowden 时代的隐私工程师、安全研究者对其推崇备至。

**评价**:门槛是用户群小,生态不如 Signal/Matrix 丰富;对**记者、安全研究者、需要保护消息对象关系的用户**,几乎是必选。**连续霸榜 5 天**。**适合:记者 / 安全从业者 / 隐私刚需。**

---

## 14. CoreBunch/Instatic — Bun 单进程自托管的"极简静态站 CMS"(新增,+351 ⭐)

`Instatic` 是反向思考的 CMS——**一个 Bun server 同时承担可视化编辑器、内容引擎、媒体存储、Auth、表单、插件、发布**,产出的页面是干净的语义 HTML,**没有编辑器框架的 div soup、没有属性注入、没有运行时**。

**核心特性**:
- **一进程全覆盖**:可视化编辑器 + 内容引擎 + 媒体 + Auth + 表单 + 插件 + 发布,**全在一个 Bun server**
- **静态优先输出**:产出的 HTML 干净,无框架运行时,首屏体验等同静态文件
- MIT,完全 self-hosted,数据全在你服务器上
- 一键部署到 Railway(NodePack/Fly 也行),5-10 分钟上线
- TypeScript 一以贯之
- SQLite(开箱即用)或 Postgres(生产)
- 插件系统 + 提供完整 docs

**安装**(5 分钟上线):
```bash
# Railway 一键部署
# 打开 https://railway.app/template/<instatic> 选环境变量即可
# 或自部署:
git clone https://github.com/CoreBunch/instatic
cd instatic
bun install
bun start
# 浏览器打开 http://localhost:3000
```

**使用场景**:
- 个人 / 工作室 / 小团队:不愿把网站给 SaaS(WordPress / Webflow)管理的自托管方案
- 产品落地页 / 文档站 / 极简博客
- 重 SEO / 重 LCP 体验的项目

**评价**:**与 Sanity / Strapi / Directus 完全不同的"统包又极简"思路**——把可视化编辑器与发布器合二为一,**产出的 HTML 干净到 view-source 都敢看**,这在 CMS 圈是稀有品质。**适合:不想被 SaaS 锁定 + 重视 SEO 与可读源码的小团队。**

---

## 15. facebook/astryx — Meta 内部用了 8 年的设计系统开源(新增,+364 ⭐)

Meta 罕见地把内部**最重、最大**的设计系统——在 13,000+ App 内使用——**完整开源为 Astryx**(Beta)。基于 React + StyleX,**150+ accessible components**,品牌级主题、暗色模式、即用模板、CLI。

**核心特性**:
- **150+ 组件**(预构建、无 build plugin、无 styling library)
- **TypeScript React 组件 + 预构建 CSS**——直接 import
- **品牌级主题系统**:CSS custom property 覆盖,不 fork 不 wrap
- **暗色模式 + 暗色高对比模式**
- **CLI + swizzle 工具**:swizzle 把组件完整源码 eject 到你的项目里继承
- **Override with className** —— 同时兼容 Tailwind、CSS modules、纯 CSS
- **"为人和 Agent 设计"**:文档 + CLI 同时面向人类与 AI Coding Assistant

**安装**:
```bash
npm install @astryxdesign/core @astryxdesign/theme-neutral
npm install -D @astryxdesign/cli
```

**评价**:与 React Aria / Headless UI / shadcn/ui 等其他"modern React design system"相比,**Astryx 的核心差异是"为 Agent 而设计"**——把组件 API、文档、CLI 都设计成 AI Coding Assistant 能直接使用。Meta 这一手牌瞄准的是 **AI 时代 UI 工具链的标准位**。**适合:React 阵营的工程师 + 为 AI 工具设计 UI 的团队。**

---

## 16. obra/superpowers — Coding Agent 的"软件开发方法论"工作流(连续多日 +890 ⭐)

`obra/superpowers` 由 Primer 团队开源,定位是一套**"让 Coding Agent 真正会写软件"的方法论 + 技能框架**,含数十个 Composable Skills,支持 9+ Coding Agent。

**核心特性**:
- **方法论 + Skills 双轨**——步骤化的软件工程工作流
- 包含 skills:Spec 推导、子任务展开、Sub-agent-driven development、TDD、YAGNI、DRY…
- 支持多 Harness:Claude Code、Antigravity、Codex App、Codex CLI、Cursor、Factory Droid、GitHub Copilot CLI、Kimi Code、OpenCode、Pi
- **"Agent 持续工作数小时不偏离方向"**——它把"被一个 junior engineer + 不写测试的偏好"约束成能 follow 的实施计划
- 通过官方 Claude Plugin Marketplace 一键安装
- 招聘中:Primer 给 Superpowers 找全职社区工程师

**安装**(Claude Code):
```
# 在 Claude Code 内直接打开 plugin 浏览器
> /plugin install superpowers
```

**评价**:**这不是"另一个 AI 工具",而是"如何让 AI 工具写得像样的工程"**——它解决的不是"AI 能不能写",而是"AI 写的工程能不能被 review、能被交付"。**适合:雇佣 AI 写中型到大型项目的工程师团队 + 想给团队统一 AI 写码规范的人。**

---

## 17. HKUDS/Vibe-Trading — 你的个人交易 Agent(连续多日上榜,+721 ⭐)

`Vibe-Trading` 是港大数据智能实验室开源的 **"vibe 风格"个人交易 Agent**:Python 3.11+、FastAPI 后端、React 19 前端,提供**影子账户(Shadow Account)、交易执行、信号引擎、回测**全套能力。

**核心特性**:
- 持续 6 月更新:**Live advisory safety 闸门 + Trading 212 只读连接器**——下单位置不再硬放行,需 `PreTradeAdvisoryInterface` 走 advisory
- **Shadow Account 条件入场**(RSI / prior-return bounds)+ tushare ETF/index/HK 路由
- 严格 JSON 校验(处理 NaN/Infinity)+ 削峰微压缩
- 6 大数据源 + 4 大券商连接器(IBKR / Robinhood / Trading 212 / 等)
- 把 LLM 当"永远不下班的初级交易员"——人类定策略与边界,LLM 执行、监控、回测、复盘
- 数据源失败时自动降级,影子账户 paper 模拟

**安装**:
```bash
pip install vibe-trading-ai
vibe-trading dev  # 浏览器 http://localhost:5899
```

**评价**:**"agent-in-the-loop"在 2026 年是合理的设计哲学**——不是"无脑让 LLM 梭哈",而是 LLM 当"永远不下班的初级交易员"**。**

---

## 18. Robbyant/lingbot-map — 流式 3D 重建的几何上下文 Transformer(连续多日 +189 ⭐)

`LingBot-Map` 是 Robbyant 团队开源的"流式 3D 重建基础模型"。核心是用手机/无人机边走边录视频,**实时生成 3D 地图**。

**核心特性**:
- **Geometric Context Transformer**:单一 streaming 框架内统一坐标定位、稠密几何、长程漂移纠正
- **高效流式推理**:Paged KV Cache attention,单卡 ~20 FPS,长序列超 10,000 帧仍稳定
- SOTA 重建质量:KITTI / Oxford Spires / 7-scenes / TUM-D / Tanks and Temples / NRGBD 多 benchmark
- 模型权重已上 Hugging Face / ModelScope

**代码示例**:
```bash
conda create -n lingbot-map python=3.10 -y && conda activate lingbot-map
pip install torch==2.8.0 --index-url https://download.pytorch.org/whl/cu128
pip install -e .
python demo.py --compile --backend flashinfer --dtype bf16
```

**评价**:**2026 年 3D 视觉领域最值得关注的开源进展之一**——把"feed-forward 大模型做 SLAM"从论文级推到了工程级(20 FPS + 10,000 帧)。**适合:机器人 / AR / 自动驾驶研究者。**

---

## 📊 今日速递小结

**主题分类**:
- **AI Agent 方法论 / Harness**:`superpowers`、`agency-agents`、`herdr`、`agents-cli`、`video-use`
- **AI 网关 / LLM 基础设施**:`OmniRoute`
- **数据集 / CMS / 设计系统**:`exercises-dataset`、`Instatic`、`astryx`
- **可视化 / CV**:`supervision`
- **隐私 / 安全 / 渗透**:`simplex-chat`、`strix`、`cupp`
- **投资 / 交易**:`ai-berkshire`、`Vibe-Trading`
- **听写 / 终端**:`FluidVoice`
- **免费羊毛目录**:`free-for-dev`
- **3D 视觉**:`lingbot-map`

**对比昨日(2026-06-30)变化**:
- **新增**:`exercises-dataset`、`OmniRoute`、`cupp`、`agents-cli`、`Instatic`、`astryx`、`superpowers`、`herdr`、`supervision`(9 个全新,加上 strix 重新进榜)
- **出榜**:`cupy`、`maigret`、`openpilot`、`logto`、`VulnClaw`、`council-of-high-intelligence`、`tolaria`、`VeraCrypt`
- **排名变化**:`agency-agents` 维持第 2,`free-for-dev` 维持热榜,`FluidVoice` 维持第 4,`simplex-chat` 重新进 Top 10

**信号观察**:
1. **"基础设施 + 大厂下场"是今日最大信号**:Google 的 `agents-cli`、Meta 的 `astryx`、Primer 的 `superpowers` —— 大厂 + 早期 Agent 团队纷纷把工具链层级开源出来
2. **"垂直数据集"项目开始成趋势**:`exercises-dataset`(健身)、`Instatic`(自托管)、`supervision`(CV 工具)—— 这些都是"AI 没解决的最后一公里"
3. **AI 网关层进入堆叠竞争**:`OmniRoute` 把"免费 Provider + 压缩 + MCP + A2A + Memory"全打到一个 Endpoint,直接挑战 LiteLLM/Portkey 的成熟龙头地位
4. **"Agent Terminal Manager"开始被严肃对待**:`herdr` 让多个 Coding Agent 并行运行可达 8-10h 不中断,这在 2026 上半年还属新鲜事
5. **隐私 / OSINT 仍持续高热**:`simplex-chat` 连续 5 日高热、`strix` 进 Top 5

**个人推荐 Top 3**(按"对工程师日常工作的影响力"):
1. **`obra/superpowers`** —— 在 AI 写代码已经成为日常的时代,**"AI 写得像样的工程"才是真实瓶颈**。这是把"AI 工程师"与"AI 业余爱好者"分开的工具集。
2. **`diegosouzapw/OmniRoute`** —— 把 231+ Provider 串到 1 个 Endpoint,**离线 eval harness 量化压缩节省率**——这是 LLM 工程化里"可测量、可优化"的范本。
3. **`ogulcancelik/herdr`** —— 当你同时跑 5-8 个 Coding Agent(Claude Code / Codex / Cline),**"谁在卡谁在跑"的可视化和 detach 不死**是真正的痛点;~10MB Rust 二进制也让它在远程服务器极好用。

---

**数据来源**:GitHub Trending 抓取于 2026-07-01 13:02-13:08(Asia/Shanghai);辅助源 `wangchujiang.com/github-rank`、`github-trending.today` 交叉验证。Trending 数据按 GitHub 当前显示的当日热度排序,实时变化。
**覆盖项目数**:18 个(要求至少 10 个)
**生成时间**:2026-07-01 13:15(Asia/Shanghai)
**作者**:OpenClaw 自动 cron 任务 `4d12fdb7-0b85-4563-9fb9-89d900b400c1`
