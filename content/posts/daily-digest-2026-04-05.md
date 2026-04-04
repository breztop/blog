---
title: "每日开源速递 - 2026-04-05"
date: 2026-04-05T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

## 今日开源热点速览

今天是 2026年4月5日，GitHub Trending 页面涌现出多个值得关注的开源项目。从 AI Agent 框架到多媒体处理工具，从代码效率提升到安全监控，今天的榜单展现了技术社区的多元创新。以下是 12 个热门项目的深度解读。

---

## 1. MLX-VLM - Mac 端视觉语言模型推理利器

**项目地址**: [Blaizzy/mlx-vlm](https://github.com/Blaizzy/mlx-vlm)

**今日热度**: 316 stars | 累计: 3,562 stars

### 核心功能

MLX-VLM 是专为 Mac 用户设计的视觉语言模型（VLM）推理和微调包，基于 Apple 的 MLX 框架构建。它让开发者能够在本地 Mac 设备上高效运行和微调多模态大模型，支持图像理解、音频处理以及多模态融合任务。

主要特性：
- 支持 OpenAI 兼容的 API 接口，可无缝对接现有应用
- 内置 Gradio Chat UI，一键启动可视化交互界面
- 支持 KV Cache 量化，显著降低内存占用
- 提供 NVIDIA GPU 支持（mxfp8/nvfp4 量化模式）
- 支持音频输入和多模态混合处理

### 适用场景

- **本地 AI 应用开发**: 在 Mac 上构建私有多模态 AI 应用
- **模型研究与实验**: 快速验证新模型架构和微调策略
- **轻量级部署**: 无需云服务，本地运行视觉语言模型
- **内容创作辅助**: 图像描述、音频转文字等创作工具

### 安装方式

```bash
# 使用 pip 安装
pip install mlx-vlm

# 启动 Chat UI
mlx_vlm.chat_ui --model mlx-community/Qwen2-VL-2B-Instruct-4bit

# 启动 API 服务器
mlx_vlm.server --model mlx-community/Qwen2-VL-2B-Instruct-4bit --port 8080
```

### 代码示例

```python
from mlx_vlm import load, generate
from mlx_vlm.prompt_utils import apply_chat_template
from mlx_vlm.utils import load_config

# 加载模型
model_path = "mlx-community/Qwen2-VL-2B-Instruct-4bit"
model, processor = load(model_path)
config = load_config(model_path)

# 准备输入
image = ["http://images.cocodataset.org/val2017/000000039769.jpg"]
prompt = "Describe this image."

# 应用聊天模板
formatted_prompt = apply_chat_template(
    processor, config, prompt, num_images=len(image)
)

# 生成输出
output = generate(model, processor, formatted_prompt, image, verbose=False)
print(output)
```

### 评价

MLX-VLM 是 Apple Silicon 生态中多模态 AI 开发的里程碑项目。它打破了传统依赖云服务的桎梏，让开发者真正实现"本地优先"的 AI 应用构建。对于隐私敏感场景和成本控制需求，这是一个极具价值的选择。项目文档完善，API 设计合理，上手门槛低，非常适合 Mac 用户快速体验多模态 AI。

---

## 2. Onyx - 企业级开源 AI 平台

**项目地址**: [onyx-dot-app/onyx](https://github.com/onyx-dot-app/onyx)

**今日热度**: 1,212 stars | 累计: 24,138 stars

### 核心功能

Onyx 是一个功能丰富的开源 AI 平台，定位为"LLM 应用层"。它提供了完整的 AI 对话、RAG（检索增强生成）、深度研究、代码执行、文档生成等能力，并支持超过 50 种数据源连接器。

核心能力：
- **Agentic RAG**: 混合索引 + AI Agent 的信息检索系统，提供顶级搜索和回答质量
- **Deep Research**: 多步骤研究流程，生成深度报告，在排行榜名列前茅
- **Custom Agents**: 可定制专属 AI Agent，配置独特指令、知识和动作
- **Web Search**: 支持 Serper、Google PSE、Brave、SearXNG 等多种搜索引擎
- **Artifacts**: 生成文档、图表等可下载产物
- **Actions & MCP**: 与外部应用交互，支持灵活的认证选项
- **Voice Mode**: 语音对话支持

### 适用场景

- **企业知识管理**: 构建企业内部知识库和智能问答系统
- **研发团队协作**: 共享对话、Agent 和知识资源
- **深度研究分析**: 复杂主题的多轮调研和报告生成
- **客户服务增强**: AI 辅助的客户支持和信息查询

### 安装方式

```bash
# 一键部署
curl -fsSL https://onyx.app/install_onyx.sh | bash

# 或使用 Docker Compose
git clone https://github.com/onyx-dot-app/onyx.git
cd onyx
docker compose up -d
```

Onyx 提供两种部署模式：
- **Lite Mode**: 轻量级 Chat UI，内存占用低于 1GB
- **完整版**: 包含向量索引、后台任务队列、模型推理服务、Redis 缓存、MinIO 存储

### 评价

Onyx 是当前最全面的开源 AI 平台之一，功能覆盖从基础对话到企业级部署的完整链路。它的 RAG 系统质量出色，深度研究能力在同类型产品中处于领先地位。对于需要构建内部 AI 能力的企业，Onyx 提供了开箱即用的解决方案，避免了从零搭建的复杂性。项目的 MIT 许可证和活跃社区使其成为企业 AI 落地的优选。

---

## 3. Oh My Codex (OMX) - Codex 工作流增强层

**项目地址**: [Yeachan-Heo/oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex)

**今日热度**: 1,803 stars | 累计: 15,528 stars

### 核心功能

OMX 是 OpenAI Codex CLI 的工作流增强层，它不替代 Codex，而是在其之上构建更好的任务路由、工作流程和运行时支持。通过角色关键词和技能系统，让 Codex 更高效地完成复杂开发任务。

核心功能：
- **$deep-interview**: 深度澄清，厘清意图、边界和非目标
- **$ralplan**: 规划审批，生成实现方案并评估权衡
- **$ralph**: 持久完成循环，推动任务到 100% 完成
- **$team**: 协调并行执行，支持多 Agent 协作
- **持久状态管理**: `.omx/` 目录存储计划、日志、记忆和运行状态
- **Tmux 集成**: 支持 Mac/Linux 的持久化团队运行时

### 适用场景

- **复杂项目开发**: 需要多轮澄清和规划的大型任务
- **并行任务处理**: 多 Agent 协同处理可拆分的工作
- **持久化开发会话**: 长时间运行的自动化任务
- **团队协作开发**: 共享计划和状态的开发流程

### 安装方式

```bash
# 安装 Codex 和 OMX
npm install -g @openai/codex oh-my-codex

# 初始化配置
omx setup

# 启动推荐配置
omx --madmax --high
```

### 代码示例（工作流）

```bash
# 深度澄清需求
$deep-interview "clarify the authentication change"

# 生成并审批计划
$ralplan "approve the safest implementation path"

# 持久执行
$ralph "carry the approved plan to completion"

# 或并行执行
$team 3:executor "execute the approved plan in parallel"
```

### 评价

OMX 的设计哲学非常优雅：不替代底层工具，而是在其之上构建更好的工作流。这种"增强层"思路避免了重复造轮子，同时大幅提升了开发效率。`$deep-interview → $ralplan → $ralph/$team` 的流程设计符合软件工程最佳实践，让 AI Agent 从"写代码"进化到"管理项目"。对于已经习惯使用 Codex 的开发者，OMX 是一个值得尝试的升级。

---

## 4. OpenScreen - 免费 Screen Studio 替代品

**项目地址**: [siddharthvaddem/openscreen](https://github.com/siddharthvaddem/openscreen)

**今日热度**: 1,600 stars | 累计: 19,524 stars

### 核心功能

OpenScreen 是 Screen Studio 的开源免费替代品，用于创建精美的产品演示视频。它提供屏幕录制、自动/手动缩放、背景定制、运动模糊、注释添加等核心功能，100% 免费用于个人和商业用途。

主要特性：
- 全屏或特定窗口录制
- 自动和手动缩放，可自定义深度级别
- 麦克风和系统音频捕获
- 自定义缩放位置和持续时间
- 视频裁剪隐藏部分内容
- 墙纸、纯色、渐变或自定义背景
- 运动模糊平滑效果
- 文字、箭头、图像注释
- 视频片段裁剪和速度调整
- 多种分辨率和比例导出

### 适用场景

- **产品演示视频**: 软件功能展示和教程制作
- **营销内容创作**: 应用介绍和推广视频
- **教学培训**: 操作流程演示和讲解
- **团队协作**: 远程工作演示和反馈

### 安装方式

从 [GitHub Releases](https://github.com/siddharthvaddem/openscreen/releases) 下载对应平台的安装包：

```bash
# macOS Gatekeeper 绕过（如果需要）
xattr -rd com.apple.quarantine /Applications/Openscreen.app

# Linux AppImage
chmod +x Openscreen-Linux-*.AppImage
./Openscreen-Linux-*.AppImage
```

### 评价

Screen Studio 的 $29/月订阅对许多用户来说是负担。OpenScreen 提供了核心功能的免费替代，虽然不是完整复刻，但覆盖了大多数人需求。项目的 Electron + React + TypeScript 技术栈选择合理，界面设计现代。作为 beta 状态项目，可能存在 bug，但开源性质意味着用户可以贡献修复。对于预算有限的内容创作者，这是一个值得尝试的选择。

---

## 5. Goose - Block 出品的本地 AI Agent

**项目地址**: [block/goose](https://github.com/block/goose)

**核心功能**

Goose 是 Block（原 Square）推出的本地 AI Agent，定位为"超越代码建议"的开发助手。它能自主完成从原型构建、代码编写执行、调试失败到工作流编排的完整工程任务。

主要特性：
- 本地运行，完全自主的 AI Agent
- 支持任何 LLM，多模型配置优化性能和成本
- MCP 服务器无缝集成
- 提供 Desktop App 和 CLI 两种使用方式
- 自动化复杂开发任务从始至终

### 适用场景

- **端到端项目开发**: 从想法到实现的完整流程自动化
- **代码重构和维护**: 批量修改和优化
- **调试和修复**: 自动定位和解决问题
- **工作流编排**: 复杂任务的多步骤协调

### 安装方式

```bash
# 通过 Homebrew 安装 (macOS/Linux)
brew install goose

# 或查看详细安装指南
# https://block.github.io/goose/docs/getting-started/installation
```

### 评价

Goose 代表了 AI Agent 从"建议工具"到"执行者"的进化。Block 作为金融科技公司，选择开源这一工具体现了对开发者生态的投资。项目支持多模型、MCP 集成、本地运行的设计思路符合当前 AI 开发趋势。文档结构完善，有快速入门、教程、故障排查等完整指南。对于追求"动手不动脑"的开发者，Goose 提供了有趣的可能性。

---

## 6. Microsoft Agent Framework - 企业级多 Agent 框架

**项目地址**: [microsoft/agent-framework](https://github.com/microsoft/agent-framework)

**今日热度**: 66 stars | 累计: 8,678 stars

### 核心功能

Microsoft Agent Framework 是微软推出的多语言 Agent 开发框架，同时支持 Python 和 .NET。它提供从简单聊天 Agent 到复杂多 Agent 工作流的完整构建能力，采用图结构编排，支持流式处理、检查点、人机交互和时间旅行。

核心特性：
- **图结构工作流**: Agent 和确定性函数通过数据流连接
- **DevUI**: 交互式开发调试界面
- **AF Labs**: 实验性功能包（基准测试、强化学习）
- **OpenTelemetry**: 内置分布式追踪和监控
- **多 Provider 支持**: Azure OpenAI、Microsoft Foundry、Anthropic 等
- **中间件系统**: 灵活的请求/响应处理管道
- **A2A 协议**: Agent-to-Agent 通信支持

### 适用场景

- **企业级 AI 应用**: 需要可靠性和可观测性的生产系统
- **多 Agent 系统**: 复杂协作和任务编排
- **工作流自动化**: 业务流程的 AI 增强处理
- **Python/.NET 双栈**: 跨语言团队协作

### 安装方式

```bash
# Python
pip install agent-framework

# .NET
dotnet add package Microsoft.Agents.AI
```

### 代码示例

```python
import asyncio
from agent_framework import Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential

async def main():
    agent = Agent(
        client=FoundryChatClient(credential=AzureCliCredential()),
        name="HaikuBot",
        instructions="You are an upbeat assistant that writes beautifully.",
    )
    print(await agent.run("Write a haiku about Microsoft Agent Framework."))

asyncio.run(main())
```

### 评价

微软进军 Agent Framework 领域是意料之中的战略布局。这个框架的 Python/.NET 双栈支持使其在企业环境中有独特优势。图结构工作流设计借鉴了 LangGraph 等成功经验，而 DevUI 和 OpenTelemetry 集成体现了生产级考量。对于已经在 Microsoft 生态的企业，这是构建 AI Agent 系统的自然选择。迁移指南（从 Semantic Kernel、AutoGen）降低了切换成本。

---

## 7. Sherlock - 社交媒体账户追踪工具

**项目地址**: [sherlock-project/sherlock](https://github.com/sherlock-project/sherlock)

### 核心功能

Sherlock 是一个跨平台社交媒体账户追踪工具，能在 400+ 个社交网络上搜索特定用户名的账户存在。主要用于 OSINT（开源情报）调查、数字取证和网络安全审计。

主要特性：
- 支持 400+ 社交网络平台
- Tor 网络支持，增强隐私保护
- CSV/XLSX 输出格式
- 代理配置支持
- 可自定义站点列表
- NSFW 站点可选检测

### 适用场景

- **OSINT 调查**: 开源情报收集和分析
- **数字取证**: 账户追踪和证据收集
- **安全审计**: 发现潜在账户暴露
- **品牌保护**: 监控品牌名在社交平台的使用

### 安装方式

```bash
# pipx 安装
pipx install sherlock-project

# Docker 运行
docker run -it --rm sherlock/sherlock

# Fedora
dnf install sherlock-project
```

### 代码示例

```bash
# 搜单个用户
sherlock user123

# 搜多个用户
sherlock user1 user2 user3

# 使用 Tor
sherlock --tor user123

# 输出为 CSV
sherlock --csv user123
```

### 评价

Sherlock 是安全社区的经典工具，已有多年历史。400+ 站点的覆盖使其成为社交媒体账户追踪的标准选择。工具设计简洁，输出格式灵活，支持多种隐私保护机制。需要注意的是，这类工具的使用应在合法合规范围内，尊重用户隐私和平台政策。对于安全研究人员和取证专家，Sherlock 是必备工具之一。

---

## 8. LightRAG - 高效检索增强生成框架

**项目地址**: [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG)

### 核心功能

LightRAG 是香港大学数据科学研究院推出的高效 RAG 框架，发表于 EMNLP 2025。它采用双层级检索策略和知识图谱增强，在查询效率和答案质量上表现出色。

核心特性：
- **双层级检索**: 低层级精确匹配 + 高层级概念理解
- **知识图谱集成**: 实体关系提取增强检索
- **多存储后端**: Neo4j、MongoDB、PostgreSQL、OpenSearch
- **WebUI**: 可视化文档索引和知识图谱探索
- **多模态支持**: 通过 RAG-Anything 集成
- **引用功能**: 源码追踪和文档溯源
- **Reranker**: 混合查询性能提升

### 适用场景

- **企业知识库**: 大规模文档检索和问答
- **研究辅助**: 学术文献理解和知识整合
- **多模态 RAG**: 文本、图像、表格、公式混合处理
- **生产部署**: Docker Compose 一键部署

### 安装方式

```bash
# PyPI 安装
pip install lightrag-hku

# 服务端安装（含 API）
uv tool install "lightrag-hku[api]"

# Docker Compose
git clone https://github.com/HKUDS/LightRAG.git
cd LightRAG
cp env.example .env
docker compose up
```

### 评价

LightRAG 在学术研究转化为实用工具方面做得出色。双层级检索和知识图谱的结合解决了传统 RAG 的碎片化问题。WebUI 和 Ollama 兼容接口使其易于集成到现有系统。项目文档详细，有完整的环境配置指南和部署教程。对于追求高质量 RAG 的应用开发者，LightRAG 是值得深入研究的选择。

---

## 9. Qwen Code - 通义千问终端 AI Agent

**项目地址**: [QwenLM/qwen-code](https://github.com/QwenLM/qwen-code)

**今日热度**: 91 stars | 累计: 21,780 stars

### 核心功能

Qwen Code 是阿里通义千问团队推出的终端 AI Agent，专为代码理解和开发任务优化。它提供免费 OAuth 登录（每日 1000 次请求），支持多种 LLM Provider。

核心特性：
- **多协议支持**: OpenAI/Anthropic/Gemini 兼容 API
- **免费层级**: Qwen OAuth 登录，每日 1000 次免费请求
- **Skills/SubAgents**: 完整 Agent 工作流支持
- **IDE 集成**: VS Code、Zed、JetBrains 支持
- **TypeScript SDK**: 程序化调用
- **Thinking 模式**: 支持思考增强模型

### 适用场景

- **终端开发**: 命令行环境下的代码辅助
- **代码理解**: 大型项目结构分析
- **自动化任务**: CI/CD 集成和脚本执行
- **多模型切换**: 不同任务使用不同模型

### 安装方式

```bash
# 一键安装脚本
bash -c "$(curl -fsSL https://qwen-code-assets.oss-cn-hangzhou.aliyuncs.com/installation/install-qwen.sh)"

# npm 安装
npm install -g @qwen-code/qwen-code@latest

# Homebrew
brew install qwen-code

# 启动
qwen
```

### 评价

Qwen Code 代表了中国 AI 公司在开源生态的重要布局。免费 OAuth 层级降低了使用门槛，多协议支持提供了灵活性。项目文档完善，有详细的配置指南和多种使用模式说明。与 OpenAI Codex、Claude Code 的竞争格局中，Qwen Code 以开放性和免费额度作为差异化优势。对于中国开发者，这是本地化支持和合规性更好的选择。

---

## 10. Repomix - 代码仓库打包工具

**项目地址**: [yamadashy/repomix](https://github.com/yamadashy/repomix)

**今日热度**: 63 stars | 累计: 23,031 stars

### 核心功能

Repomix 是将整个代码仓库打包成单一 AI友好文件的工具。它自动处理 .gitignore、计算 Token 数量、检测敏感信息，让开发者轻松将代码库提供给 LLM 分析。

核心特性：
- **AI 优化格式**: XML、Markdown、纯文本输出
- **Token 计数**: 每文件和仓库总 Token 数
- **Git 感知**: 自动遵守 .gitignore、.ignore、.repomixignore
- **安全检测**: Secretlint 集成，防止敏感信息泄露
- **代码压缩**: Tree-sitter 提取关键元素
- **远程仓库**: 直接处理 GitHub 仓库
- **浏览器扩展**: Chrome/Firefox一键访问

### 适用场景

- **代码重构**: 将整个项目提供给 AI 分析重构方案
- **代码审查**: 批量代码理解和审查
- **文档生成**: 基于代码生成说明文档
- **迁移规划**: 项目迁移前的整体分析

### 安装方式

```bash
# npx 直接使用（无需安装）
npx repomix@latest

# npm 全局安装
npm install -g repomix

# Homebrew
brew install repomix
```

### 代码示例

```bash
# 打包当前目录
repomix

# 打包特定目录
repomix path/to/directory

# 打包远程仓库
repomix --remote yamadashy/repomix

# 压缩模式
repomix --compress

# 包含 git logs
repomix --include-logs
```

### 评价

Repomix 解决了 LLM 代码分析的痛点：如何将完整项目上下文高效传递给模型。其安全检测功能尤其重要，防止 API Key 等敏感信息意外泄露。项目获得了 JSNation Open Source Awards 2025 提名，体现了社区认可。浏览器扩展和在线服务 repomix.com 降低了使用门槛。对于需要 AI 辅助代码理解的开发者，这是必备工具。

---

## 11. Oh My OpenAgent (OmO) - 顶级 Agent Harness

**项目地址**: [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)

**今日热度**: 468 stars | 累计: 48,174 stars

### 核心功能

Oh My OpenAgent（原名 Oh My Opencode）是一个强大的 Agent Harness，被社区称为"最佳 Agent Harness"。它整合了 LSP、AST-Grep、Tmux、MCP 等工具，提供完整的 AI 开发体验。

核心特性：
- **ultrawork (ulw)**: 一词激活所有 Agent，任务不完成不停止
- **IntentGate**: 分析真实用户意图，避免误解
- **Hash-Anchored Edit**: LINE#ID 内容哈希验证，零陈旧行错误
- **LSP + AST-Grep**: IDE 级精度重写和诊断
- **Background Agents**: 5+ 专家并行执行
- **Ralph Loop**: 自引用循环，100% 完成保证
- **Todo Enforcer**: Agent 闲置时系统自动拉回
- **多模型路由**: visual-engineering、deep、quick、ultrabrain 分类自动选模型

### 适用场景

- **大规模重构**: 复杂项目的自动化重构
- **并行开发**: 多任务同时处理
- **持续执行**: 不中断的长时任务
- **Claude Code 用户**: 完全兼容现有配置

### 安装方式

```bash
# 让 Agent 自动安装
curl -s https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md

# 或手动安装
npm install -g oh-my-opencode
```

### 评价

项目的 slogan "Anthropic blocked OpenCode because of us" 体现了其竞争力。ultrawork 的设计理念——"一词激活，不停直到完成"——解决了 Agent 工具常见的中途放弃问题。多模型路由策略避免了手动切换的麻烦。社区反馈积极，有用户表示"一天清理了 8000 个 eslint warnings"、"取消了 Cursor 订阅"。对于追求效率的开发者，这是一个值得关注的项目。

---

## 12. ChangeDetection.io - 网站变化监控工具

**项目地址**: [dgtlmoon/changedetection.io](https://github.com/dgtlmoon/changedetection.io)

### 核心功能

ChangeDetection.io 是最佳的网站变化监控工具，支持价格追踪、内容更新检测、库存监控等多种场景。通过 Docker 部署，支持 80+ 种通知渠道。

核心特性：
- **多通知渠道**: Discord、Email、Slack、Telegram、Webhook 等
- **浏览器步骤**: 自动登录、点击按钮、填写表单
- **Visual Selector**: 可视化选择监控元素
- **价格监控**: 产品价格变化和库存追踪
- **PDF 监控**: 文本变化、文件大小、校验和
- **JSON API**: 监控 API 响应变化
- **条件触发**: 价格阈值、关键词匹配
- **调度器**: 业务时段检查控制

### 适用场景

- **价格追踪**: 电商产品降价和库存恢复提醒
- **内容监控**: 新闻、公告、政策更新
- **安全合规**: 网站篡改监控、PCI合规增强
- **求职跟踪**: 职位发布和更新提醒

### 安装方式

```bash
# Docker Compose
git clone https://github.com/dgtlmoon/changedetection.io.git
cd changedetection.io
docker compose up -d

# Docker 单容器
docker run -d --restart always -p "127.0.0.1:5000:5000" \
  -v datastore-volume:/datastore \
  --name changedetection.io dgtlmoon/changedetection.io

# Python pip
pip3 install changedetection.io
changedetection.io -d /path/to/data -p 5000
```

### 评价

ChangeDetection.io 在网站监控领域是标杆项目。其 Playwright 集成的浏览器步骤功能让复杂登录场景变得简单。价格监控的自动提取和条件触发设计实用。对于需要持续监控多个信息源的用户，这个工具的价值远超 $8.99/月的 SaaS 服务费。开源版本支持自部署，完全掌控数据和隐私。

---

## 今日总结

2026年4月5日的 GitHub Trending 榜单展现了几个重要趋势：

1. **AI Agent 竞争白热化**: Microsoft Agent Framework、Goose、Oh My Codex、Oh My OpenAgent、Qwen Code 等项目同台竞技，Agent 工具从"建议者"进化为"执行者"

2. **多模态 AI 本地化**: MLX-VLM 让 Mac 用户无需云服务即可运行视觉语言模型，隐私和成本控制成为关键需求

3. **工作流增强层兴起**: OMX 等项目不替代底层工具，而是在其之上构建更好的流程管理

4. **内容创作工具开源化**: OpenScreen 等项目提供商业工具的免费替代，降低创作门槛

5. **企业级框架成熟**: Onyx、Microsoft Agent Framework 提供完整的企业部署支持

**推荐关注优先级**：
- **开发者**: MLX-VLM、Repomix、Oh My OpenAgent
- **企业用户**: Onyx、Microsoft Agent Framework
- **安全研究**: Sherlock、ChangeDetection.io
- **内容创作者**: OpenScreen

---

Current time: Sunday, April 5th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-04-04 22:00 UTC