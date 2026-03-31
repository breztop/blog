---
title: "每日开源速递 - 2026-03-31"
date: 2026-03-31T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-03-31

今天是 2026 年 3 月 31 日，GitHub Trending 榜单上涌现出了一批令人瞩目的开源项目。今天我们将深入探讨 10 个最热门的项目，涵盖 AI 开发工具、语音处理框架、自动化会计系统以及经典的学习资源等广泛领域。

---

## 目录

1. [Claude How To](#1-claude-how-to) - 学习 Claude Code 的最佳指南
2. [VibeVoice](#2-vibvoice) - Microsoft 开源的前沿语音 AI 框架
3. [oh-my-claudecode](#3-oh-my-claudecode) - 多智能体编排工具
4. [claude-code-best-practice](#4-claude-code-best-practice) - Claude Code 最佳实践大全
5. [hermes-agent](#5-hermes-agent) - 自进化的 AI 智能体框架
6. [Superpowers](#6-superpowers) - 智能体技能开发框架
7. [PaddleOCR](#7-paddleocr) - 强大的 OCR 和文档解析工具
8. [TaxHacker](#8-taxhacker) - 自托管 AI 会计应用
9. [ChatDev](#9-chatdev) - 多智能体协作平台
10. [coding-interview-university](#10-coding-interview-university) - 编程面试学习路线

---

## 1. Claude How To

**仓库**: [`luongnv89/claude-howto`](https://github.com/luongnv89/claude-howto)  
**星标数**: 12,717 ⭐ | **forks**: 1,355 🍴  
**更新时间**: 2026 年 3 月 31 日新增 2,390 星标

### 核心功能

Claude How To 是一个视觉化、示例驱动的 Claude Code 学习指南。它不是简单的功能参考文档，而是一个结构化的学习路径，带你从基础概念到高级智能体编排。该项目包含：

- **10 个教程模块**：涵盖所有 Claude Code 功能，从斜杠命令到自定义智能体团队
- **可复制的配置模板**：包括斜杠命令、CLAUDE.md 模板、钩子脚本、MCP 配置、子智能体定义和完整插件包
- **Mermaid 图表**：展示每个功能的内部工作原理，让你理解为什么有效，而不仅仅是如何操作
- **渐进式学习路径**：从初学者到专家只需 11-13 小时
- **自我评估功能**：运行 `/self-assessment` 或 `/lesson-quiz` 钩子直接定位知识盲点

### 适用场景

| 使用场景 | 结合的功能 |
|---------|-----------|
| 自动化代码审查 | 斜杠命令 + 子智能体 + 记忆 + MCP |
| 团队入门 | 记忆 + 斜杠命令 + 插件 |
| CI/CD 自动化 | CLI 参考 + 钩子 + 后台任务 |
| 文档生成 | 技能 + 子智能体 + 插件 |
| 安全审计 | 子智能体 + 技能 + 钩子（只读模式） |
| DevOps 管道 | 插件 + MCP + 钩子 + 后台任务 |

### 安装方式

```bash
# 1. 克隆学习指南
git clone https://github.com/luongnv89/claude-howto.git
cd claude-howto

# 2. 复制你的第一个斜杠命令
mkdir -p /path/to/your-project/.claude/commands
cp 01-slash-commands/optimize.md /path/to/your-project/.claude/commands/

# 3. 在 Claude Code 中尝试输入
# /optimize

# 4. 设置项目记忆
cp 02-memory/project-CLAUDE.md /path/to/your-project/CLAUDE.md

# 5. 安装技能
cp -r 03-skills/code-review ~/.claude/skills/
```

快速设置（1 小时基础版）：
```bash
# 斜杠命令 (15 分钟)
cp 01-slash-commands/*.md .claude/commands/

# 项目记忆 (15 分钟)
cp 02-memory/project-CLAUDE.md ./CLAUDE.md

# 安装技能 (15 分钟)
cp -r 03-skills/code-review ~/.claude/skills/
```

### 代码示例

**基础斜杠命令**（优化分析）：
```markdown
<!-- .claude/commands/optimize.md -->
---
name: Optimize Code
description: Analyze code for optimization opportunities
triggers: ["optimize", "refactor"]
---

## Quick Optimization Analysis

Let's analyze your code for potential optimizations:

1. Review algorithmic complexity
2. Identify redundant operations
3. Suggest memory-efficient alternatives
4. Check for caching opportunities

Run this command to get a structured analysis report.
```

**项目记忆配置**：
```markdown
# CLAUDE.md - Project Standards

## Coding Standards
- Always use TypeScript strict mode
- Write unit tests for all public functions
- Document with JSDoc comments

## Preferred Patterns
- Use compositional design for utilities
- Implement error boundaries in React components
- Follow semantic versioning for API changes

## Context Rules
When working on auth modules:
- Review security implications
- Check token expiration handling
- Validate input sanitization
```

### 评价

⭐️⭐️⭐️⭐️⭐️ 这是一份不可多得的实用学习资源。相比官方文档的功能罗列，这个指南提供了真实世界的模板和渐进式学习路径。无论你是刚开始接触 Claude Code 的初学者，还是想要掌握高级技巧的资深开发者，都能从中获得立即可用的价值。唯一的改进空间是希望能增加更多中文社区贡献的案例。

---

## 2. VibeVoice

**仓库**: [`microsoft/VibeVoice`](https://github.com/microsoft/VibeVoice)  
**星标数**: 32,979 ⭐ | **forks**: 3,715 🍴  
**更新时间**: 2026 年 3 月 31 日新增 3,862 星标

### 核心功能

Microsoft 推出的开源前沿语音 AI 框架，包含文本转语音（TTS）和自动语音识别（ASR）模型。核心创新在于使用持续语音编码器（Acoustic 和 Semantic），以超低帧率 7.5 Hz 高效保持音频保真度，同时显著提升长序列处理的计算效率。

**主要模型：**

| 模型 | 参数量 | 特点 |
|-----|-------|------|
| VibeVoice-ASR-7B | 7B | 单路处理最长 60 分钟音频，结构化转录（谁、何时、什么内容） |
| VibeVoice-TTS-1.5B | 1.5B | 生成长达 90 分钟的对话，支持最多 4 个不同说话人 |
| VibeVoice-Realtime-0.5B | 0.5B | 实时 TTS，约 300 毫秒首字延迟，支持流式文本输入 |

**关键特性：**

- 🌐 **超多元语言支持**：超过 50 种语言原生支持
- ⏱️ **超长音频处理**：60 分钟连续音频单次处理，保持全局一致性
- 👥 **多说话人支持**：单个对话中最多支持 4 个不同说话人
- ✨ **表达性语音**：捕捉对话动态和情感细微差别
- 🔥 **vLLM 推理加速**：支持更快的推理性能

### 适用场景

- **播客制作**：将音频转换为带说话人和时间戳的结构化字幕
- **会议记录**：自动生成带有精确时间标记的会议纪要
- **语音助手**：实时文本转语音交互应用
- **教育内容**：生成多语言教学音频材料
- **无障碍应用**：为视障用户提供语音合成服务

### 安装方式

```bash
# 通过 Hugging Face Transformers 使用
pip install transformers torch

from transformers import pipeline

# 加载 ASR 模型
asr = pipeline("automatic-speech-recognition", 
               model="microsoft/VibeVoice-ASR")

# 处理音频文件
result = asr("audio_file.wav")
print(result["text"])

# 或使用 Realtime 模型进行低延迟推理
from vibevoice_realtime import VibeVoiceRealtime

tts = VibeVoiceRealtime.from_pretrained("microsoft/VibeVoice-Realtime-0.5B")
with tts.stream() as streamer:
    for audio_chunk in streamer.feed_text("Hello world"):
        play_audio(audio_chunk)
```

Docker 部署：
```bash
docker pull ghcr.io/microsoft/vibevoice-asr:latest
docker run -p 8000:8000 -v ./audio:/input ghcr.io/microsoft/vibevoice-asr:latest
```

### 代码示例

**批处理会议录音：**
```python
from vibevoice_asr import VibeVoiceASR

# 初始化模型
model = VibeVoiceASR.from_pretrained("microsoft/VibeVoice-ASR-7B")

# 处理长达 1 小时的会议录音
transcription = model.transcribe(
    "meeting_recording.mp3",
    hotwords=["product", "launch", "Q2"],  # 自定义热词
    language="zh-cn"  # 指定语言
)

# 输出结构化数据
for segment in transcription.segments:
    print(f"[{segment.start:.2f}s - {segment.end:.2f}s]")
    print(f"Speaker: {segment.speaker}")
    print(f"Text: {segment.text}\n")
```

**实时语音助手：**
```python
import sounddevice as sd
from vibevoice_realtime import VibeVoiceRealtime

# 初始化实时 TTS
tts = VibeVoiceRealtime.from_pretrained("microsoft/VibeVoice-Realtime-0.5B")

def voice_assistant():
    streamer = tts.stream()
    
    while True:
        text = get_user_input()  # 获取用户输入
        for audio in streamer.feed_text(text):
            sd.play(audio, tts.sample_rate)
            sd.wait()
        
        break  # 等待下一轮输入

voice_assistant()
```

### 评价

⭐️⭐️⭐️⭐️☆ Microsoft 的开源旗舰作品。VibeVoice 在长音频处理和对话模拟方面展现了卓越的能力，特别是其 60 分钟单次处理能力远超现有方案。多说话人支持和元语言覆盖使其成为播客、教育和企业应用的理想选择。美中不足的是 TTS 部分已移除代码，仅保留 ASR 模型，限制了全栈语音应用的构建能力。建议关注后续是否重新开源完整功能。

---

## 3. oh-my-claudecode

**仓库**: [`Yeachan-Heo/oh-my-claudecode`](https://github.com/Yeahchan-Heo/oh-my-claudecode)  
**星标数**: 18,833 ⭐ | **forks**: 1,384 🍴  
**更新时间**: 2026 年 3 月 31 日新增 1,127 星标

### 核心功能

面向团队的多智能体编排框架，为零配置学习和自然语言接口而生。OMC 让多个智能体协同工作，自动并行分配复杂任务，并通过持久执行确保任务完成。

**核心架构：**

| 表面类型 | 工作进程 | 最佳用途 |
|---------|---------|---------|
| `omc team N:codex` | N 个 Codex CLI 面板 | 代码审查、安全分析、架构设计 |
| `omc team N:gemini` | N 个 Gemini CLI 面板 | UI/UX 设计、文档编写、大上下文任务 |
| `omc team N:claude` | N 个 Claude CLI 面板 | 通用任务的 Claude CLI 执行 |
| `/ccg` | 三模型综合 | 混合后端+UI 任务需要 Codex 和 Gemini |

**执行模式：**

1. **Team（推荐）**：标准流水线（team-plan → team-prd → team-exec → team-verify → team-fix）
2. **omc team（CLI）**：tmux CLI 工作进程，按需生成，任务完成后自动销毁
3. **ccg**：通过 `/ask codex` + `/ask gemini` 后进行 Claude 综合
4. **Autopilot**：自主执行（单一主导智能体）
5. **Ultrawork**：最大并行度（非团队模式）
6. **Ralph**：持久模式，验证修复循环

### 适用场景

- **团队协作开发**：多个智能体协同处理大型项目
- **跨模型任务**：同时利用 Claude、Codex、Gemini 的优势
- **长期维护任务**：确保复杂任务最终完成而非中途放弃
- **成本优化**：智能路由节省 30-50% token 费用
- **实时监控**：HUD 状态栏显示底层执行细节

### 安装方式

**推荐方式（通过 Marketplace）：**
```bash
# 在 Claude Code 中
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
/setup
```

**NPM CLI 方式：**
```bash
npm i -g oh-my-claude-sisyphus@latest
```

启用原生团队支持：
```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 代码示例

**多智能体协作：**
```bash
# 3 个 executor 智能体一起修复所有 TypeScript 错误
/team 3:executor "fix all TypeScript errors"

# 使用 OMC CLI 启动 tmux 工作进程
omc team 2:codex "review auth module for security issues"
omc team status auth-review
omc team shutdown auth-review
```

**混合模型任务：**
```bash
/ccg Review this PR — architecture (Codex) and UI components (Gemini)
```

**自动学习技能：**
```markdown
# .omc/skills/fix-proxy-crash.md
---
name: Fix Proxy Crash
description: aiohttp proxy crashes on ClientDisconnectedError
triggers: ["proxy", "aiohttp", "disconnected"]
source: extracted
---
Wrap handler at server.py:42 in try/except ClientDisconnectedError...
```

查看和管理技能：
```bash
/skill list | add | remove | edit | search
/learner extracts reusable patterns with strict quality gates
```

### 评价

⭐️⭐️⭐️⭐️⭐️ OMC 代表了智能体协作的未来方向。零配置设计和自然语言接口极大地降低了使用门槛，同时强大的团队编排能力又满足了专业需求。跨模型支持和成本优化策略使其成为企业级应用的首选。唯一需要注意的是需要安装 tmux 和相应的 CLI 工具才能发挥全部能力。

---

## 4. claude-code-best-practice

**仓库**: [`shanraisshan/claude-code-best-practice`](https://github.com/shanraisshan/claude-code-best-practice)  
**星标数**: 最新更新同步至 v2.1.87（2026 年 3 月 31 日）

### 核心功能

这是一个全面的 Claude Code 最佳实践合集，涵盖了从基础知识到高级配置的方方面面。项目采用统一的架构模式：研究 → 规划 → 执行 → 审查 → 交付。

**主要组成部分：**

| 功能 | 位置 | 描述 |
|-----|------|------|
| Subagents | `.claude/agents/<name>.md` | 隔离上下文中的自主行为者 |
| Commands | `.claude/commands/<name>.md` | 用户调用的提示模板 |
| Skills | `.claude/skills/<name>/SKILL.md` | 可配置的自动触发能力 |
| Workflows | 特定 YAML | 命令 → 智能体 → 技能的编排 |
| Hooks | `.claude/hooks/` | 事件驱动的用户定义处理器 |
| MCP Servers | `.mcp.json` | Model Context Protocol 连接 |
| Plugins | 可分发包 | 技能、子智能体等的集合 |

### 适用场景

- **团队协作规范**：建立统一的项目标准和智能体行为准则
- **新手入门引导**：逐步学习从基础到高级的所有功能
- **生产环境部署**：遵循经过验证的最佳实践
- **技能库建设**：积累可复用的技能和解决方案

### 安装方式

```bash
# 克隆仓库
git clone https://github.com/shanraisshan/claude-code-best-practice.git
cd claude-code-best-practice

# 查看功能矩阵
cat README.md | grep -A 20 "Feature Table"

# 复制到项目中使用
cp -r best-practice/ .claude/
```

### 代码示例

**指令模板**：
```markdown
# .claude/commands/weather-orchestrator.md
---
name: Weather Update
description: Get current weather with recommendations
---

## Weather Report

Let me check the weather for you:

1. Get current conditions via API
2. Analyze forecast trends
3. Provide activity recommendations

Would you like me to integrate this into your daily briefings?
```

**工作流程编排**：
```yaml
# orchestration-workflow.yaml
study:
  - research_requirements
  - gather_examples
  
plan:
  - define_specification
  - break_down_tasks

execute:
  - parallel_execution
  - checkpoint_creation

review:
  - code_review
  - test_validation

ship:
  - documentation_update
  - deployment_prep
```

### 评价

⭐️⭐️⭐️⭐️⭐️ 这是 Claude Code 生态中最全面的参考资料之一。它不仅列出了所有功能，还提供了实际可用的模板和工作流。对于想要系统化学习和使用 Claude Code 的团队来说，这是不可或缺的资源。建议定期关注更新以跟上平台演进。

---

## 5. hermes-agent

**仓库**: [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent)  
**星标数**: 20,200 ⭐ | **forks**: 2,450 🍴  
**更新时间**: 2026 年 3 月 31 日新增 1,909 星标

### 核心功能

一个由 Nous Research 打造的自进化 AI 智能体框架。它是唯一一个内置学习循环的智能体——它根据经验创建技能，在使用过程中改进技能，主动提醒自身保留知识，搜索自己的过往对话，并建立跨会话的深层用户模型。

**核心能力：**

- 🔄 **闭环学习**：Agent-curated memory 配合周期性的自我提示
- 📅 **定时自动化**：内置 cron 调度器，向任何平台投递结果
- 🤖 **子智能体并发**：派生隔离的子智能体进行并行工作流
- ☁️ **随处运行**：支持本地终端、Docker、SSH、Daytona、Singularity 和 Modal 六个终端后端
- 💬 **跨平台通讯**：Telegram、Discord、Slack、WhatsApp、Signal 和 CLI

**支持的模型提供商：**
- Nous Portal
- OpenRouter（200+ 模型）
- z.ai/GLM
- Kimi/Moonshot
- MiniMax
- OpenAI
- 自有端点

### 适用场景

- **个人助手**：7×24 小时运行的个性化助理
- **团队自动化**：批量数据处理、报告生成、审计任务
- **跨设备协作**：在 Telegram 中发送消息，云端 VM 上执行
- **研究实验**：批量轨迹生成、RL 训练环境
- **历史遗留迁移**：OpenClaw 用户的无缝迁移工具

### 安装方式

```bash
# 一键安装（适用于 Linux、macOS、WSL2）
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 重新加载 shell
source ~/.bashrc  # 或 source ~/.zshrc

# 开始使用
hermes  # 交互式 CLI
hermes model  # 选择 LLM 提供商和模型
hermes tools  # 配置启用的工具
hermes config set  # 设置配置值
hermes gateway  # 启动即时通讯网关
hermes setup  # 运行完整设置向导
```

OpenClaw 迁移：
```bash
hermes claw migrate  # 交互式迁移
hermes claw migrate --dry-run  # 预览将要迁移的内容
hermes claw migrate --preset user-data  # 仅迁移用户数据
```

### 代码示例

**基础对话：**
```bash
# 启动交互式 CLI
hermes

# 切换模型
/model openrouter/nousresearch/hermes-2-pro-mixtral-8x7b

# 检查上下文使用
/compress
/usage --days 7

# 查看可用技能
/skills

# 中断当前工作
Ctrl+C
```

**API 调用示例（Python SDK）：**
```python
from hermes_sdk import HermesAgent

agent = HermesAgent(model="openrouter:gpt-4o")

# 发送邮件摘要任务
task = agent.create_task(
    name="Daily Email Summary",
    schedule="0 9 * * *",  # 每天早上 9 点
    prompt="Summarize today's unread emails by priority"
)

agent.run_async(task)
```

### 评价

⭐️⭐️⭐️⭐️⭐️ Hermes Agent 展示了自进化智能体的未来方向。其独特的学习循环设计使其能够随着时间的推移变得更聪明，而不是每次会话都从零开始。对开源模型的广泛支持和灵活部署选项使其成为技术爱好者的首选。开箱即用的开放云迁移功能也体现了其对用户体验的重视。

---

## 6. Superpowers

**仓库**: [`obra/superpowers`](https://github.com/obra/superpowers)  
**星标数**: 127,000+ ⭐  
**更新时间**: 持续活跃

### 核心功能

Superpowers 是一套完整的软件开发生成 workflow，基于一系列可组合的"技能"和初始指令构建。当你启动编码智能体时，它不会立即开始写代码，而是先询问你到底想做什么。一旦从对话中提炼出规格说明，它会以易于消化的分段形式展示给你。

**核心流程：**

1. **头脑风暴（brainstorming）**：激活于写代码前，通过提问完善粗略想法
2. **Git 工作树（using-git-worktrees）**：设计批准后创建隔离工作区
3. **撰写计划（writing-plans）**：将工作分解为 2-5 分钟的块级任务
4. **子智能体驱动开发（subagent-driven-development）**：为每个工程任务派遣新鲜子智能体
5. **测试驱动开发（test-driven-development）**：强制 RED-GREEN-REFACTOR 循环

### 适用场景

- **新项目启动**：帮助理清需求并制定详细的实现计划
- **复杂功能开发**：将大任务分解为可管理的子任务
- **团队协作**：多个智能体并行处理不同的工程任务
- **质量保障**：严格的测试驱动开发和代码审查流程

### 安装方式

**Claude Code Marketplace：**
```bash
/plugin install superpowers@claude-plugins-official
```

注册自定义 Marketplace：
```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

**Cursor Agent Chat：**
```bash
/add-plugin superpowers
# 或在插件市场中搜索 "superpowers"
```

### 代码示例

**技能触发演示：**
```bash
# 当智能体检测到你在构建新功能时，自动触发 brainstorming
"Help me plan this feature"

# 设计批准后自动进入 writing-plans 阶段
"go ahead with implementation"

# 执行过程中自动启用 TDD
running tasks... (RED-GREEN-REFACTOR loop activated)
```

**技能文件示例：**
```markdown
# skills/writing-plans/SKILL.md
---
name: Writing Plans
trigger: "After design approval"
workflow: plan → task_breakdown → verification
---

## Implementation Plan Template

Every task includes:
1. Exact file paths
2. Complete code snippets
3. Verification steps
4. Expected outcomes

Follow these principles:
- true red/green TDD
- YAGNI (You Aren't Gonna Need It)
- DRY (Don't Repeat Yourself)
```

### 评价

⭐️⭐️⭐️⭐️⭐️ Superpowers 代表了一种全新的智能体辅助开发范式。它将软件工程的最佳实践（如 TDD、代码审查、清晰的设计）内化为智能体的工作方式，使得即使是初级工程师也能产出高质量代码。模块化技能设计允许团队根据实际需求定制工作流。唯一需要注意的是需要投入时间熟悉这套方法论的全貌。

---

## 7. PaddleOCR

**仓库**: [`PaddlePaddle/PaddleOCR`](https://github.com/PaddlePaddle/PaddleOCR)  
**星标数**: 70,000+ ⭐  
**发布时间**: 2026 年 3 月 31 日新增大量关注度

### 核心功能

PaddleOCR 是一个强大的 OCR 和文档解析工具，可将 PDF 和图像转换为结构化的 LLM 就绪数据（JSON/Markdown）。它填补了图像/PDF 与大语言模型之间的空白，支持 100+ 种语言。

**核心组件：**

| 组件 | 特点 |
|-----|------|
| **PaddleOCR-VL-1.5** | 0.9B 轻量级视觉语言模型，OmniDocBench 准确率 94.5% |
| **PP-StructureV3** | 精细坐标信息（表格单元格坐标、文本坐标等） |
| **PP-OCRv5** | 109 种语言支持，精度提升 13% |

**关键特性：**

- 🎯 **SOTA 文档 VLM**：处理扭曲、扫描、屏幕摄影、光照不均、倾斜文档
- 📊 **结构化转换**：精确的坐标信息和层级标题识别
- ⚡ **生产就绪**：商业级精度，资源占用极小
- 🌐 **100+ 语言**：原生支持全球主流语言及方言
- 🔧 **多种硬件支持**：NVIDIA GPU、Intel CPU、昆仑芯片 XPU 等

### 适用场景

- **企业文档数字化**：将纸质档案转换为可检索的数字格式
- **RAG 系统**：为知识库提供高质量的文档解析
- **跨境支付**：处理多语言发票和收据的自动识别
- **学术研究**：文献资料的 OCR 和结构化提取
- **移动应用**：扫码开票、证件识别等功能

### 安装方式

```bash
# 通过 pip 安装
pip install paddleocr>=3.3.0

# 导入并使用
from paddleocr import PaddleOCR, PPStructure

# 初始化 OCR 引擎
ocr = PaddleOCR(use_angle_cls=True, lang="ch")

# 识别图像
result = ocr.ocr("document.jpg", cls=True)

# 结构化解析
structure = PPStructure(table=True, layout=True)
result = structure("complex_document.pdf")
```

Docker 部署：
```bash
docker pull docker.paddlepaddle.org/paddleocr/paddleocr:3.3-latest
docker run -v ./data:/data docker.paddlepaddle.org/paddleocr/paddleocr:3.3-latest
python -u app.py --use_gpu --gpu_id=0
```

### 代码示例

**文档解析完整流程：**
```python
from paddleocr import PaddleOCR, PPStructure

# 初始化
ocr = PaddleOCR(lang="multi")
structure = PPStructure(table=True, layout=True)

# 处理 PDF 文档
pdf_path = "invoice_20260331.pdf"
img_list = convert_pdf_to_images(pdf_path)

results = []
for img in img_list:
    ocr_result = ocr.ocr(img, det=True, rec=True)
    structure_result = structure(img)[0] if isinstance(img, str) else structure(img[0])
    
    results.append({
        "ocr": ocr_result,
        "layout": structure_result.layout,
        "tables": structure_result.tables
    })

# 转换为 Markdown
markdown_content = markdownify(results)
save_to_database(markdown_content)
```

**批量处理多语言文档：**
```python
languages = ["en", "zh", "ja", "ko", "ar", "hi"]
documents = glob.glob("/uploads/documents/*")

for doc in documents:
    detected_lang = detect_language(doc)  # 自动检测语言
    if detected_lang in languages:
        result = ocr.ocr(doc, lang=detected_lang)
        process_and_store(result)
```

### 评价

⭐️⭐️⭐️⭐️⭐️ PaddleOCR 已经是业界的事实标准，特别是在需要处理复杂文档和多语言的场景中。PaddleOCR-VL-1.5 的推出标志着其在文档解析领域的领先地位进一步巩固。94.5% 的 OmniDocBench 准确率已经超越了许多闭源方案。对于需要构建 RAG 系统或文档处理流水线的团队来说，这是必选的工具。

---

## 8. TaxHacker

**仓库**: [`vas3k/TaxHacker`](https://github.com/vas3k/TaxHacker)  
**星标数**: 3,689 ⭐ | **forks**: 594 🍴  
**更新时间**: 2026 年 3 月 31 日新增 325 星标

### 核心功能

一个自托管的 AI 会计应用，专为自由职业者、独立开发者和小型企业设计。上传收据、发票或 PDF 的照片，TaxHacker 会自动识别并提取会计所需的所有重要数据，并将其保存到结构化的 Excel 类数据库中。

**核心功能：**

- 📸 **AI 数据提取**：自动抓取日期、金额、商户、行项目
- 🏷️ **自动分类**：交易根据内容自动归类到相关类别
- 🌍 **货币转换**：基于交易日期的历史汇率自动转换
- 📝 **自定义字段**：创建无限的自定义字段提取额外信息
- 📤 **导出功能**：过滤交易导出为 CSV，包含所有附件

### 适用场景

- **自由职业者记账**：自动化处理杂乱的收据和发票
- **初创企业管理**：低成本的企业级财务管理系统
- **跨境电商**：处理多货币交易和历史汇率转换
- **税务申报准备**：生成符合会计师要求的报表
- **隐私敏感型用户**：完全自托管，数据所有权在自己手中

### 安装方式

**Docker Compose（推荐）：**
```bash
# 下载配置
curl -O https://raw.githubusercontent.com/vas3k/TaxHacker/main/docker-compose.yml

# 启动服务
docker compose up
```

**手动部署：**
```bash
# 克隆仓库
git clone https://github.com/vas3k/TaxHacker.git
cd TaxHacker

# 安装依赖
npm install

# 设置环境变量
cp .env.example .env

# 编辑 .env 配置数据库连接
# DATABASE_URL=postgresql://user@localhost:5432/taxhacker

# 初始化数据库
npx prisma generate && npx prisma migrate dev

# 启动开发服务器
npm run dev
```

### 代码示例

**REST API 调用：**
```python
import requests
from datetime import datetime

API_BASE = "http://localhost:7331/api"

# 上传收据图片
with open("receipt.jpg", "rb") as f:
    response = requests.post(
        f"{API_BASE}/upload",
        files={"file": f},
        headers={"Authorization": f"Bearer {API_KEY}"}
    )
    
document_id = response.json()["id"]

# AI 提取数据
extraction = requests.post(
    f"{API_BASE}/documents/{document_id}/extract",
    json={"provider": "openai", "prompt": "Extract merchant, date, total, items"}
)

# 保存结构化数据
transaction = {
    "date": extraction.json()["date"],
    "merchant": extraction.json()["merchant"],
    "amount": extraction.json()["total"],
    "currency": extraction.json()["currency"],
    "category": extraction.json()["category"],
    "document_id": document_id
}

requests.post(f"{API_BASE}/transactions", json=transaction)
```

**自定义字段配置：**
```json
{
  "custom_fields": [
    {
      "name": "project_code",
      "prompt_template": "Extract project code from invoice header",
      "example_values": ["PRJ-2026-001", "PRJ-2026-002"]
    },
    {
      "name": "department",
      "prompt_template": "Identify billing department",
      "categories": ["Engineering", "Marketing", "Sales"]
    }
  ]
}
```

### 评价

⭐️⭐️⭐️⭐️☆ TaxHacker 完美解决了自由职业者和小型企业面临的记账痛点。AI 驱动的收据识别准确率高，支持自定义字段的设计非常灵活。自建部署选项对注重数据隐私的用户非常有吸引力。目前处于早期开发阶段，功能还在快速迭代中。希望未来能增加更多报表和分析功能。

---

## 9. ChatDev 2.0

**仓库**: [`OpenBMB/ChatDev`](https://github.com/OpenBMB/ChatDev)  
**版本**: v2.0（2026 年 1 月 7 日发布）

### 核心功能

ChatDev 已从专门软件开发的多智能体系统演变为全面的零代码多智能体编排平台。用户可以通过简单配置快速构建和执行自定义多智能体系统，无需编写代码即可定义智能体、工作流和任务。

**ChatDev 2.0 新特性：**

- 🎨 **零代码界面**：可视化画布设计多智能体系统
- 🚀 **拖拽编排**：轻松定义节点参数和上下文流
- 📊 **实时监控**：实时日志、中间产物检查和人工干预
- 🔌 **SDK 支持**：Python SDK 可编程执行工作流

**典型用例：**

| 类别 | 工作流 | 示例 |
|-----|--------|------|
| 数据可视化 | data_visualization_basic.yaml | 创建 4-6 张高质量 PNG 图表 |
| 3D 生成 | blender_3d_builder.yaml | 圣诞树 3D 模型生成 |
| 游戏开发 | GameDev_v1.yaml | 井字棋游戏开发 |

### 适用场景

- **快速原型**：无需专业知识即可搭建多智能体系统
- **数据分析**：自动生成可视化报告和洞察
- **内容创作**：多智能体协作生成文章、视频脚本
- **教育科研**：模拟复杂的社会、经济或政治场景
- **业务自动化**：定制化信息收集和内容发布流水线

### 安装方式

```bash
# 环境要求
# OS: macOS / Linux / WSL / Windows
# Python: 3.12+
# Node.js: 18+
# Package Manager: uv

# 克隆仓库
git clone https://github.com/OpenBMB/ChatDev.git
cd ChatDev

# 同步依赖
uv sync

# 前端依赖
cd frontend && npm install

# 环境变量配置
cp .env.example .env

# 同时启动前后端
make dev

# 访问 Web 控制台 http://localhost:5173
```

**Docker 部署：**
```bash
# 确保有 .env 文件和 API 密钥
docker compose up --build

# 后端：http://localhost:6400
# 前端：http://localhost:5173
```

### 代码示例

**Python SDK 使用：**
```python
from runtime.sdk import run_workflow

# 执行工作流
result = run_workflow(
    yaml_file="yaml_instance/data_analysis.yaml",
    task_prompt="Create 4–6 high-quality PNG charts for my real-estate dataset",
    attachments=["/path/to/housing_data.csv"],
    variables={"API_KEY": "sk-xxxx"}
)

if result.final_message:
    print(f"Output: {result.final_message.text_content()}")
```

**自定义工作流 YAML：**
```yaml
# custom_workflow.yaml
nodes:
  - id: researcher
    type: llm_agent
    prompt: "Research market trends in ${industry} sector"
    providers: [deep-research, web-search]
    
  - id: analyst
    type: llm_agent
    prompt: "Analyze ${research_results} for insights"
    dependencies: [researcher]
    
  - id: reporter
    type: output_node
    template: |
      ## Market Report
      
      Based on analysis:
      ${analysis_summary}
      
      Key findings:
      ${key_insights}
    
    dependencies: [analyst]
```

### 评价

⭐️⭐️⭐️⭐️⭐️ ChatDev 2.0 的零代码设计理念极具前瞻性，真正降低了多智能体系统的构建门槛。可视化的工作流编辑器让非技术人员也能参与智能体编排，扩展了 AI 技术的应用边界。Python SDK 的加入又满足了开发者的灵活性需求。开源社区的活跃度和文档完善度都值得称赞。

---

## 10. coding-interview-university

**仓库**: [`jwasham/coding-interview-university`](https://github.com/jwasham/coding-interview-university)  
**星标数**: 数十万（经典项目持续受欢迎）

### 核心功能

这是一个完整的计算机科学学习计划，旨在帮助你成为软件工程师。最初是作者为自己求职准备的待办清单，后来成长为一份详尽的学习路线图。通过这份学习计划，作者成功被亚马逊聘为软件开发工程师。

**涵盖范围：**

📚 **基础科目**：
- 数据结构：数组、链表、栈、队列、哈希表、树、图
- 算法：排序、搜索、动态规划、递归
- 复杂度分析：Big-O、渐近分析

💻 **实战技能**：
- 编程语言选择与实践
- 算法题目练习（LeetCode 等）
- 系统设计原理

🎯 **面试准备**：
- 简历优化
- 面试流程指导
- 常见问题库

### 适用场景

- **求职准备**：针对大厂技术面试的系统复习
- **CS 补强**：自学计算机科学的非科班人士
- **技能刷新**：职场人士的阶段性能力提升
- **团队培训**：组织内部的技术培训计划

### 使用方法

```bash
# 克隆仓库
git clone https://github.com/jwasham/coding-interview-university.git
cd coding-interview-university

# Fork 并跟踪进度
git remote add upstream https://github.com/jwasham/coding-interview-university.git
git checkout -b my-progress

# 完成后标记进度
[x] Read Introduction
[x] Choose Programming Language
[x] Complete Data Structures section

git commit -am "Completed foundational topics"
git push origin my-progress
```

### 学习建议

根据作者的反馈：

- **每天投入**：8-12 小时全职学习
- **预计时长**：数月（取决于基础）
- **语言选择**：建议使用 C（底层体验）+ Python（快速实现）
- **避免误区**：不必追求 100% 掌握，75% 足以应对面试

### 评价

⭐️⭐️⭐️⭐️⭐️ 这几乎是编程面试领域的圣经级资源。虽然作者强调现代求职者不需要花这么多时间，但其全面性和系统性仍然无可替代。多语言翻译支持让全球学习者受益。对于有明确求职目标的人来说，这是最高效的路径图。

---

## 总结

今天的 GitHub Trending 榜单呈现了 AI 时代的多元发展：

1. **AI 开发工具普及化**：Claude How To、Oh-My-ClaudeCode、Best Practices 等项目显示 AI 编程助手正在从工具演变为生态
2. **语音技术突破**：Microsoft 的 VibeVoice 证明开源模型在专业性上可以媲美甚至超越商业产品
3. **智能体协作成熟**：Superpowers、Hermes-Agent、ChatDev 等展现出多智能体系统的实际应用潜力
4. **垂直领域深耕**：PaddleOCR、TaxHacker 在特定场景达到 SOTA 水平
5. **传统价值回归**：Coding Interview University 这样的经典学习资源依然不可替代

这些项目共同指向一个趋势：**AI 正在从单打独斗转向群体协作，从通用能力走向专业精耕**。对于开发者和创业者而言，现在是拥抱这些变化、构建下一代智能应用的最好时机。

---

*本文档由每日开源速递团队整理，旨在为开发者提供最新的开源项目资讯和技术洞察。欢迎在评论区分享你的使用体验和见解！*

Current time: Wednesday, April 1st, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-31 22:00 UTC