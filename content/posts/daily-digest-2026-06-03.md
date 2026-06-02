---
title: "每日开源速递 - 2026-06-03"
date: 2026-06-03T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-06-03

> 每个伟大的开发者背后，都有一个 starred 列表比 TODO 列表还长的 GitHub 账号。

今天的 GitHub Trending 页面可谓精彩纷呈——从 AI 上下文压缩到多语言语音合成，从 OSINT 调查图谱到生产级 RAG 课程，覆盖了当前技术圈最火热的方向。以下是 10 个值得深度关注的项目解读。

---

## 1. Headroom — AI Agent 的上下文压缩层

**仓库：** [chopratejas/headroom](https://github.com/chopratejas/headroom)

### 核心功能

Headroom 是一个面向 AI Agent 的上下文压缩层（Context Compression Layer），能在工具输出、日志文件、RAG 分块和对话历史到达 LLM 之前进行压缩，实现 **60-95% 的 token 节省**，同时保持答案质量不变。

它提供四种使用模式：
- **Library**：`compress(messages)` 直接嵌入 Python/TypeScript 应用
- **Proxy**：`headroom proxy --port 8787`，零代码改动适配任何语言
- **Agent Wrap**：`headroom wrap claude|codex|cursor` 一键包装主流编码 Agent
- **MCP Server**：通过 `headroom_compress`、`headroom_retrieve`、`headroom_stats` 工具接入任意 MCP 客户端

### 技术亮点

Headroom 内部使用了 6 种压缩算法：
- **SmartCrusher**：针对 JSON 数据的通用压缩
- **CodeCompressor**：基于 AST 的代码压缩（支持 Python/JS/Go/Rust/Java/C++）
- **Kompress-base**：在 HuggingFace 上训练的自研文本压缩模型
- **CacheAligner**：稳定前缀以命中 Anthropic/OpenAI 的 KV Cache
- **CCR（可逆压缩）**：原始内容存储在本地，LLM 可按需检索

### 安装方式

```bash
# Python
pip install "headroom-ai[all]"

# Node / TypeScript
npm install headroom-ai

# Docker
docker pull ghcr.io/chopratejas/headroom:latest
```

### 代码示例

```python
from headroom import compress

# 压缩消息列表
compressed = compress(messages, model="claude-sonnet-4-20250514")
# compressed 包含压缩后的 prompt 和检索工具
```

```bash
# 包装 Claude Code
headroom wrap claude --memory --code-graph

# 启动代理
headroom proxy --port 8787
```

### 适用场景

- 日常使用 AI 编码 Agent，想在不改代码的前提下降低成本
- 跨多个 Agent（Claude Code、Codex、Cursor）需要共享记忆
- 需要可逆压缩——原始内容始终可通过 CCR 检索

### 评价

Headroom 解决的是 AI Agent 时代最痛的成本问题。随着上下文窗口越来越长、工具调用越来越频繁，token 消耗呈指数级增长。Headroom 的差异化在于**可逆压缩**——不是简单丢弃信息，而是让 LLM 在需要时主动检索原始内容。在 GSM8K、TruthfulQA 等基准测试上，压缩后的准确率几乎无损失。对于重度 Agent 用户来说，这是一个 ROI 极高的工具。

---

## 2. MarkItDown — 微软出品的万能文档转 Markdown 工具

**仓库：** [microsoft/markitdown](https://github.com/microsoft/markitdown)

### 核心功能

MarkItDown 是微软开源的轻量级 Python 工具，用于将各类文件转换为 Markdown 格式，专为 LLM 和文本分析管道设计。它的设计理念很简单：**Markdown 是 LLM 的母语**——主流大模型都原生理解 Markdown，且 token 效率极高。

支持格式覆盖极广：
- PDF、PowerPoint、Word、Excel
- 图片（EXIF 元数据 + OCR）
- 音频（EXIF 元数据 + 语音转录）
- HTML、CSV、JSON、XML
- ZIP 文件（自动遍历内容）
- YouTube URL、EPub

### 安装与使用

```bash
# 安装（全功能）
pip install 'markitdown[all]'

# 或按需安装
pip install 'markitdown[pdf, docx, pptx]'

# 命令行使用
markitdown path-to-file.pdf > document.md
markitdown path-to-file.pdf -o document.md

# 管道
cat path-to-file.pdf | markitdown
```

### 代码示例

```python
from markitdown import MarkItDown
from openai import OpenAI

# 基础用法
md = MarkItDown(enable_plugins=False)
result = md.convert("test.xlsx")
print(result.text_content)

# 使用 LLM 进行图片描述
client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o")
result = md.convert("example.jpg")
print(result.text_content)

# Azure Content Understanding（高级结构化提取）
md = MarkItDown(cu_endpoint="<endpoint>")
result = md.convert("report.pdf")
print(result.markdown)
```

### 插件生态

MarkItDown 支持第三方插件扩展。搜索 GitHub 上的 `#markitdown-plugin` 标签即可找到可用插件。例如 `markitdown-ocr` 插件可为 PDF/DOCX/PPTX 添加 OCR 能力，通过 LLM Vision 提取嵌入图片中的文字。

### 适用场景

- 构建 RAG 系统的文档预处理管道
- 将企业知识库（Office 文档、PDF）批量转换为 LLM 可消化的格式
- 数据分析管道的文档解析环节

### 评价

MarkItDown 是一个"简单到不像微软出品"的工具。它不做花哨的事情，但在文档转换这个细分领域做到了极致的覆盖面和易用性。与 textract 相比，它更关注**保留文档结构**（标题、列表、表格、链接），这对于 LLM 理解文档语义至关重要。如果你在做 RAG，这个工具大概率能省掉你不少胶水代码。

---

## 3. ECC — AI Agent 性能优化系统

**仓库：** [affaan-m/ECC](https://github.com/affaan-m/ECC)

### 核心功能

ECC（Agent Harness Performance Optimization System）不是一个简单的配置包，而是一个完整的 Agent 性能优化系统：**技能（Skills）、直觉（Instincts）、记忆优化（Memory）、持续学习（Continuous Learning）、安全扫描（Security Scanning）**。

它支持主流 AI Agent 框架：Claude Code、Codex、Cursor、OpenCode、Gemini、Zed、GitHub Copilot 等，覆盖 **12+ 语言生态系统**，拥有 63 个 Agent、249 个 Skill、79 个遗留命令兼容。

### 核心能力

| 能力 | 说明 |
|------|------|
| Token 优化 | 模型选择、系统提示精简、后台进程管理 |
| 记忆持久化 | Hook 自动在会话间保存/加载上下文 |
| 持续学习 | 自动从会话中提取模式，转化为可复用技能 |
| 验证循环 | 检查点评估 vs 持续评估，pass@k 指标 |
| 并行化 | Git worktree、cascade 方法、弹性扩展 |
| 子 Agent 编排 | 解决上下文问题，迭代检索模式 |

### 安装方式

```bash
# 推荐方式：Claude Code 插件安装
/plugin marketplace add https://github.com/affaan-m/ECC
/plugin install ecc@ecc

# 或手动安装
git clone https://github.com/affaan-m/ECC.git
cd ECC
npm install
./install.sh --profile core --target claude

# npm 方式
npx ecc-install --profile minimal --target claude
```

### 代码示例

```bash
# 咨询适合的组件
npx ecc consult "security reviews" --target claude

# 查看安装状态
node scripts/ecc.js list-installed

# 诊断和修复
node scripts/ecc.js doctor
node scripts/ecc.js repair

# 生成状态快照
ecc status --markdown --write status.md

# 多 Agent 编排
/pm2 start
/multi-plan "refactor auth module"
/multi-execute
```

### 适用场景

- 重度使用 AI 编码 Agent 的开发者，希望系统化优化工作流
- 需要跨多个 Agent 框架保持一致的行为和记忆
- 团队级别的 AI Agent 标准化和治理

### 评价

ECC 是一个野心极大的项目——它试图成为 AI Agent 时代的"操作系统"。从 182K+ star 和 170+ 贡献者来看，它确实击中了一个痛点：当你在多个 Agent 之间切换时，上下文丢失、行为不一致、成本失控是普遍问题。ECC 的持续学习机制（自动从失败中提取教训并写入技能文件）尤其有意思。但它的复杂度也不低——安装路径繁多，新手容易踩坑。建议从 `minimal` profile 开始，按需添加组件。

---

## 4. Scrapling — 自适应 Web 爬虫框架

**仓库：** [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling)

### 核心功能

Scrapling 是一个自适应 Web 爬虫框架，从单个请求到大规模爬取一站式解决。它的核心卖点是**自适应解析器**——当网站结构变化时，它能自动重新定位你的元素。

主要特性：
- 🕷️ Scrapy 风格的 Spider API，支持异步解析回调
- ⚡ 并发爬取，可配置并发限制和域名节流
- 🔄 多会话支持：HTTP 和隐身浏览器在同一个 Spider 中混用
- 💾 断点续爬：Ctrl+C 优雅暂停，重启后恢复
- 📡 流式模式：实时获取爬取结果
- 🛡️ 自动反封锁检测和重试
- 🤖 内置 Cloudflare Turnstile 绕过

### 安装方式

```bash
pip install scrapling
```

### 代码示例

**基础请求：**
```python
from scrapling.fetchers import Fetcher, StealthyFetcher

# 隐身模式抓取
page = StealthyFetcher.fetch('https://example.com', headless=True, network_idle=True)
products = page.css('.product', auto_save=True)  # 自动保存，网站改版时自动适应
```

**完整爬虫：**
```python
from scrapling.spiders import Spider, Response

class QuotesSpider(Spider):
    name = "quotes"
    start_urls = ["https://quotes.toscrape.com/"]
    concurrent_requests = 10

    async def parse(self, response: Response):
        for quote in response.css('.quote'):
            yield {
                "text": quote.css('.text::text').get(),
                "author": quote.css('.author::text').get(),
            }
        next_page = response.css('.next a')
        if next_page:
            yield response.follow(next_page[0].attrib['href'])

result = QuotesSpider().start()
result.items.to_json("quotes.json")
```

**断点续爬：**
```python
# 第一次运行
QuotesSpider(crawldir="./crawl_data").start()
# Ctrl+C 暂停，再次运行自动恢复
```

### 适用场景

- 需要长期监控的爬虫场景（网站改版后自动适应）
- 需要绕过 Cloudflare 等反爬机制的数据采集
- 大规模并发爬取，需要断点续爬和流式处理

### 评价

Scrapling 的自适应解析是真正的差异化能力。传统爬虫在网站改版时直接挂掉，而 Scrapling 通过相似性算法自动重新定位元素——这对长期运行的数据采集项目来说是杀手级功能。加上内置的反爬绕过、断点续爬和 MCP Server（让 AI 帮你写爬虫），它可能是 2026 年最值得关注的 Python 爬虫框架。92% 的测试覆盖率和完整的类型标注也说明工程质量扎实。

---

## 5. Hermes WebUI — 给 Hermes Agent 一个家

**仓库：** [nesquena/hermes-webui](https://github.com/nesquena/hermes-webui)

### 核心功能

Hermes WebUI 是为 Hermes Agent（一个自托管的自主 AI Agent）打造的轻量级 Web 界面。它提供与 CLI 完全对等的体验——终端能做的，浏览器都能做。

设计理念：三栏布局，左侧栏管理会话和导航，中间是聊天区，右侧是工作区文件浏览。所有设置和会话工具都在 Hermes Control Center 中。

### 核心特性

- **流式响应**：SSE 实时推送 token
- **多 Provider 支持**：OpenAI、Anthropic、Google、DeepSeek、OpenRouter 等
- **会话管理**：创建/重命名/搜索/归档/导出会话
- **工作区浏览**：文件树、代码预览、Markdown 渲染、内联编辑
- **语音输入**：Web Speech API 麦克风录音
- **CLI 会话桥接**：CLI 创建的会话自动出现在 WebUI 中
- **工具调用展示**：内联展示工具名称、参数和结果
- **Mermaid 图表渲染**：流程图、时序图、甘特图

### 安装方式

```bash
git clone https://github.com/nesquena/hermes-webui.git hermes-webui
cd hermes-webui
python3 bootstrap.py

# 或使用启动脚本
./start.sh

# 后台守护进程模式
./ctl.sh start
./ctl.sh status
./ctl.sh logs --lines 100
```

### 适用场景

- 已经在使用 Hermes Agent 的开发者，想要一个图形化界面
- 需要通过手机 SSH 隧道远程访问 Agent
- 自托管 AI Agent 方案的选择和评估

### 评价

Hermes WebUI 的定位非常清晰——它不是要做另一个 ChatGPT 界面，而是为 Hermes Agent 这个特定的自托管 Agent 提供完整的 Web 前端。零构建步骤、零框架依赖（纯 Python + vanilla JS）的设计让部署极其简单。与 OpenClaw 的对比很有意思：Hermes 更注重 Python 生态和自改进技能，OpenClaw 更注重 Node.js 生态和社区技能市场。如果你想要一个"永远在线、有记忆、能定时干活"的 AI Agent，Hermes 生态值得深入评估。

---

## 6. Flowsint — OSINT 图谱调查平台

**仓库：** [reconurge/flowsint](https://github.com/reconurge/flowsint)

### 核心功能

Flowsint 是一个开源的 OSINT（开源情报）图谱探索工具，专注于侦察和调查。它通过可视化图谱界面探索实体间的关系，并支持自动化数据丰富（Enricher）。

支持的数据类型和丰富器覆盖：
- **域名**：反向 DNS、DNS 解析、子域名枚举、WHOIS、历史记录
- **IP**：地理位置、ASN 查询
- **ASN/CIDR**：IP 范围枚举
- **社交媒体**：Maigret 用户名搜索
- **组织**：ASN 关联、域名关联
- **加密货币**：交易历史、NFT 查询
- **网站**：爬虫、链接提取、追踪器识别
- **邮箱**：Gravatar、数据泄露查询、关联域名
- **电话**：数据泄露查询

### 安装方式

```bash
git clone https://github.com/reconurge/flowsint.git
cd flowsint

# Docker 部署（推荐）
make prod

# 开发模式
make dev
```

访问 `http://localhost:5173/register` 创建账号，无默认凭据。

### 架构设计

```
flowsint-app (前端)
    ↓
flowsint-api (FastAPI)
    ↓
flowsint-core (编排器、任务、存储)
    ↓
flowsint-enrichers (丰富器和工具)
    ↓
flowsint-types (类型定义)
```

所有数据存储在本地，完全自托管，确保调查隐私。

### 适用场景

- 网络安全分析师进行威胁情报调查
- 记者和 OSINT 调查员追踪实体关系
- 企业内部安全团队进行数字风险分析

### 评价

Flowsint 填补了开源 OSINT 工具链中图谱可视化这一空白。现有的 OSINT 工具大多是命令行或单点查询，而 Flowsint 提供了一个完整的图谱界面，让你直观地看到域名、IP、组织、个人之间的关联。项目还在早期阶段，但模块化架构设计得很好——添加新的数据类型和丰富器只需要在对应模块中添加文件。对于安全研究人员来说，这是一个值得持续关注和贡献的项目。

---

## 7. VoxCPM2 — 无分词器多语言 TTS 系统

**仓库：** [OpenBMB/VoxCPM](https://github.com/OpenBMB/VoxCPM)

### 核心功能

VoxCPM2 是 OpenBMB 推出的 2B 参数 TTS（文本转语音）系统，采用**无分词器（Tokenizer-Free）**的端到端扩散自回归架构，直接生成连续语音表示，绕过离散分词化，实现高度自然和富有表现力的语音合成。

核心亮点：
- 🌍 **30 种语言**支持，无需指定语言标签
- 🎨 **Voice Design**：用自然语言描述创建全新声音（性别、年龄、音色、情感、语速）
- 🎛️ **可控声音克隆**：短参考音频克隆 + 风格引导
- 🎙️ **终极克隆**：参考音频 + 转录文本，忠实再现每个声音细节
- 🔊 **48kHz 高质量音频**：通过 AudioVAE V2 的非对称编码/解码直接输出
- ⚡ **实时流式**：RTX 4090 上 RTF 低至 ~0.3，Nano-vLLM 加速至 ~0.13
- 📜 **Apache-2.0 开源**，可商用

### 安装方式

```bash
pip install voxcpm
# 要求：Python ≥ 3.10 (<3.13), PyTorch ≥ 2.5.0, CUDA ≥ 12.0
```

### 代码示例

```python
from voxcpm import VoxCPM
import soundfile as sf

model = VoxCPM.from_pretrained("openbmb/VoxCPM2", load_denoiser=False)

# 基础合成
wav = model.generate(
    text="VoxCPM2 是当前推荐的发布版本。",
    cfg_value=2.0,
    inference_timesteps=10,
)
sf.write("demo.wav", wav, model.tts_model.sample_rate)

# Voice Design — 用文字描述创建声音
wav = model.generate(
    text="(年轻女性，温柔甜美的声音)你好，欢迎来到 VoxCPM2！",
    cfg_value=2.0,
    inference_timesteps=10,
)

# 声音克隆
wav = model.generate(
    text="这是 VoxCPM2 的声音克隆演示。",
    reference_wav_path="path/to/voice.wav",
)

# 流式合成
import numpy as np
chunks = []
for chunk in model.generate_streaming(text="流式语音合成非常简单！"):
    chunks.append(chunk)
wav = np.concatenate(chunks)
```

### 性能对比

| 模型 | 参数量 | 开源 | 英文 WER↓ | 中文 CER↓ | 相似度↑ |
|------|--------|------|-----------|-----------|---------|
| VoxCPM2 | 2B | ✅ | 1.84% | 0.97% | 79.0% |
| CosyVoice3 | 1.5B | ❌ | 2.02% | 1.16% | 78.1% |
| F5-TTS | 0.3B | ✅ | 2.00% | 1.53% | 76.0% |

### 评价

VoxCPM2 在开源 TTS 领域投下了一枚重磅炸弹。**Voice Design** 功能是真正的范式转变——不需要任何参考音频，仅凭文字描述就能创造出全新的声音，这在语音内容创作领域有巨大的想象空间。加上 30 种语言支持和 Apache-2.0 商用许可，它可能是 2026 年最值得关注的开源 TTS 项目。从基准测试看，它在开源模型中全面领先，甚至在某些指标上超过了闭源模型。OpenBMB 团队（来自清华大学）在开源 AI 领域的贡献令人敬佩。

---

## 8. Machine Learning for Trading — 量化交易机器学习实战

**仓库：** [stefan-jansen/machine-learning-for-trading](https://github.com/stefan-jansen/machine-learning-for-trading)

### 核心功能

这是 Stefan Jansen 所著《Machine Learning for Algorithmic Trading》第二版的配套代码仓库，包含 **150+ 个 Jupyter Notebook**，覆盖 800 页书的内容，系统化展示如何将 ML 应用于算法交易。

全书四大部分 23 章：
1. **从数据到策略**：市场数据、基本面数据、另类数据、特征工程、组合优化
2. **ML 基础**：线性模型、ML4T 工作流、时间序列、贝叶斯 ML、随机森林、Boosting、无监督学习
3. **NLP for Trading**：情感分析、主题建模、词向量
4. **深度学习与强化学习**：CNN、RNN、自编码器、GAN、深度强化学习

### 内容亮点

- 从线性回归到深度强化学习的完整覆盖
- 端到端 ML4T 工作流：数据获取 → 特征工程 → 模型优化 → 策略设计 → 回测
- 多资产类别：美股、国际股票、ETF、日内策略
- 复现顶刊论文：CNN 时间序列图像化、自编码器资产定价、GAN 合成数据
- 定制版 Zipline 回测引擎

### 安装方式

```bash
git clone https://github.com/stefan-jansen/machine-learning-for-trading.git
cd machine-learning-for-trading

# 安装 conda 环境（推荐）
conda env create -f installation/conda_envs/ml4t.yml
conda activate ml4t

# 或使用 Docker
cd installation
docker-compose up
```

### 代码示例

```python
# 第 11 章：随机森林 — 日股多空策略
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# 特征工程
features = engineered_factors.dropna()
target = (returns > 0).astype(int)

# 训练
model = RandomForestClassifier(n_estimators=100, max_depth=5)
model.fit(features, target)

# 预测和回测
predictions = model.predict(test_features)
# 使用 Zipline 进行回测
```

### 适用场景

- 想系统学习 ML 在量化交易中应用的开发者
- 金融工程专业参考
- 量化研究人员的实践指南

### 评价

这是量化交易 + 机器学习领域最全面的开源学习资源，没有之一。150+ 个可执行的 Notebook、800 页的内容深度、复现多篇顶刊论文——Stefan Jansen 用一个仓库构建了一门完整的课程。第二版引入的 ML4T 工作流概念尤其有价值：它强调 ML 不是孤立的模型训练，而是从数据到策略回测的完整链条。不过需要注意，书中的代码基于较老的库版本（TensorFlow 2.2、pandas 1.0），运行前可能需要做一些适配。

---

## 9. Production Agentic RAG Course — 生产级 Agentic RAG 课程

**仓库：** [jamwithai/production-agentic-rag-course](https://github.com/jamwithai/production-agentic-rag-course)

### 核心功能

这是一个以学习者为中心的实战项目课程，教你从零构建一个完整的 AI 研究助手系统——**arXiv Paper Curator**，自动获取学术论文、理解内容、并用高级 RAG 技术回答研究问题。

课程的核心理念：**先掌握关键词搜索基础，再用向量增强做混合检索**——这是专业公司的做法，而不是直接跳到向量搜索。

7 周课程安排：

| 周 | 主题 | 内容 |
|----|------|------|
| 1 | 基础设施 | Docker、FastAPI、PostgreSQL、OpenSearch、Airflow |
| 2 | 数据管道 | arXiv API 集成、PDF 解析、自动化数据摄入 |
| 3 | BM25 关键词搜索 | OpenSearch 索引管理、BM25 算法、查询 DSL |
| 4 | 混合搜索 | 智能分块 + RRF 融合的关键词+语义检索 |
| 5 | 完整 RAG | 本地 LLM 集成、流式响应、Gradio 界面 |
| 6 | 生产监控 | Langfuse 追踪、Redis 缓存、性能优化 |
| 7 | Agentic RAG | LangGraph 工作流、Telegram Bot、自适应检索 |

### 安装方式

```bash
git clone https://github.com/jamwithai/production-agentic-rag-course.git
cd arxiv-paper-curator

# 配置环境
cp .env.example .env
uv sync

# 启动所有服务
docker compose up --build -d

# 验证
curl http://localhost:8000/api/v1/health
```

### 技术栈

- FastAPI + PostgreSQL 16 + OpenSearch 2.19
- Apache Airflow 3.0 + Ollama + Redis + Langfuse
- Jina AI 嵌入 + Gradio 前端
- LangGraph Agent 编排

### 适用场景

- AI/ML 工程师学习生产级 RAG 架构
- 软件工程师构建端到端 AI 应用
- 数据科学家实现生产 AI 系统

### 评价

这是 RAG 领域最好的实战课程之一。它的差异化在于**先搜索后 AI** 的理念——大多数 RAG 教程直接跳到向量搜索，但真正在生产环境中运行的系统都需要坚实的搜索基础。从 BM25 到混合搜索再到 Agentic RAG 的渐进式教学非常合理。第 7 周的 LangGraph + Telegram Bot 集成尤其精彩——它展示了如何让 RAG 系统具备自主决策能力（文档评分、查询重写、领域守卫）。完全免费、本地运行，成本几乎为零。

---

## 10. Supermemory — AI 时代的记忆引擎

**仓库：** [supermemoryai/supermemory](https://github.com/supermemoryai/supermemory)

### 核心功能

Supermemory 是一个面向 AI 的记忆和上下文引擎，在 LongMemEval、LoCoMo、ConvoMem 三大 AI 记忆基准测试中均排名第一。它解决了 AI 领域最基本的问题：**你的 AI 每次对话都会忘记一切**。

核心能力：
- 🧠 **记忆引擎**：从对话中自动提取事实，处理时间变化、矛盾和自动遗忘
- 👤 **用户画像**：自动维护用户上下文——稳定事实 + 近期活动，一次调用 ~50ms
- 🔍 **混合搜索**：RAG + 记忆在单一查询中完成
- 🔌 **连接器**：Google Drive、Gmail、Notion、OneDrive、GitHub 自动同步
- 📄 **多模态提取**：PDF、图片（OCR）、视频（转录）、代码（AST 感知分块）

### 安装方式

```bash
# Python
pip install supermemory

# Node.js
npm install supermemory
```

### 代码示例

```python
from supermemory import Supermemory

client = Supermemory()

# 存储对话内容
client.add(
    content="用户喜欢 TypeScript，偏好函数式编程",
    container_tag="user_123"
)

# 获取用户画像 + 相关记忆（一次调用）
result = client.profile(container_tag="user_123", q="编程风格偏好")
print(result.profile.static)   # → ["喜欢 TypeScript", "偏好函数式"]
print(result.profile.dynamic)  # → ["正在做 API 集成"]
```

```typescript
// TypeScript
import Supermemory from "supermemory";
const client = new Supermemory();

// 混合搜索：知识库 + 用户记忆
const results = await client.search.memories({
  q: "how do I deploy?",
  containerTag: "user_123",
  searchMode: "hybrid",
});
```

### MCP 集成

```json
{
  "mcpServers": {
    "supermemory": {
      "url": "https://mcp.supermemory.ai/mcp"
    }
  }
}
```

### 框架集成

支持 Vercel AI SDK、LangChain、LangGraph、OpenAI Agents SDK、Mastra、Agno、n8n 等主流框架的开箱即用包装器。

### 适用场景

- 为 AI 聊天应用添加持久记忆能力
- 个性化 AI 助手（记住用户偏好、项目上下文）
- 构建 RAG + 记忆混合系统
- 为编码 Agent（Claude Code、Cursor）添加跨会话记忆

### 评价

Supermemory 的核心洞察是：**记忆不是 RAG**。RAG 检索的是文档块——无状态、对所有人结果相同。而记忆提取和跟踪的是用户的事实，理解"我刚搬到上海"取代了"我住在北京"。Supermemory 将两者融合在同一个查询中，这是正确的架构决策。自动遗忘机制也很关键——临时事实（"明天有考试"）在日期过后自动过期，噪音不会变成永久记忆。对于构建个性化 AI 应用的开发者来说，Supermemory 提供了一个开箱即用的解决方案，省去了自建记忆系统的复杂度。

---

## 趋势观察

从今天的 GitHub Trending 可以看到几个清晰的技术方向：

1. **AI Agent 基础设施成熟**：Headroom（上下文压缩）、ECC（Agent 优化）、Supermemory（记忆引擎）、Hermes WebUI（Agent 界面）——Agent 生态正在从"能用"走向"好用"
2. **多语言 AI 能力爆发**：VoxCPM2 支持 30 种语言 TTS，MarkItDown 处理所有文档格式——全球化 AI 工具链在快速成型
3. **RAG 进入深水区**：Production Agentic RAG Course 展示了从 BM25 到 Agentic RAG 的完整路径，不再是简单的向量搜索
4. **安全调查工具民主化**：Flowsint 让专业级 OSINT 调查变得触手可及
5. **Web 爬虫智能化**：Scrapling 的自适应解析和 MCP 集成代表了爬虫框架的新一代进化

---

*Current time: Wednesday, June 3rd, 2026 — 6:00 AM (Asia/Shanghai) / 2026-06-02 22:00 UTC*
