---
title: "每日开源速递 - 2026-03-11"
date: 2026-03-11T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-03-11

今天是 2026 年 3 月 11 日，AI Agent 生态继续蓬勃发展。本周 GitHub Trending 上的热门项目呈现出一个鲜明的趋势：**多智能体系统、自学习代理、AI 测试工具和低门槛 LLM 训练框架**。下面让我们深入探索这些改变开发方式的开源项目。

---

## 🚀 1. Agency Agents - 组建你的 AI 虚拟团队

**仓库**: [`msitarzewski/agency-agents`](https://github.com/msitarzewski/agency-agents)  
**星标数**: 24,748 (今日 +6,235)  
**编程语言**: Shell  
**许可证**: MIT

### 核心功能

Agency Agents 是一个精心设计的 **AI 代理个性集合**，从前端开发者到 Reddit 社区专家，每个代理都有独特的性格、工作流程和可交付成果。这个项目的灵感源自 Reddit 上的讨论帖，经过数月迭代而成。

想象一下：组建一支梦想团队，但它们从不休息、从不抱怨，而且始终交付结果。

### 主要特性

- **专业化分工**: 每个代理在其领域拥有深厚专业知识，而非通用提示模板
- **人格驱动**: 独特的声音、沟通风格和做事方式
- **可交付成果导向**: 真正的代码、流程和可衡量成果
- **生产就绪**: 经过实战检验的工作流程和成功指标
- **跨工具集成**: 支持 Claude Code、Cursor、Copilot、Aider、Windsurf 等主流工具

### 内置代理类型

#### 工程类
| 代理 | 专长 | 适用场景 |
|------|------|----------|
| Frontend Developer | React/Vue/Angular, UI 实现，性能优化 | 现代 Web 应用、像素级 UI、Core Web Vitals 优化 |
| Backend Architect | API 设计、数据库架构、可扩展性 | 服务端系统、微服务、云基础设施 |
| Mobile App Builder | iOS/Android、React Native、Flutter | 原生和跨平台移动应用 |
| AI Engineer | ML 模型、部署、AI 集成 | 机器学习功能、数据管道、AI 驱动应用 |
| DevOps Automator | CI/CD、基础设施自动化、云运维 | 流水线开发、部署自动化、监控 |
| Security Engineer | 威胁建模、安全代码审查、安全架构 | 应用安全、漏洞评估、安全 CI/CD |

#### 设计类
| 代理 | 专长 | 适用场景 |
|------|------|----------|
| UI Designer | 视觉设计、组件库、设计系统 | 界面创建、品牌一致性、组件设计 |
| UX Researcher | 用户测试、行为分析、研究 | 理解用户、可用性测试、设计洞察 |
| Whimsy Injector | 个性、惊喜元素、趣味交互 | 增加乐趣、微交互、彩蛋、品牌个性 |

#### 营销与增长类
| 代理 | 专长 | 适用场景 |
|------|------|----------|
| Growth Hacker | 快速用户获取、病毒循环、实验 | 爆发式增长、用户获取、转化率优化 |
| Content Creator | 多平台内容、编辑日历 | 内容策略、文案写作、品牌故事 |
| PPC Campaign Strategist | Google/Microsoft/Amazon Ads | 账户搭建、预算分配、扩展、性能诊断 |

### 安装方式

```bash
# 将代理复制到你的 Claude Code 目录
cp -r agency-agents/* ~/.claude/agents/

# 现在可以在 Claude Code 会话中激活任何代理：
# "Hey Claude, activate Frontend Developer mode and help me build a React component"

# 或者使用 Multi-Tool Integration
./scripts/convert.sh
./scripts/install.sh

# 或针对特定工具
./scripts/install.sh --tool cursor
./scripts/install.sh --tool copilot
./scripts/install.sh --tool aider
./scripts/install.sh --tool windsurf
```

### 代码示例

在Claude Code中使用：

```bash
# 启动Claude Code
claude

# 然后输入：
"请激活 Senior Developer agent 来帮助我重构这个项目"

# 你将看到一个专业的前端开发对话，包含：
# - 需求分析
# - 方案设计
# - 逐步实施计划
# - TDD 风格的实现
```

### 评价

这是目前**最实用的 AI 代理框架之一**。它的优势在于：

✅ **真实性能**: 不是花哨的提示词堆砌，而是真正可用的工作流  
✅ **开箱即用**: 复制即可使用，无需复杂配置  
✅ **持续更新**: 社区贡献新代理，不断扩展能力边界  
✅ **跨工具兼容**: 不局限于单一 IDE 或平台  

**适合人群**: 希望用 AI 提升工作效率的开发者、产品经理和技术领导者。如果你曾经让 AI 写代码但效果不理想，这套专门设计的代理可能会给你惊喜。

---

## 🐟 2. MiroFish - 群体智能预测引擎

**仓库**: [`666ghj/MiroFish`](https://github.com/666ghj/MiroFish)  
**星标数**: 13,931 (今日 +4,469)  
**编程语言**: Python  
**许可证**: Apache-2.0

### 核心功能

MiroFish（微舆）是一款基于**多智能体技术**的新一代 AI 预测引擎。它通过提取现实世界的种子信息（如突发新闻、政策草案、金融信号），自动构建高保真的平行数字世界。在这个空间内，成千上万个具有独立人格、长期记忆与行为逻辑的智能体进行自由交互与社会演化。

你可以透过「上帝视角」动态注入变量，精准推演未来走向——**让未来在数字沙盘中预演，助决策在百战模拟后胜出**。

### 设计理念

MiroFish 致力于打造映射现实的群体智能镜像，通过捕捉个体互动引发的群体涌现，突破传统预测的局限：

- **于宏观**: 决策者的预演实验室，让政策与公关在零风险中试错
- **于微观**: 个人用户的创意沙盘，推演小说结局、探索脑洞、趣味仿真

> "让每一个如果都能看见结果，让预测万物成为可能。"

### 工作原理

1. **图谱构建**: 现实种子提取 & 个体与群体记忆注入 & GraphRAG 构建
2. **环境搭建**: 实体关系抽取 & 人设生成 & 环境配置 Agent 注入仿真参数
3. **开始模拟**: 双平台并行模拟 & 自动解析预测需求 & 动态更新时序记忆
4. **报告生成**: ReportAgent 拥有丰富的工具集与模拟后环境进行深度交互
5. **深度互动**: 与模拟世界中的任意一位进行对话 & 与 ReportAgent 进行对话

### 适用场景

| 场景类型 | 具体用途 |
|----------|----------|
| **金融方向** | 市场趋势预测、投资策略回测、风险评估 |
| **时政要闻** | 政策影响推演、舆情发展预测、国际关系模拟 |
| **创意写作** | 《红楼梦》结局预测、小说剧情分支推演 |
| **热点舆情** | 企业危机管理、品牌声誉预测 |

### 安装方式

#### 本地部署

```bash
# 克隆仓库
git clone https://github.com/666ghj/MiroFish.git
cd MiroFish

# 检查依赖
node -v         # Node.js 18+
python --version  # Python ≥3.11, ≤3.12
uv --version      # 最新版 uv

# 复制示例配置文件
cp .env.example .env

# 编辑 .env 文件，填入必要的 API 密钥
export LLM_API_KEY=your_api_key
export ZEP_API_KEY=your_zep_api_key

# 一键安装所有依赖
npm run setup:all

# 启动服务
npm run dev

# 访问地址
# 前端：http://localhost:3000
# 后端 API: http://localhost:5001
```

#### Docker 部署

```bash
# 配置环境变量
cp .env.example .env

# 拉取镜像并启动
docker compose up -d

# 默认会读取根目录下的 .env，并映射端口 3000（前端）/5001（后端）
```

### 代码示例

```python
# 简单的 Python SDK 调用示例
from mirofish import PredictionEngine

# 初始化引擎
engine = PredictionEngine(
    llm_api_key="your_api_key",
    zep_api_key="your_zep_api_key"
)

# 定义预测任务
task = {
    "seed_material": "武大舆情事件的新闻稿和相关数据",
    "prediction_goal": "预测该事件在未来一周内的舆论走向",
    "simulation_rounds": 50
}

# 执行预测
result = engine.predict(task)

# 获取预测报告和交互式世界
print(result.report)
engine.interact_with_world(result.world_id)
```

### 在线演示

访问在线 Demo 环境体验：[mirofish-live-demo](https://666ghj.github.io/mirofish-demo/)

### 评价

这是一个**极具创新性且雄心勃勃的项目**：

🌟 **亮点**:
- 创新性地应用群体智能理论于预测领域
- 提供严肃预测（金融、时政）和趣味仿真（小说结局）两种模式
- 基于成熟的开源项目 OASIS（CAMEL-AI 团队）构建
- 得到盛大集团的战略支持和孵化

⚠️ **注意事项**:
- API 消耗较大，建议先从小规模模拟（<40 轮）尝试
- 需要配置多个外部 API 密钥
- 对硬件有一定要求

**适合人群**: 研究者、数据科学家、投资决策者、内容创作者。如果你需要探索"如果...会怎样"的场景，MiroFish 提供了前所未有的能力。

---

## 🧠 3. Hermes-Agent - 与你共同成长的 AI 代理

**仓库**: [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent)  
**星标数**: 3,655 (今日 +776)  
**编程语言**: Python  
**许可证**: MIT

### 核心功能

Hermes Agent 是 Nous Research 开发的**自改进 AI 代理**，它是唯一一个拥有**内置学习循环**的代理——它能够从经验中创造技能、在使用过程中自我改进、主动巩固知识、搜索过去的对话，并在跨会话中建立对你更深的理解。

你可以在 $5 的 VPS、GPU 集群或几乎空闲时成本近乎为零的无服务器基础设施上运行它。它**不绑定在你的电脑上**——你可以在 Telegram 中与它交谈，而它在云端 VM 上工作。

### 核心特性

| 特性 | 描述 |
|------|------|
| **终端界面** | 完整的 TUI，支持多行编辑、斜杠命令自动补全、对话历史、中断重定向、流式工具输出 |
| **无处不在** | Telegram、Discord、Slack、WhatsApp、Signal、CLI——所有这些都来自单个网关进程 |
| **闭环保学习环** | 代理策划的记忆、周期性提醒、复杂任务后的自主技能创建、FTS5 会话搜索、用户建模 |
| **定时自动化** | 内置 Cron 调度器，可向任何平台递送日报、夜间备份、每周审计等 |
| **并发子代理** | 为并行工作流派发隔离的子代理，通过 RPC 调用工具的 Python 脚本 |
| **随处运行** | 六个终端后端——本地、Docker、SSH、Daytona、Singularity 和 Modal |
| **研究就绪** | 批量轨迹生成、Atropos RL 环境、用于训练下一代工具调用模型的轨迹压缩 |

### 模型兼容性

使用你喜欢的任何模型——Nous Portal、OpenRouter（200+ 模型）、z.ai/GLM、Kimi/Moonshot、MiniMax、OpenAI 或你自己的 endpoint。**只需一条命令切换，无需代码修改，无锁定**。

### 安装方式

```bash
# 一键安装（适用于 Linux、macOS 和 WSL2）
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# Windows 用户需要先安装 WSL2，然后运行上述命令

# 重新加载 shell
source ~/.bashrc  # 或 source ~/.zshrc

# 配置你的 LLM 提供商
hermes setup

# 开始聊天！
hermes
```

### CLI 命令参考

```bash
# 常用命令
hermes          # 交互式 CLI——开始对话
hermes model    # 切换提供商或模型
hermes setup    # 重新运行设置向导
hermes gateway  # 启动消息网关（Telegram、Discord 等）
hermes update   # 更新到最新版本
hermes doctor   # 诊断任何问题
```

### 代码示例

```bash
# 开始在对话中
$ hermes

Welcome to Hermes Agent! Type /help for available commands.

> Help me write a Python script that fetches weather data from an API.

[Hermes will now:]
1. Think through the requirements
2. Choose appropriate tools
3. Write the code with proper error handling
4. Execute it if needed
5. Explain the results

---

# 使用斜杠命令
/help           # 显示所有可用命令
/list-sessions  # 查看当前会话
/skills         # 查看和管理技能
/cron add       # 添加定时任务
```

### 文档资源

完整文档位于：[hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/)

涵盖主题包括：
- 快速入门指南
- CLI 使用说明
- 配置选项
- 消息网关集成
- 安全实践
- 40+ 工具和工具集
- 技能系统
- MCP 集成
- Cron 调度

### 评价

这是**目前最成熟、最全面的独立 AI 代理框架之一**：

🌟 **优势**:
- **真正的自学习能力**: 能够从零经验中学习并创造新技能
- **灵活的平台集成**: 支持几乎所有主流消息平台
- **模型无关性**: 可以与任何 LLM 配合使用
- **研究友好**: 为 RLHF 和工具调用模型训练提供支持
- **低成本部署**: 可在廉价 VPS 甚至免费层级上运行

⚠️ **学习曲线**: 虽然安装简单，但要充分利用所有功能需要一定时间学习

**适合人群**: 高级开发者、研究人员、需要自主代理进行复杂任务的团队。如果你是技术极客，Hermes Agent 会让你爱不释手。

---

## 🔍 4. Promptfoo - LLM 应用的测试和红队工具

**仓库**: [`promptfoo/promptfoo`](https://github.com/promptfoo/promptfoo)  
**星标数**: 11,880 (今日 +632)  
**编程语言**: TypeScript  
**许可证**: MIT

### 核心功能

Promptfoo 是用于**评估和对抗测试 LLM 应用**的 CLI 和库。告别 trial-and-error 方法——开始构建安全、可靠的 AI 应用。

### 主要特性

- **自动化评估**: 测试你的提示词和模型表现
- **红队测试**: 对抗测试和漏洞扫描，增强 LLM 应用安全性
- **模型对比**: 左右对比（OpenAI、Anthropic、Azure、Bedrock、Ollama 等）
- **CI/CD 集成**: 自动化检查嵌入工作流
- **代码扫描**: 审核 PR 中的 LLM 相关安全和合规问题
- **团队协作**: 与团队成员共享结果

### 工作原理

Promptfoo 让你以声明式配置文件的形式定义评估，它会自动：

1. 运行提示词变体
2. 比较不同模型的表现
3. 计算评分指标
4. 生成可视化报告

### 安装方式

```bash
# npm 全局安装
npm install -g promptfoo

# 或使用 Homebrew（macOS/Linux）
brew install promptfoo

# 或使用 pip（Python 环境）
pip install promptfoo

# 或不安装，直接用 npx
npx promptfoo@latest <command>
```

### 快速开始

```bash
# 初始化示例项目
promptfoo init --example getting-started

# 设置 API 密钥（大多数 LLM 提供商需要）
export OPENAI_API_KEY=sk-abc123
export ANTHROPIC_API_KEY=sk-ant-xxx

# 进入示例目录并运行评估
cd getting-started
promptfoo eval
promptfoo view  # 查看交互式结果仪表板
```

### 配置文件示例

```yaml
# promptfooconfig.yaml
description: "我的第一个评估"

prompts:
  - "Write a poem about {{topic}}"

providers:
  - id: openai:gpt-4o
  - id: anthropic:claude-3-opus

tests:
  - vars:
      topic: "春天的花园"
    assert:
      - type: contains-substring
        value: "花朵"

      - type: javascript
        value: 0.5  # 最低长度阈值
        timeoutMs: 5000

evaluation:
  maxConcurrency: 10
  providers: 2
```

### 红队测试

```bash
# 启动红队测试
promptfoo redteam generate --help

# 运行红队扫描
promptfoo redteam evaluate \
  --config my-app-config.yaml \
  --outputs ./outputs.json
```

Promptfoo 可以识别常见安全问题，如：
- 提示词注入攻击
- 敏感数据泄露
- 越狱尝试
- 不当内容生成

### CI/CD 集成

```yaml
# .github/workflows/promptfoo.yml
name: Promptfoo Eval

on: [push, pull_request]

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Promptfoo Evals
        run: |
          npm install -g promptfoo
          promptfoo eval --config promptfooconfig.yaml
      
      - name: Upload Results
        uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: promptfoo_results.json
```

### 代码示例

```javascript
// Node.js SDK 使用
const promptfoo = require('promptfoo');

const results = await promptfoo.evaluate({
  prompts: ['Hello {{name}}'],
  provider: 'openai:gpt-4',
  tests: [
    {
      vars: { name: 'World' },
      assert: [{ type: 'contains', value: 'Hello' }]
    }
  ]
});

console.log(results);
```

### 评价

Promptfoo 正在成为**LLM 应用的行业标准测试工具**：

🌟 **优点**:
- **开发者友好**: 简单易用的 CLI 和清晰的配置语法
- **隐私优先**: 评估在本地完全运行，提示词不会离开你的机器
- **灵活性强**: 支持任何 LLM API 或编程语言
- **经过验证**: 在生产环境中服务超过 1000 万用户的应用
- **数据驱动**: 基于指标做决策，而非凭直觉

⚠️ **考虑事项**:
- 大型评估可能需要较长时间
- 某些高级功能需要理解 LLM 评估方法论

**适合人群**: 所有开发 LLM 应用的工程师、QA 团队、DevOps 专业人员。如果你的应用使用了大语言模型，Promptfoo 几乎是必备工具。

---

## 🦌 5. DeerFlow - ByteDance 的 SuperAgent 框架

**仓库**: [`bytedance/deer-flow`](https://github.com/bytedance/deer-flow)  
**星标数**: 28,458 (今日 +1,443)  
**编程语言**: Python  
**许可证**: Apache-2.0

### 核心功能

DeerFlow（Deep Exploration and Efficient Research Flow）是一个开源的**SuperAgent 编排框架**，借助沙箱、记忆、工具、技能和子代理的能力，它可以处理从几分钟到几小时不等各种级别的复杂任务。

**重要更新**: 2026 年 2 月 28 日，DeerFlow 2.0 发布——这是完全重写版本，与原 v1 无任何代码共享。活跃开发已移至 2.0。

### 核心特性

| 特性 | 描述 |
|------|------|
| **Skills & Tools** | 可扩展的工具和技能系统 |
| **Claude Code 集成** | 深度集成主流编码助手 |
| **子代理系统** | 派发的隔离子代理处理并行工作流 |
| **沙箱 & 文件系统** | 安全的代码执行环境 |
| **上下文工程** | 项目管理代码库上下文 |
| **长期记忆** | 跨会话持久化状态 |

### 额外集成

DeerFlow 现已集成字节独立研发的智能搜索和爬虫工具集——**InfoQuest**。

### 安装方式

#### Docker（推荐）

```bash
# 克隆仓库
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow

# 生成本地配置文件
make config

# 编辑配置文件
nano config.yaml
# 至少配置一个模型
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY
    max_tokens: 4096
    temperature: 0.7
```

配置 API 密钥（推荐方式）：

```bash
# 编辑 .env 文件
cp .env.example .env
nano .env

# 添加必要的环境变量
TAVILY_API_KEY=your-tavily-api-key
OPENAI_API_KEY=your-openai-api-key
INFOQUEST_API_KEY=your-infoquest-api-key
```

启动服务：

```bash
# 初始化 Docker 镜像（首次运行）
make docker-init

# 启动服务（自动检测 sandbox 模式）
make docker-start

# 访问 http://localhost:2026
```

#### 本地开发

```bash
# 检查前置条件
make check  # 验证 Node.js 22+, pnpm, uv, nginx

# 安装依赖
make install  # 安装后端 + 前端依赖

# （可选）预拉取沙箱镜像
make setup-sandbox

# 启动服务
make dev

# 访问 http://localhost:2026
```

### IM 通道集成

DeerFlow 支持从 messaging apps 接收任务：

| 通道 | 传输 | 难度 |
|------|------|------|
| Telegram | Bot API（长轮询） | 简单 |
| Slack | Socket Mode | 中等 |
| Feishu / Lark | WebSocket | 中等 |

配置示例 (`config.yaml`):

```yaml
channels:
  langgraph_url: http://localhost:2024
  gateway_url: http://localhost:8001
  
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
    allowed_users: []  # 空 = 允许所有人
    
  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN
    app_token: $SLACK_APP_TOKEN
    allowed_users: []
    
  feishu:
    enabled: true
    app_id: $FEISHU_APP_ID
    app_secret: $FEISHU_APP_SECRET
```

### Docker 沙箱模式

支持多种沙箱执行模式：

- **本地执行**: 直接在主机上运行沙箱代码
- **Docker 执行**: 在隔离的 Docker 容器中运行
- **Kubernetes 执行**: 通过 provisioner 服务在 K8s pods 中运行

### 评价

DeerFlow 是目前**最全面的企业级 SuperAgent 框架**：

🌟 **突出优势**:
- **完全重写**: v2.0 解决了 v1 的所有设计缺陷
- **丰富集成**: 支持多种 IM 平台和消息通道
- **InfoQuest 集成**: 内置强大的智能搜索能力
- **生产就绪**: 为企业场景设计，考虑周全
- **活跃开发**: 刚刚登上 GitHub Trending #1，说明受到广泛欢迎

⚠️ **注意事项**:
- 配置相对复杂，需要多个 API 密钥
- 对系统资源有一定要求
- 最佳实践需要了解其设计哲学

**适合人群**: 企业开发者、AI 应用架构师、需要处理复杂任务自动化的团队。如果你想构建自己的 SuperAgent 系统，DeerFlow 是最好的起点之一。

---

## 💻 6. NanoChat - Karpathy 的 $100 ChatGPT

**仓库**: [`karpathy/nanochat`](https://github.com/karpathy/nanochat)  
**星标数**: 快速增长中  
**编程语言**: Python  
**作者**: Andrej Karpathy

### 核心概念

NanoChat 是 Andrej Karpathy 创建的**最简单实验性 LLM 训练框架**。它专为单 GPU 节点设计，代码简洁易改，涵盖了 LLM 的主要阶段：**分词、预训练、微调、评估、推理和聊天 UI**。

最令人惊叹的是：**你可以仅花费约 $48（约 2 小时的 A100/H100 GPU 节点费用）来训练自己的 GPT-2 级别的语言模型**，然后在熟悉的 ChatGPT 风格 Web UI 中与它对话。相比 2019 年 OpenAI 训练 GPT-2 花费的约 $43,000，这是一个惊人的进步。

### 核心设计理念

1. **极简主义**: 所有代码集中在单一文件中
2. **自动化超参数**: 所有其他超参数（宽度、注意力头数、学习率、训练时长、权重衰减等）都以最优方式自动计算
3. **复杂度旋钮**: 只需调整 `--depth` 参数（GPT transformer 的层数），其他一切自动优化

### 工作原理

```bash
# 在 Lambda 或其他 GPU 云上启动 8XA100/H100 节点
# 运行 speedrun 脚本（整个 pipeline 约 3 小时）
bash runs/speedrun.sh

# 完成后，启动聊天 Web UI
python -m scripts.chat_web

# 在浏览器中访问显示的 URL（如 http://your-gpu-node-ip:8000/）
# 然后就可以像和 ChatGPT 聊天一样和你的模型对话了
```

### 性能追踪

NanoChat 维护着一个"GPT-2 Speedrun"排行榜，记录训练到 GPT-2 级别能力所需的墙钟时间（以 DCLM CORE 分数衡量）。以下是部分榜单：

| 排名 | 时间 | val_bpb | CORE | 描述 | 日期 |
|------|------|---------|------|------|------|
| 0 | 168h | - | 0.2565 | 原始 OpenAI GPT-2 检查点 | 2019 |
| 1 | 3.04h | 0.74833 | 0.2585 | d24 baseline，略过拟合 | Jan 29 |
| 4 | 2.02h | 0.71854 | 0.2571 | 改用 NVIDIA ClimbMix | Mar 4 |
| 5 | 1.80h | 0.71808 | 0.2690 | 自动搜索第一轮 | Mar 9 |

当前里程碑：**仅用不到 2 小时就能达到 GPT-2 级别的能力！**

### 高级用法

```bash
# 训练更小的模型用于快速实验（约 5 分钟）
OMP_NUM_THREADS=1 torchrun --standalone --nproc_per_node=8 \
  -m scripts.base_train -- \
  --depth=12 \
  --run="d12" \
  --model-tag="d12" \
  --core-metric-every=999999 \
  --sample-every=-1 \
  --save-every=-1

# 监控 wandb 指标
# - val_bpb: 验证损失（词汇表大小无关单位）
# - core_: 核心能力度量
```

### 硬件要求

- **推荐**: 8X A100/H100 GPU 节点（80GB VRAM）
- **可行**: 单个 A100/H100（速度降低 8 倍）
- **最小**: Ampere A100（稍慢一些）
- **限制**: 如果 GPU VRAM < 80GB，需要调优 `--device_batch_size` 参数

### 评价

这是**LML 教育的杰作**：

🌟 **教育价值**:
- 展示了 LLM 训练的完整流程，却只有几千行代码
- 任何人都可以用极低的学习成本理解 LLM 训练
- 让"自己动手训练一个 GPT"成为可能

🌟 **工程亮点**:
- 极致优化的性能
- 自动调参的智慧
- 简洁的代码组织

⚠️ **实际考量**:
- 需要 GPU 云实例（虽便宜但仍有成本）
- 生成的模型能力有限（类似幼儿园水平的"孩子"）
- 主要用于学习和实验，而非生产

**适合人群**: 机器学习学生、AI 爱好者、想深入理解 LLM 训练原理的工程师。无论你是否有深度学习背景，NanoChat 都值得花时间尝试。

---

## ⚡ 7. Superpowers - Agentic Skills 框架

**仓库**: [`obra/superpowers`](https://github.com/obra/superpowers)  
**星标数**: 76,432 (今日 +1,387)  
**编程语言**: Shell  
**许可证**: MIT

### 核心功能

Superpowers 是一套**完整的软件开发工作流**，为你的编码代理而建。它基于一系列可组合的"skills"，确保你的代理从一开始就按正确的方式工作。

### 独特之处

当你启动编码代理时，它不会立即跳入写代码，而是：

1. **退一步询问**: 你到底想做什么？
2. **提炼需求**: 从对话中提取详细规格
3. **设计方案**: 展示足够简短的设计供消化
4. **实施计划**: 生成清晰到"热情的初级工程师"都能遵循的计划
5. **子代理开发**: 启动多代理协作，逐个处理工程任务

### 核心 Skills

| Skill | 触发时机 | 功能 |
|-------|----------|------|
| brainstorming | 写代码前 | 苏格拉底式设计细化 |
| using-git-worktrees | 设计批准后 | 创建工作树隔离空间 |
| writing-plans | 设计批准后 | 详细的实施计划 |
| subagent-driven-development | 计划确认后 | 两阶段审核（规范合规→代码质量） |
| test-driven-development | 实施期间 | RED-GREEN-REFACTOR 循环 |
| requesting-code-review | 任务间 | 计划对比审查 |
| finishing-a-development-branch | 任务完成 | 验证测试、合并/PR 决策 |

### 安装方式

根据平台不同，安装方式也不同：

#### Claude 插件市场

```bash
/plugin install superpowers@claude-plugins-official
```

#### Claude Code

```bash
# 注册市场
/plugin marketplace add obra/superpowers-marketplace

# 安装插件
/plugin install superpowers@superpowers-marketplace
```

#### Cursor Agent

```bash
/plugin-add superpowers
```

#### Codex

```bash
# 获取并阅读安装说明
Fetch and follow instructions from:
https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

#### OpenCode

```bash
Fetch and follow instructions from:
https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

### 使用示例

```bash
# 在新会话中开始，输入触发 skill 的内容
"帮我规划这个新功能"

# 系统将自动触发：
# 1. brainstorming skill → 探讨需求
# 2. 生成设计文档
# 3. writing-plans skill → 分解实施步骤
# 4. subagent-driven-development → 并行执行任务

# 全程无需手动干预，代理会自动选择合适的技能
```

### 评价

Superpowers 代表了**AI 辅助编程的新范式**：

🌟 **核心理念**:
- **过程优于猜测**: Systematic over ad-hoc
- **证据优于断言**: Evidence over claims
- **复杂性简化**: Complexity reduction is primary goal

🌟 **实践价值**:
- 显著减少 AI 幻觉和错误
- 提高代码质量和可靠性
- 让 AI 能够长时间自主学习（长达几小时）

⚠️ **学习要点**:
- 需要信任代理的流程
- 初期适应可能需要时间
- 最适合结构化项目

**适合人群**: 追求高质量代码的团队、愿意拥抱 AI 工作流的开发者。如果你喜欢系统化方法胜过即兴发挥，Superpowers 会很对你的胃口。

---

## 📊 8. Generative AI on Google Cloud

**仓库**: [`GoogleCloudPlatform/generative-ai`](https://github.com/GoogleCloudPlatform/generative-ai)  
**星标数**: 15,673 (今日 +534)  
**主要语言**: Jupyter Notebook

### 核心内容

这是 Google Cloud Platform 官方的**生成式 AI 资源和教程集合**，包含面向 Gemini 和 Vertex AI 的笔记本、代码示例、样本应用和学习资源。

### 主要内容区域

| 目录 | 内容 |
|------|------|
| `/gemini` | Gemini 入门笔记本、用例、函数调用、示例应用 |
| `/search` | Vertex AI Search 构建网站和企业搜索 |
| `/rag-grounding` | RAG 和 Grounding 索引 |
| `/vision` | Imagen API 图像生成、编辑、问答 |
| `/audio` | Chirp 语音模型 |
| `/setup-env` | Google Cloud 和 Vertex AI 环境设置 |
| `/RESOURCES.md` | 博客、YouTube 播放列表等资源 |

### 最新资源

Gemini 3.1 Pro 发布后，新增了多个笔记本和演示：

- [Intro to Gemini 3.1 Pro](https://github.com/GoogleCloudPlatform/generative-ai/blob/main/gemini/getting-started/intro_gemini_3_1_pro.ipynb)

### 相关项目

- [Agent Development Kit Samples](https://github.com/google/adk-samples) - 基于 ADK 的即 agents
- [Agent Starter Pack](https://github.com/GoogleCloudPlatform/agent-starter-pack) - 生产就绪的 AI Agent 模板
- [Gemini Cookbook](https://github.com/google-gemini/cookbook/)
- [Vertex AI GenMedia Creative Studio](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio)
- [Generative AI for Marketing using Google Cloud](https://github.com/GoogleCloudPlatform/genai-for-marketing)
- [Generative AI for Developer Productivity](https://github.com/GoogleCloudPlatform/genai-for-developers)

### 评价

对于**希望在 Google Cloud 平台上构建生成式 AI 应用的开发者**来说，这是一个不可或缺的参考资料：

🌟 **优点**:
- 官方维护，质量保证
- 覆盖广泛的主题
- 实时更新的示例
- 与企业级集成

🎯 **适合人群**: GCP 用户、需要企业级 GenAI 方案的团队、学习 Vertex AI 的开发者

---

## 📁 总结与建议

今天的 Trending 榜单呈现出几个明显的趋势：

### 趋势分析

1. **多智能体系统崛起**: Agency Agents、MiroFish、DeerFlow、Hermes-Agent 都强调多智能体协作
2. **AI 测试工具需求增长**: Promptfoo 反映行业对 LLM 质量控制的重视
3. **LLM 训练门槛降低**: NanoChat 证明普通人也能掌握 LLM 训练
4. **AI 编程工作流成熟**: Superpowers 提供了结构化的 AI 协作方法
5. **企业级解决方案增多**: Google Cloud、ByteDance 等大厂都在布局

### 精选推荐

根据我的经验，以下几个最值得尝试：

| 优先级 | 项目 | 推荐理由 |
|--------|------|----------|
| ⭐⭐⭐⭐⭐ | Promptfoo | 如果你开发 LLM 应用，这是必备工具 |
| ⭐⭐⭐⭐⭐ | Agency Agents | 最能立即见效的 AI 效率工具 |
| ⭐⭐⭐⭐ | DeerFlow | 最全面的 SuperAgent 框架 |
| ⭐⭐⭐⭐ | NanoChat | 理解 LLM 训练的绝佳入口 |
| ⭐⭐⭐ | Hermes-Agent | 最灵活的独立 AI 代理 |
| ⭐⭐⭐ | Superpowers | 有纪律的 AI 编程工作流 |

---

*Current time: Wednesday, March 11th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-10 22:00 UTC*
