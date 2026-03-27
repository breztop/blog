---
title: "每日开源速递 - 2026-03-27"
date: 2026-03-27T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026年3月27日

GitHub Trending 项目的深度解读与推荐

Current time: Saturday, March 28th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-27 22:00 UTC

<!-- more -->

## 1. last30days-skill

**GitHub:** [mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill)

**Stars:** 12,474 | **Language:** Python

### 核心功能

last30days-skill 是一个专门用于 AI 代理（如 Claude Code、Codex、Gemini CLI）的插件技能，旨在帮助用户快速掌握过去 30 天内任何主题的最新动态和社区讨论。这个工具以 "AI 世界每月 reinvent 一次" 的理念为基础，通过系统性地搜索、分析和综合多个网络源的数据，生成经过验证的、可靠的最新信息报告。

该工具的核心创新在于其**多信号融合的质量分级相关性评分系统**。它不仅仅依赖于文本相似度，还综合考虑了多个维度：

- **多源并行搜索**：同时搜索 Reddit、X（Twitter）、Bluesky、YouTube、TikTok、Instagram、Hacker News、Polymarket prediction markets 等至少 10 个来源
- **阶段化搜索**：采用两阶段补充搜索策略，先初步搜索，再扩展搜索
- **数据质量评分**：对每个搜索结果实施综合评分流程，包括文本相似度（双向 synonym 扩展 + token 重叠）、参与度速度归一化、源权威性加权、跨平台收敛检测（trigram-token Jaccard 相似度）、以及时间衰减
- **Polymarket 优化**：对预测市场数据采用 5 因子加权复合排名：文本相关性（30%）、24 小时交易量（30%）、流动深度（15%）、价格波动速度（15%）、结果竞争力（10%）

特别值得一提的是其**X handle 自动解析**功能，能够将人名、品牌、产品名自动转换为 X（Twitter）句柄（handle），极大地提高了搜索的准确性和召回率。例如搜索 "Dor Brothers" 能够自动解析为 @thedorbrothers，从而发现那些关键字搜索无法触及的高热度内容。

### 适用场景

last30days-skill 非常适合以下场景：

1. **Prompt 技术研究**：快速了解最新有效的提示词技巧，应用于 ChatGPT、Midjourney、Claude、Paper 等各种 AI 工具
2. **趋势追踪**：追踪音乐、文化、新闻、产品推荐、病毒式传播趋势，回答 "人们现在到底在说什么？"
3. **预测市场分析**：查看 Polymarket 等预测市场的实时数据，了解各方对特定事件的概率评估和资金流向
4. **人工智能领域监控**：监控 AI 技术的最新进展，如 Seedance 2.0 访问权限、paper.design 提示词、Nano Banana Pro 技术等
5. **竞争情报**：自动跟踪竞争对手、特定人物、热门主题的最新动态

工具设计上特别强调**实用性和可操作性**，而不是学术性的严谨。v2.9.5 版本引入了**比较模式**（Comparative Mode），用户可以询问 "X vs Y"（例如 "/last30 cursor vs windsurf"），系统会并行进行 3 轮研究并提供对比分析表、优缺点评估和数据驱动的结论。

### 安装方式

#### Claude Code 插件（推荐）

```bash
/plugin marketplace add mvanhorn/last30days-skill
/plugin install last30days@last30days-skill
```

#### Gemini CLI

```bash
gemini extensions install https://github.com/mvanhorn/last30days-skill.git
```

#### 手动安装（Claude Code / Codex）

```bash
# 克隆仓库
git clone https://github.com/mvanhorn/last30days-skill.git ~/.claude/skills/last30days

# 创建配置目录并设置 API 密钥
mkdir -p ~/.config/last30days
cat > ~/.config/last30days/.env << 'EOF'
SCRAPECREATORS_API_KEY=... # Reddit + TikTok + Instagram - scrapecreators.com
OPENAI_API_KEY=sk-...      # 可选 -Codex 登录的旧版 Reddit 回退
AUTH_TOKEN=...             # 推荐用于 X 搜索
CT0=...                    # 推荐用于 X 搜索
XAI_API_KEY=xai-...        # 可选 -X 搜索回退
BSKY_HANDLE=you.bsky.social       # 可选 -Bluesky 搜索
BSKY_APP_PASSWORD=xxxx-xxxx-xxxx  # 可选 -bsky.app/settings/app-passwords
EOF
chmod 600 ~/.config/last30days/.env
```

#### X 搜索认证

X 搜索推荐使用显式的环境变量认证。步骤：

1. 登录 x.com，打开浏览器开发者工具
2. 复制 `auth_token` 和 `ct0` cookie
3. 保存为 `AUTH_TOKEN` 和 `CT0` 到 `~/.config/last30days/.env`

#### Codex CLI

```bash
git clone https://github.com/mvanhorn/last30days-skill.git ~/.agents/skills/last30days
```

### 代码示例

last30days-skill 主要通过命令行调用，以下是一些典型用法：

```bash
# 基本搜索 - 自动分析过去 30 天的最新动态
last30 query "AI video tools"

# 使用快速模式（牺牲深度换取速度）
last30 query "AI video tools" --quick

# 限制搜索天数
last30 query "AI video tools" --days=7

# 比较模式 - 分析两个选项
last30 query "cursor vs windsurf"
```

**推荐的 .env 配置文件示例：**

```bash
# 一个 API 密钥覆盖多个来源
SCRAPECREATORS_API_KEY=your-scrapecreators-key
# 提供 x.com 搜索以获取最佳结果
AUTH_TOKEN=copy-from-x-com-cookies
CT0=copy-from-x-com-cookies
# 可选的 Bluesky 搜索配置
BSKY_HANDLE=you.bsky.social
BSKY_APP_PASSWORD=app-password-from-settings
```

#### Open Variant（用于 always-on bots）

专为 Open Claw 和类似 always-on AI 环境设计的变体：

```bash
# 启用 open variant
cp variants/open/SKILL.md ~/.claude/skills/last30days/SKILL.md

# 添加监控主题到 watchlist
last30 watch my biggest competitor every week
last30 watch Peter Steinberger every 30 days
last30 watch AI video tools monthly

# 手动运行所有 watchlist
watch run-all

# 手动运行单个主题
watch run-one
```

watchlist 数据存储在 SQLite 数据库中，可以与 cron job 配合实现定期自动更新。

### 评价

**优势：**

1. **多源融合能力强大**：同时搜索 10+ 个不同类型的来源，并采用 sophisticated 的评分算法，确保信息质量和多样性
2. **人性化功能设计**：Bluesky/AT Protocol 支持、Comparative Mode、自动 handle 解析、自动保存到 ~/Documents/等都是非常实用的功能
3. **灵活的配置选项**：支持项目特定的 .env 配置文件，方便不同项目使用不同的 API 密钥
4. **配置验证机制**：v2.9.5 引入的 SessionStart config check 可以在 session 启动时自动验证配置
5. **快速模式**：在需要速度而非深度时，可以使用 --quick 模式

**需要注意的：**

1. **运行时间较长**：v2.5 版本的全面搜索可能需要 2-8 分钟，具体取决于主题的冷门程度
2. **需要多个 API 密钥**：要获得最佳体验，需要配置多个不同的 API 密钥（ScrapeCreators、X、xAI 等）
3. **技术门槛**：手动安装需要一定的 terminal 技巧，特别是 X 搜索认证部分
4. **资源消耗**：并行搜索多个来源需要一定的网络和计算资源

**总结：** last30days-skill 是一个功能强大、设计精良的 AI 调研工具。它不是简单的"搜索并返回结果"，而是提供了完整的"研究工作流"：搜索、评分、去重、综合、引用。对于任何需要快速掌握最新信息的场景，这是一个非常有价值的投资。

---

## 2. Deep-Live-Cam

**GitHub:** [hacksider/Deep-Live-Cam](https://github.com/hacksider/Deep-Live-Cam)

**Stars:** 4,572+ | **Language:** Python

### 核心功能

Deep-Live-Cam 是一个革命性的实时人脸交换和视频深度伪造工具， claimed 可以通过"单击"和"单张图像"实现人脸交换。这是该工具 2.1 版本的主要特性，经过重大改进后变得更容易使用和更强大。

**核心能力包括：**

- **实时人脸交换**：使用摄像头进行实时人脸交换，无需预处理视频
- **单张图像驱动**：只需要一张目标人物的图片即可生成人脸交换效果
- **实时视频处理**：支持从摄像头或预录视频进行实时深度伪造
- **多目标支持**：支持在同一视频中同时使用不同的人脸交换多个主题
- **实时秀场模式**：专门优化的直播表演模式
- **面部细节保留**：Mouth Mask 功能保留原始 mouth 运动，实现更自然的效果

**关键的技术特性：**

- **GFPGANv1.4**：用于面部修复和增强
- **inswapper_128_fp16.onnx**：ONNX 格式的深度学习模型，优化了交换效果
- **支持多种执行提供者**：CUDA（NVIDIA GPU）、CoreML（Apple Silicon）、DirectML（Windows）、OpenVINO（Intel）

### 适用场景

Deep-Live-Cam 的适用场景非常广泛：

1. **内容创作**：创作者可以将自己或角色的脸"放到"视频中，创造有趣的内容
2. **电影和娱乐**：观众可以在观看电影时实时将角色换成自己喜欢的演员
3. **直播和表演**：Live Show 模式支持实时直播表演，创造独特体验
4. **社交媒体**：创建病毒式传播的 meme 和有趣内容
5. **测试和教育**：演示深度学习技术和面部交换原理

**免责声明：** 工具内置了保护机制，防止处理不当内容（裸露、血腥、敏感材料、战争 footage 等）。开发者强调负责任的使用，并建议在使用真实人物人脸时获得同意，并在分享时明确标注为深度伪造内容。

### 安装方式

#### 快速启动版本（Windows/Mac Silicon/CPU）

**推荐非技术人员或没有时间手动安装的用户**。预构建版本性能最佳，支持 NVIDIA/AMD GPU、CPU 和 Mac Silicon，并提供优先支持。

链接：https://deeplivecam.net/index.php/quickstart

#### 手动安装（需要技术技巧）

** prerequisites：**

- Python 3.11（推荐）
- pip
- git
- ffmpeg
- Visual Studio 2022 Runtimes（Windows）

**安装步骤：**

```bash
# 克隆仓库
git clone https://github.com/hacksider/Deep-Live-Cam.git
cd Deep-Live-Cam

# 下载模型
# 1. GFPGANv1.4.onnx
# 2. inswapper_128_fp16.onnx
# 将这些文件放在 "models" 文件夹中

# 创建虚拟环境（推荐）
python -m venv venv

# Windows
venv\Scripts\activate
pip install -r requirements.txt

# Linux
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**macOS 特别设置（Apple Silicon M1/M2/M3）：**

```bash
# 安装 Python 3.11（版本很重要）
brew install python@3.11

# 安装 tkinter 包
brew install python-tk@3.10

# 创建虚拟环境
python3.11 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

#### GPU 加速

**CUDA（NVIDIA GPU）：**

```bash
# 安装 CUDA Toolkit 12.8.0
# 安装 cuDNN v8.9.7 for CUDA 12.x
# 安装依赖
pip install -U torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
pip uninstall onnxruntime onnxruntime-gpu
pip install onnxruntime-gpu==1.21.0

# 运行
python run.py --execution-provider cuda
```

**CoreML（Apple Silicon）：**

```bash
# 安装依赖
pip uninstall onnxruntime onnxruntime-silicon
pip install onnxruntime-silicon==1.13.1

# 运行（重要：指定 Python 3.10）
python3.10 run.py --execution-provider coreml
```

### 代码示例

```bash
# 从任何路径启动
python run.py

# 选择源人脸图像
# 选择摄像头或目标视频
# 点击 "Start" 或 "Live"

# GPU 加速版本
python run.py --execution-provider cuda  # NVIDIA
python run.py --execution-provider coreml  # Apple Silicon
python run.py --execution-provider directml  # Windows
python run.py --execution-provider openvino  # Intel
```

**图像/视频模式：**

1. 执行 `python run.py`
2. 选择源人脸图像
3. 选择目标图像/视频
4. 点击 "Start"
5. 输出保存在以目标视频命名的目录中

**摄像头模式：**

1. 执行 `python run.py`
2. 选择源人脸图像
3. 点击 "Live"
4. 等待开始

### 评价

**优势：**

1. **易用性**：2.1 版本大大简化了使用流程，3 步点击即可完成深度伪造
2. **实时性能**：支持实时人脸交换，而非预处理视频
3. **多平台支持**：支持 Windows、macOS、Linux，以及多种 GPU 类型
4. **功能丰富**：Mouth Mask、Faces Mapping、Movie Mode、Live Show 等多种模式
5. **安全性**：内置内容过滤机制，防止不当内容

**注意事项：**

1. **技术门槛**：手动安装需要一定的 terminal 技能和深度学习基础
2. **资源需求**：GPU 版本需要 NVIDIA GPU（CUDA）或 Apple Silicon（CoreML）
3. **道德问题**：工具本身非常强大，但也带来了深度伪造伦理问题
4. **法律风险**：使用他人人脸需要获得同意，否则可能涉及法律问题

**总结：** Deep-Live-Cam 是一个功能强大且用户友好的实时人脸交换工具。它将深度伪造技术带入寻常百姓家，为内容创作者提供了新的表达方式。然而，它的强大能力也带来了责任，用户需要道德地使用这项技术。

---

## 3. AI-Scientist-v2

**GitHub:** [SakanaAI/AI-Scientist-v2](https://github.com/SakanaAI/AI-Scientist-v2)

**Stars:** 2,818 | **Language:** Python

### 核心功能

AI-Scientist-v2 是一个通用的端到端 agent 系统，能够自主完成从假设生成、实验运行、数据分析到科学论文撰写的全过程。这是该系统的第二版本，相比 v1 移除了对人类编写的模板的依赖，泛化能力更强，适用于各种机器学习领域，并采用渐进式 agent 树搜索策略。

**核心功能模块：**

1. **研究想法生成（Ideation）**：使用 LLM 基于高层次主题描述生成潜在研究想法，通过工具（如 Semantic Scholar）检查新颖性
2. **实验执行（Experimentation）**：使用 agentic tree search 执行实验，可以并行探索多个路径
3. **数据分析（Analysis）**：分析实验结果，绘制图表，计算统计数据
4. **论文撰写（Writing）**：生成科学论文草稿，包括引言、方法、结果、讨论等部分
5. **多模型支持**：支持 OpenAI、Gemini、Anthropic Bedrock 等多种 LLMs

**关键技术特点：**

- **Progressive Agentic Tree Search**：采用渐进式 agent 树搜索策略，由实验管理 agent 引导
- **无模板设计**：v2 版本不再依赖人类编写的模板，完全由 agent 自主生成
- **ML 域泛化**：设计为在各种机器学习子领域都能工作
- **多 LLM 支持**：支持多种后端模型，包括 GPT、Gemini、Claude（通过 Bedrock）

### 适用场景

AI-Scientist-v2 适用于以下场景：

1. **自动化研究**：研究人员可以快速验证研究想法并生成初步的实验结果
2. **教育和学习**：学生可以学习科学研究的完整流程
3. **想法生成**：为研究人员提供新的研究方向和实验设计
4. **论文草稿生成**：快速生成论文的初始版本
5. **跨领域研究**：适用于机器学习领域的各种子领域

**注意！** v2 版本不一定比 v1 生成更好的论文，特别是当有强大的起始模板时。v1 遵循定义良好的模板，成功率达到高，而 v2 采取更广泛的探索性方法，成功率较低。v1 最适合有明确目标和坚实基础的任务，而 v2 专为开放式的科学探索设计。

### 安装方式

**系统要求：** Linux + NVIDIA GPU + CUDA + PyTorch

**安装步骤：**

```bash
# 创建新的 conda 环境
conda create -n ai_scientist python=3.11
conda activate ai_scientist

# 安装 PyTorch with CUDA 支持
conda install pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia

# 安装 PDF 和 LaTeX 工具
conda install anaconda::poppler
conda install conda-forge::chktex

# 安装 Python 依赖
pip install -r requirements.txt
```

安装通常不超过一小时。

### 代码示例

#### 生成研究想法

在运行完整的 AI Scientist-v2 实验流水线之前，首先使用 `ai_scientist/perform_ideation_temp_free.py` 脚本生成潜在研究想法。

**1. 准备主题描述：**

创建 Markdown 文件（如 `my_research_topic.md`），包含 Title、Keywords、TL;DR、Abstract 等部分。

```markdown
Title: "EfficientTransformer: Rethinking Attention Mechanisms for Long-Sequence Processing"
Keywords: transformers, efficiency, long sequences, compression
TL;DR: We propose a novel attention compression technique that reduces memory usage by 60% while maintaining 95% of original performance.
Abstract: Long sequence processing remains a challenge for transformer-based models due to quadratic complexity of self-attention. We introduce EfficientTransformer, a novel architecture that combines structured sparsity patterns with adaptive computation routing...
```

**2. 运行 Ideation 脚本：**

```bash
python ai_scientist/perform_ideation_temp_free.py \
 --workshop-file "ai_scientist/ideas/my_research_topic.md" \
 --model gpt-4o-2024-05-13 \
 --max-num-generations 20 \
 --num-reflections 5
```

参数说明：

- `--workshop-file`：主题描述 Markdown 文件路径
- `--model`：用于生成想法的 LLM（确保设置了相应的 API 密钥）
- `--max-num-generations`：尝试生成的不同研究想法数量
- `--num-reflections`：LLM 对每个想法的迭代优化次数

**3. 输出：**

脚本会生成一个 JSON 文件（如 `ai_scientist/ideas/my_research_topic.json`），包含结构化的研究想法列表。

#### 运行 AI Scientist-v2 论文生成实验

使用生成的 JSON 文件启动主 AI Scientist-v2 流水线。

```bash
python launch_scientist_bfts.py \
 --load_ideas "ai_scientist/ideas/my_research_topic.json" \
 --load_code \
 --add_dataset_ref \
 --model_writeup o1-preview-2024-11-20 \
 --model_reviewer gpt-4o-2024-05-13 \
 --num-verifiers 3 \
 --max-debug-depth 3 \
 --num-drafts 3 \
 --num-workers 3 \
 --steps 21
```

配置文档位于 `bfts_config.yaml`，主要参数：

- `agent.num_workers`：并行探索路径数量
- `agent.steps`：最大探索节点数量
- `agent.num_seeds`：通常与 `num_workers` 相同
- `search.max_debug_depth`：agent 放弃搜索路径前的调试尝试次数
- `search.num_drafts`：初始根节点数量

### 评价

**优势：**

1. **端到端自动化**：从想法到论文的完整自动化工作流
2. **无模板设计**：v2 版本更灵活，不依赖预定义模板
3. **多模型支持**：支持多种 LLMs，包括开源和闭源
4. ** tree search 策略**：渐进式树搜索可以探索多种可能性
5. **实用价值**：可以加速研究过程，帮助生成论文草稿

**限制：**

1. **成功率**：v2 版本采用探索性方法，成功率可能低于 v1
2. **硬件要求**：需要 NVIDIA GPU + CUDA 环境
3. **专业性**：主要用于机器学习和 AI 领域
4. **成本**：使用高级 LLM（如 GPT-4o、ClaudeSonnet）会产生 API 成本
5. **理解门槛**：需要理解科学方法论和机器学习研究流程

**警告：** 该代码库将执行 LLM 编写的代码，存在各种风险，包括使用危险包、不受控制的网络访问、意外进程生成等。确保在受控沙箱环境（如 Docker 容器）中运行。

**总结：** AI-Scientist-v2 代表了自动化科学工具的重要进展。它不是要取代人类科学家，而是作为一个强大的助手，加速研究过程，生成 novel ideas，并提供可执行的工作流。对于研究人员、学生和 AI 爱好者来说，这是一个非常有趣的工具。

---

## 4. VibeVoice

**GitHub:** [microsoft/VibeVoice](https://github.com/microsoft/VibeVoice)

**Stars:** 10,000+ | **Language:** Python

### 核心功能

VibeVoice 是微软开源的前沿语音 AI 模型家族，包括 Text-to-Speech（TTS）和 Automatic Speech Recognition（ASR）模型。其核心创新在于使用**连续语音 tokenizers**（声学和语义），以极低的帧率（7.5 Hz）操作，从而在保持音频保真度的同时显著提升处理长序列的计算效率。

**主要模型：**

1. **VibeVoice-ASR-7B**：长语音识别模型，支持长达 60 分钟的音频
2. **VibeVoice-TTS-1.5B**：长语音多说话人 TTS 模型，支持长达 90 分钟
3. **VibeVoice-Realtime-0.5B**：实时流式 TTS 模型，参数量仅 0.5B

**技术创新：**

- **Next-Token Diffusion Framework**：使用 LLM 理解文本上下文和对话流程，扩散头生成高保真声学细节
- **7.5 Hz 粗粒度 tokenizers**：比传统方法（通常 50-100 Hz）大幅降低帧率
- **大规模多语言支持**：ASR 模型支持超过 50 种语言
- **实时流式处理**：支持流式音频输入，约 300 毫秒的首次可听延迟

**技术细节：**

- **ASR 模型**：采用 next-token diffusion 框架，处理 64K token 长度的输入
- **TTS 模型**：支持长达 90 分钟的语音生成，最多 4 个不同说话人
- **Realtime 模型**：轻量级设计，参数量 0.5B，支持流式输入

### 适用场景

VibeVoice 特别适合以下应用场景：

1. **长音频处理**：播客、会议记录、长篇访谈的自动转录和生成
2. **多说话人对话**：多人对话系统、虚拟助手、有声书
3. **实时语音交互**：客户服务、虚拟助手、实时翻译
4. **多语言应用**：跨语言内容生成、本地化、国际传播
5. **语音合成**：个性化语音助手、语音导航、语音内容创作

**新闻动态：**

- **2026-03-06**：VibeVoice ASR 已集成到 Hugging Face Transformers 5.3.0 中
- **2026-01-21**：开源 VibeVoice-ASR，支持 60 分钟长音频单次处理
- **2025-12-16**：添加实验性发音人到 Realtime-0.5B 模型
- **2025-12-03**：开源 VibeVoice-Realtime-0.5B 实时流式 TTS 模型
- **2025-08-25**：开源 VibeVoice-TTS-1.5B 长语音多说话人 TTS 模型

### 安装方式

#### 使用 pipx（推荐）

```bash
# 安装 pipx（如果还没有）
pip install pipx

# 安装 VibeVoice
pipx install vibevoice
```

#### 使用 pip

```bash
pip install vibevoice
```

#### Docker（可选）

```bash
docker build -t vibevoice .
docker run -it --rm vibevoice
```

#### 从源码安装

```bash
git clone https://github.com/microsoft/VibeVoice.git
cd VibeVoice
pip install -r requirements.txt
```

### 代码示例

#### VibeVoice-ASR - 语音识别

```python
from transformers import pipeline

# 加载 ASR 模型
pipe = pipeline(
    "automatic-speech-recognition",
    model="microsoft/VibeVoice-ASR",
    torch_dtype=torch.float16,
    device="cuda:0",
)

# 转录音频文件
outputs = pipe(
    "path/to/audio.mp3",
    return_timestamps=True,
    return_word_timestamps=True,
)

print(outputs["text"])
print(outputs["chunks"])  # 包含谁说了什么和何时说的结构化输出
```

**命令行工具：**

```bash
# 转录音频文件
insanely-fast-whisper --file-name audio.mp3

# 使用 VibeVoice-ASR 模型
insanely-fast-whisper --model-name microsoft/VibeVoice-ASR-7B --file-name audio.mp3

# 使用 Flash Attention 2
insanely-fast-whisper --file-name audio.mp3 --flash True
```

#### VibeVoice-TTS - 文本转语音

```python
from vibevoice import TTS

# 初始化 TTS
tts = TTS(model_name="microsoft/VibeVoice-TTS-1.5B")

# 生成语音
audio = tts.generate(
    text="Hello, this is a test of VibeVoice TTS.",
    speaker="default",
    language="en",
)

# 保存音频
audio.save("output.wav")
```

#### VibeVoice-Realtime - 实时流式 TTS

```python
from vibevoice import RealtimeTTS

# 初始化实时 TTS
tts = RealtimeTTS(model_name="microsoft/VibeVoice-Realtime-0.5B")

# 流式生成（支持延迟输入）
for chunk in tts.stream("This is a test of streaming TTS. "):
    play_audio(chunk)
```

#### 完整示例 - 长音频处理

```python
from vibevoice import ASR

# 加载 ASR 模型
asr = ASR(model_name="microsoft/VibeVoice-ASR-7B")

# 处理长达 60 分钟的音频
result = asr.transcribe(
    "path/to/long_audio.mp3",
    return_diarization=True,  # 返回谁说了什么
    return_timestamps=True,    # 返回时间戳
    hotwords=["specific", "technical", "terms"],  # 自定义热词
)

# 输出结构化结果
print(f"Transcript: {result.text}")
print(f"Speakers: {result.speakers}")
print(f"Timestamps: {result.timestamp_chunks}")
```

### 评价

**优势：**

1. **突破性性能**：ASR 支持 60 分钟单次处理，TTS 支持 90 分钟，Realtime 模型仅 300ms 延迟
2. **多语言支持**：ASR 支持 50+ 种语言，TTS 支持英语、中文等
3. **实时能力**：Realtime 模型参数量仅 0.5B，适合部署
4. **结构化输出**：ASR 返回谁说了什么、何时说的结构化信息
5. **自定义热词**：支持用户自定义热词和上下文

**需要注意：**

1. **v2025-09-05 变更**：TTS 代码从仓库中移除（出于负责任 AI 的考虑）
2. **硬件要求**：需要 GPU 进行高效推理
3. ** API 密钥**：部分功能可能需要 Hugging Face token 进行高级特色（如 diarization）
4. **模型大小**：7B ASR 模型较大，对内存要求较高

**总结：** VibeVoice 是微软在语音 AI 领域的重要贡献。它不仅提供了强大的模型，还展示了如何通过创新的 tokenization 和 diffusion framework 实现长序列处理的突破。对于需要处理长音频、多说话人对话的开发者来说，这是一个非常值得考虑的工具。

---

## 5. twenty

**GitHub:** [twentyhq/twenty](https://github.com/twentyhq/twenty)

**Stars:** 41,943 | **Language:** TypeScript

### 核心功能

Twenty 是一个现代化的开源 CRM（客户关系管理）系统，被描述为"开源 CRM 的 #1"。它旨在解决传统 CRM 系统的问题：价格昂贵、用户被锁定、体验过时。Twenty 通过开源模式和现代化技术栈，提供了一个灵活、可定制、无锁定的 CRM 解决方案。

**核心特性：**

1. **可定制的对象和字段**：用户可以创建自定义对象和字段，适应各种业务需求
2. **灵活的视图系统**：支持过滤、排序、分组、看板（Kanban）和表格视图
3. **自定义角色和权限**：创建和管理复杂的权限系统
4. **自动化工作流**：通过触发器和动作自动化业务流程
5. **集成能力**：支持邮件、日历事件、文件存储等

**技术栈：**

- **前端**：React、Jotai（状态管理）、Linaria（CSS-in-JS）、Lingui（国际化）
- **后端**：NestJS、BullMQ（任务队列）、PostgreSQL、Redis
- **构建工具**：TypeScript、Nx

### 适用场景

Twenty 适合以下场景：

1. **中小企业 CRM**：作为企业级 CRM 系统的开源替代
2. **SaaS 创业公司**：快速搭建客户管理系统，避免供应商锁定
3. **定制化需求**：需要高度定制 CRM 的企业
4. **数据隐私要求高**：希望数据完全控制在自己手中的组织
5. **技术团队**：有能力部署和维护开源系统的公司

**与传统 CRM 的对比：**

| 特性 | Traditional CRM | Twenty |
|---|---|---|
| 价格 | 昂贵，按用户收费 | 开源，免费或自托管成本 |
| 定制化 | 有限，需要定制开发 | 高度可定制 |
| 数据所有权 | 供应商锁定 | 用户完全控制 |
| 扩展性 | 受限于供应商 | 可自建扩展 |
| 隐私 | 第三方托管 | 自托管，完全私有 |

### 安装方式

#### Docker Compose（推荐）

```bash
# 克隆仓库
git clone https://github.com/twentyhq/twenty.git
cd twenty

# 复制环境变量示例
cp apps/backend/.env.example apps/backend/.env

# 启动服务
docker-compose up -d
```

#### 本地设置

```bash
# 克隆仓库
git clone https://github.com/twentyhq/twenty.git
cd twenty

# 安装依赖
bun install

# 设置环境变量
cp apps/backend/.env.example apps/backend/.env

# 启动后端
bun run apps/backend:dev

# 启动前端
bun run apps/web:dev
```

**详细文档：**

- [自托管](https://docs.twenty.com/developers/self-host/capabilities/docker-compose)
- [本地设置](https://docs.twenty.com/developers/contribute/capabilities/local-setup)

### 代码示例

#### 创建自定义对象

```typescript
// 定义自定义对象
const customObject = {
  name: "Project",
  label: "Project",
  fields: [
    {
      name: "name",
      label: "Name",
      type: "text",
    },
    {
      name: "client",
      label: "Client",
      type: "relation",
      referenceTo: "Company",
    },
    {
      name: "status",
      label: "Status",
      type: "select",
      options: [
        { label: "Draft", value: "draft" },
        { label: "In Progress", value: "in_progress" },
        { label: "Completed", value: "completed" },
      ],
    },
  ],
};

// 创建对象
await twenty.object.create(customObject);
```

#### 自定义工作流

```typescript
// 定义工作流
const workflow = {
  name: "Notify on Deal Close",
  triggers: [
    {
      type: "record.update",
      object: "Deal",
      condition: {
        field: "stage",
        value: "closed_won",
      },
    },
  ],
  actions: [
    {
      type: "send.email",
      to: "{{record.owner.email}}",
      subject: "Deal Closed: {{record.name}}",
      body: "Congratulations! The deal {{record.name}} has been closed.",
    },
  ],
};

// 创建工作流
await twenty.workflow.create(workflow);
```

#### API 调用示例

```typescript
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:3000",
  headers: {
    Authorization: `Bearer ${process.env.TWENTY_API_KEY}`,
  },
});

// 获取记录
const records = await api.get("/records/Company");

// 创建记录
const newRecord = await api.post("/records/Company", {
  name: "Acme Corp",
  website: "https://acme.com",
  industry: "Technology",
});

// 更新记录
await api.patch(`/records/Company/${recordId}`, {
  name: "Acme Corporation",
});

// 删除记录
await api.delete(`/records/Company/${recordId}`);
```

### 评价

**优势：**

1. **开源免费**：完全开源，避免供应商锁定
2. **现代化体验**：借鉴 Notion、Airtable、Linear 等现代化工具的设计模式
3. **高度可定制**：自定义对象、字段、工作流
4. **社区驱动**：数百名开发者共同建设
5. **自托管能力**：数据完全控制在自己手中

**限制：**

1. **技术门槛**：需要一定的技术能力进行部署和维护
2. **功能相对新**：作为较新的系统，某些功能可能不如成熟 CRM 完善
3. **社区支持**：虽然成长迅速，但社区支持可能不如商业产品成熟
4. **插件生态**：插件生态系统还在发展中

**总结：** Twenty 代表了 CRM 系统的未来方向：开源、现代化、可定制。对于那些希望避免传统 CRM 的高昂成本和锁定问题的公司来说，它是一个非常有吸引力的选择。虽然项目还比较年轻，但势头强劲，社区活跃。

---

## 6. chandra

**GitHub:** [datalab-to/chandra](https://github.com/datalab-to/chandra)

**Stars:** 6,957 | **Language:** Python

### 核心功能

Chandra 是一个先进的 OCR（光学字符识别）模型，专门设计用于处理复杂文档，包括表格、表单、手写文本和完整布局识别。它是 DataLab.to 项目的一部分，旨在提供高精度的文档理解能力。

**核心能力：**

1. **复杂表格处理**：识别和提取复杂表格数据
2. **表单理解**：理解结构化表单字段和值
3. **手写识别**：支持手写文本识别
4. **完整布局分析**：识别文档的完整布局结构
5. **多语言支持**：支持多种语言的文本识别

**技术特点：**

- 基于深度学习的端到端 OCR 系统
- 支持复杂文档布局分析
- 先进的文本识别算法
- 高精度的表格结构理解

### 适用场景

Chandra 非常适合以下应用场景：

1. **文档数字化**：将纸质文档转换为可编辑的数字格式
2. **发票和收据处理**：自动提取发票和收据信息
3. **表格数据提取**：从 PDF 或图像中提取表格数据
4. **手写笔记数字化**：将手写笔记转换为电子文本
5. **法律文档处理**：处理复杂的法律文档和合同
6. **医疗记录数字化**：处理医疗表格和记录

### 安装方式

```bash
# 从 PyPI 安装
pip install chandra-ocr

# 或从源码安装
git clone https://github.com/datalab-to/chandra.git
cd chandra
pip install -e .
```

**依赖要求：**

- Python 3.8+
- PyTorch
- OpenCV
- NumPy

### 代码示例

#### 基本使用

```python
from chandra import OCR

# 初始化 OCR
ocr = OCR()

# 处理图像
result = ocr.recognize("document.jpg")

# 输出结果
print(result.text)  # 识别的文本
print(result.layout)  # 文档布局
print(result.tables)  # 识别的表格
```

#### 处理复杂文档

```python
from chandra import DocumentProcessor

# 创建文档处理器
processor = DocumentProcessor()

# 处理文档
document = processor.process("complex_document.pdf")

# 提取信息
for page in document.pages:
    print(f"Page {page.number}")
    
    # 表格
    for table in page.tables:
        print(f"Table: {table.dataframe}")
    
    # 文本块
    for block in page.text_blocks:
        print(f"Text: {block.text}")
        print(f"Position: {block.position}")
        print(f"Confidence: {block.confidence}")
```

#### 自定义配置

```python
from chandra import OCR, Config

# 创建自定义配置
config = Config(
    language="en",
    detect_tables=True,
    detect_form=True,
    high_resolution=True,
)

# 使用配置初始化 OCR
ocr = OCR(config=config)

# 处理文档
result = ocr.recognize("document.jpg")
```

### 评价

**优势：**

1. **高精度**：在复杂文档处理上表现出色
2. **多功能**：支持表格、表单、手写等多种文档类型
3. **完整布局**：提供详细的布局分析
4. **Python API**：易于集成到 Python 应用中

**限制：**

1. **计算资源**：需要足够的计算资源进行推理
2. **文档支持**：可能不支持所有文档格式
3. **定制化**：某些特定场景可能需要定制化训练

**总结：** Chandra 是一个功能强大的 OCR 工具，特别适合处理复杂的文档结构。对于需要处理大量文档的业务场景，它提供了高精度的解决方案。

---

## 7. superpowers

**GitHub:** [obra/superpowers](https://github.com/obra/superpowers)

**Stars:** 1,000+ | **Language:** TypeScript

### 核心功能

Superpowers 是一个完整的软件开发工作流，专为编码 agent 设计，基于一组可组合的"技能"（skills）和一些初始指令，确保 agent 能够正确使用这些技能。

**核心理念：**

Superpowers 的工作方式是从 agent 启动时开始。当它检测到你在构建东西时，不会立即开始写代码。相反，它会退后一步，先问你真正想要做什么。在从对话中梳理出规格说明后，它会以可读的分块形式向你展示设计。在你批准设计后，agent 会制定一个implementation plan，清晰到让一个热心但经验不足的 junior engineer 也能遵循。然后它会启动 subagent-driven-development 过程，让 agents 完成每个 engineering task，检查和审查它们的工作，并继续前进。

### 适用场景

Superpowers 适用于以下场景：

1. **AI 辅助开发**：使用 Claude Code、Codex、Gemini CLI 等 AI agent 进行软件开发
2. **代码审查自动化**：在任务之间自动请求代码审查
3. **测试驱动开发**：强制执行 RED-GREEN-REFACTOR 测试模式
4. **版本控制工作流**：使用 git worktrees 进行并行开发
5. **系统调试**：使用系统化调试技术

### 安装方式

#### Claude Code 官方市场

```bash
/plugin install superpowers@claude-plugins-official
```

#### Claude Code（通过 Plugin Marketplace）

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

#### Cursor

```text
/add-plugin superpowers
```

#### Codex

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

#### OpenCode

```text
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

#### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
gemini extensions update superpowers  # 更新
```

### 代码示例

#### 基本工作流

1. **构思阶段**（brainstorming）：在写代码之前激活
2. **使用 git worktrees**（using-git-worktrees）：设计批准后激活
3. **编写计划**（writing-plans）：有批准的设计后激活
4. **执行计划**（executing-plans）：激活计划时
5. **测试驱动开发**（test-driven-development）：实现期间激活

#### 技能示例

**测试驱动开发** - RED-GREEN-REFACTOR 循环

```text
// 1. 编写失败的测试（RED）
// 2. 编写最小代码使其通过（GREEN）
// 3. 重构改进代码（REFACTOR）
```

**系统化调试** - 4 阶段根 cause 过程

```text
// 1. 复现问题
// 2. 创建 hypothesis
// 3. 测试 hypothesis
// 4. 分析结果
```

**请求代码审查** - 任务之间的审查

```text
// 自动请求审查
// 报告严重问题
// 阻止关键问题的_progress
```

### 评价

**优势：**

1. **系统化开发**：提供结构化的开发工作流
2. **可组合技能**：技能可以组合和重用
3. **自动触发**：技能根据上下文自动触发
4. **测试驱动**：强制执行 TDD 原则
5. **社区支持**：活跃的社区和 Discord 支持

**限制：**

1. **学习曲线**：需要理解技能系统的工作方式
2. **特定工具**：主要针对特定的 AI coding 工具
3. **配置**：某些设置可能需要配置

**总结：** Superpowers 为 AI 辅助软件开发提供了一个系统化的工作流。它不是简单的代码生成，而是提供了完整的开发流程，从构思到部署。对于那些希望在 AI 辅助开发中保持一致性和质量的团队来说，这是一个有价值的工具。

---

## 8. dexter

**GitHub:** [virattt/dexter](https://github.com/virattt/dexter)

**Stars:** 19,636 | **Language:** TypeScript

### 核心功能

Dexter 是一个自主的金融研究 agent，边工作边思考、计划和学习。它使用任务规划、自我反思和实时市场数据来执行分析。可以想象成 Claude Code，但专门为金融研究构建。

**主要能力：**

1. **智能任务规划**：自动将复杂的查询分解为结构化的研究步骤
2. **自主执行**：选择并执行正确的工具来收集金融数据
3. **自我验证**：检查自己的工作并迭代直到任务完成
4. **实时金融数据**：访问收入报表、资产负债表和现金流量表
5. **安全特性**：内置循环检测和步骤限制以防止失控执行

**技术栈：**

- **运行时**：Bun（v1.0+）
- **语言**：TypeScript
- **integrations**：OpenAI、Financial Datasets、Exa（可选 web 搜索）

### 适用场景

Dexter 非常适合以下场景：

1. **金融研究**：自动执行股票分析、市场研究
2. **投资组合管理**：自动生成投资组合分析报告
3. **财务分析**：分析公司的财务报表
4. **市场趋势分析**：研究市场趋势和 độngstatics
5. **新闻分析**：分析新闻对公司或行业的影响

### 安装方式

```bash
# 克隆仓库
git clone https://github.com/virattt/dexter.git
cd dexter

# 安装依赖
bun install

# 设置环境变量
cp env.example .env

# 编辑 .env 添加 API 密钥
# OPENAI_API_KEY=your-openai-api-key
# ANTHROPIC_API_KEY=your-anthropic-api-key (optional)
# GOOGLE_API_KEY=your-google-api-key (optional)
# XAI_API_KEY=your-xai-api-key (optional)
# OPENROUTER_API_KEY=your-openrouter-api-key (optional)

# 实时市场数据（机构级）
# FINANCIAL_DATASETS_API_KEY=your-financial-datasets-api-key

# 可选：Web 搜索
# EXASEARCH_API_KEY=your-exa-api-key
# TAVILY_API_KEY=your-tavily-api-key
```

### 运行方式

```bash
# 交互式模式
bun start

# 开发模式（watch）
bun dev
```

### 代码示例

#### 基本使用

```bash
# 启动 Dexter
bun start

# 在交互模式下提问
> Analyze AAPL's financial health over the last 5 years
> What are the top 10% gainers in the S&P 500 this quarter?
> Compare Microsoft and Google's revenue growth
```

#### 评估

```bash
# 在所有问题上运行评估
bun run src/evals/run.ts

# 在随机样本上运行
bun run src/evals/run.ts --sample 10
```

#### 调试

Dexter 将所有工具调用记录到 scratchpad 文件进行调试和历史跟踪。每个查询在 `.dexter/scratchpad/` 中创建一个新的 JSONL 文件。

```json
{
  "type": "tool_result",
  "timestamp": "2026-01-30T11:14:05.123Z",
  "toolName": "get_income_statements",
  "args": {
    "ticker": "AAPL",
    "period": "annual",
    "limit": 5
  },
  "result": {...},
  "llmSummary": "Retrieved 5 years of Apple annual income statements showing revenue growth from $274B to $394B"
}
```

#### WhatsApp 集成

```bash
# 连接 WhatsApp 账户（扫描 QR code）
bun run gateway:login

# 启动网关
bun run gateway
```

然后在 WhatsApp 中给自己发消息，询问 Dexter 问题。

### 评价

**优势：**

1. **金融专业知识**：专门为金融研究设计
2. **实时数据**：访问实时金融数据
3. **自我验证**：确保分析的准确性
4. **多 LLM 支持**：支持 OpenAI、Anthropic、Google、xAI 等
5. **WhatsApp 集成**：可以通过 WhatsApp 进行交互

**限制：**

1. **Bun 依赖**：需要 Bun 运行时（vs. Node.js 或 Python）
2. **API 成本**：需要多个 API 密钥（OpenAI、Financial Datasets 等）
3. **计算资源**：复杂的分析可能需要大量计算资源

**总结：** Dexter 是一个功能强大的金融研究 agent，将自动化分析带入了金融领域。对于金融分析师、投资者和研究人员来说，它提供了一个强大的工具来自动化研究任务。

---

## 9. oh-my-claudecode

**GitHub:** [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)

**Stars:** 13,855 | **Language:** TypeScript

### 核心功能

Oh-My-Claudecode（OMC）是一个为 Claude Code 设计的多 agent 协调工具，提供零学习曲线的多 agent 编排体验。它不是让你学习 Claude Code，而是直接使用 OMC。

**主要特性：**

1. **Team 模式**：规范的多 agent 编排表面
2. **32 个专用 agent**：用于架构、研究、设计、测试、数据科学
3. **智能模型路由**：简单任务使用 Haiku，复杂推理使用 Opus
4. **自动委托**：每次都能找到正确的 agent
5. **持久执行**：不会放弃直到任务完全验证完成
6. **成本优化**：智能模型路由可节省 30-50% 的 token
7. **实时可见性**：HUD 状态行显示幕后发生的情况
8. **技能学习**：自动提取并复用问题解决模式

### 适用场景

Oh-My-Claudecode 适用于以下场景：

1. **复杂任务协调**：需要多个 agent 协作的复杂开发任务
2. **代码审查**：使用专门的审查 agent
3. **架构设计**：使用架构师 agent 进行设计评审
4. **测试生成**：使用测试 agent 生成测试用例
5. **研究任务**：使用研究 agent 进行文献综述

### 安装方式

```bash
# 安装插件
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode

# 设置
/setup
/omc-setup

# 开始使用
autopilot: build a REST API for managing tasks
```

### 代码示例

#### Team 模式（推荐）

```bash
/team 3:executor "fix all TypeScript errors"
```

Team 作为分阶段管道运行：
`team-plan → team-prd → team-exec → team-verify → team-fix (loop)`

#### CLI 模式（v4.4.0+）

```bash
# Codex CLI workers
omc team 2:codex "review auth module for security issues"
omc team 2:gemini "redesign UI components for accessibility"
omc team 1:claude "implement the payment flow"

# 检查状态
omc team status auth-review

# 关闭
omc team shutdown auth-review
```

#### Deep Interview（v4.1.0+）

```bash
/deep-interview "I want to build a task management app"
```

Deep interview 使用苏格拉底式提问来澄清你的思考，在编写任何代码之前。它暴露隐藏的假设，并在加权维度上衡量清晰度，确保你知道在执行开始之前要构建什么。

#### 多模型协作（/ccg）

```bash
/ccg Review this PR — architecture (Codex) and UI components (Gemini)
```

## 统计和资源

- **npm 版本**：[![npm version](https://img.shields.io/npm/v/oh-my-claude-sisyphus?color=cb3837)](https://www.npmjs.com/package/oh-my-claude-sisyphus)
- **npm 下载量**：[![npm downloads](https://img.shields.io/npm/dm/oh-my-claude-sisyphus?color=blue)](https://www.npmjs.com/package/oh-my-claude-sisyphus)
- **GitHub stars**：[![GitHub stars](https://img.shields.io/github/stars/Yeachan-Heo/oh-my-claudecode?style=flat&color=yellow)](https://github.com/Yeachan-Heo/oh-my-claudecode/stargazers)
- **许可证**：[MIT](https://opensource.org/licenses/MIT)
- **Discord**：[![Discord](https://img.shields.io/discord/1466022107199574193?color=5865F2&logo=discord&logoColor=white&label=Discord)](https://discord.gg/qRJw62Gvh7)

### 评价

**优势：**

1. **零学习曲线**：即插即用，无需学习曲线
2. **多 agent 协调**：自动协调多个 agent 完成复杂任务
3. **智能路由**：根据任务选择合适的 agent 和模型
4. **实时可见性**：实时了解系统状态
5. **成本优化**：智能模型选择节省 token 成本

**限制：**

1. **特定工具**：主要针对 Claude Code（也有 Codex/Gemini 支持）
2. **依赖**：需要 Claude Code 环境
3. **复杂性**：对于简单任务可能过于复杂

**总结：** Oh-My-Claudecode 将 Claude Code 的能力提升到了新的水平。它提供了一个完整的多 agent 编排系统，使得复杂的开发任务自动化成为可能。对于使用 Claude Code 的开发者来说，这是一个非常有价值的增强工具。

---

## 10. FreeCAD

**GitHub:** [FreeCAD/FreeCAD](https://github.com/FreeCAD/FreeCAD)

**Stars:** 20,000+ | **Language:** C++

### 核心功能

FreeCAD 是一个开源的参数化 3D 建模器，主要用于设计任何大小的真实物体。参数化建模允许你通过返回模型历史记录来轻松修改设计以更改其参数。

**核心能力：**

1. **自由构建**：开源参数化 3D 建模器
2. **从 2D 创建 3D**：绘制几何约束 2D 形状，用作构建其他对象的基础
3. **面向需求设计**：适合产品设计、机械工程、建筑等领域
4. **跨平台**：运行于 Windows、macOS 和 Linux
5. **底层技术**：
   - OpenCASCADE（强大的几何内核）
   - Coin3D 库（Open Inventor 兼容的 3D 场景表示模型）
   - Python（FreeCAD 提供广泛的 Python API）
   - Qt（使用 Qt 构建的图形用户界面）

### 适用场景

FreeCAD 适合以下场景：

1. **产品设计**：设计机械零件、外壳、组件
2. **机械工程**：创建 CAD 图纸、工程图
3. **建筑设计**：建筑模型、室内设计
4. **3D 打印**：设计 3D 打印模型
5. **教育**：学习 CAD 和 3D 建模
6. **爱好者项目**：DIY 项目、汽车改装、房屋规划

### 安装方式

#### 预编译包（稳定版本）

Windows、macOS 和 Linux 的预编译包可在 [最新版本页面](https://github.com/FreeCAD/FreeCAD/releases/latest) 获取。

大多数 Linux 发行版也可直接从软件中心应用程序安装。

#### 开发版本

每周开发版本可在 [发行页面](https://github.com/FreeCAD/FreeCAD/releases/) 获取。

#### 源码编译

查看 [Developers Handbook – Getting Started](https://freecad.github.io/DevelopersHandbook/gettingstarted/) 获取编译说明。

### 代码示例

#### Python 脚本示例

```python
import FreeCAD as App
import Part

# 创建文档
doc = App.newDocument()

# 创建零件
box = Part.makeBox(10, 10, 10)
box_obj = doc.addObject("Part::Feature", "Box")
box_obj.Shape = box

# 重新计算文档
doc.recompute()

# 保存文档
doc.saveAs("/path/to/file.FCStd")
```

#### 命令行使用

```bash
# 启动 FreeCAD
freecad

# 创建工作台
import Part
Part.gd()

# 创建新文档
App.newDocument("MyDesign")

# 查看文档
App.ActiveDocument
```

### 评价

**优势：**

1. **开源免费**：完全开源，无许可费用
2. **功能强大**：参数化建模、装配设计、工程制图
3. **跨平台**：支持 Windows、macOS、Linux
4. **Python API**：广泛的 Python API 用于自动化
5. **社区支持**：活跃的社区和论坛
6. **文档丰富**：wiki 包含广泛的文档

**限制：**

1. **学习曲线**：参数化建模需要时间学习
2. **性能**：大型模型可能较慢
3. **UI**：界面可能不如商业 CAD 时尚

**总结：** FreeCAD 是一个功能强大的开源 CAD 工具，适合从爱好者到专业的各种用户。它提供了一个灵活的参数化建模环境，适用于各种 3D 建模任务。

---

## 11. insanely-fast-whisper

**GitHub:** [Vaibhavs10/insanely-fast-whisper](https://github.com/Vaibhavs10/insanely-fast-whisper)

**Stars:** 11,826 | **Language:** Jupyter Notebook

### 核心功能

Insanely Fast Whisper 是一个 opinionated 的 CLI 工具，用于使用 Whisper 在本地转录音频文件！由 🤗 Transformers、Optimum 和 flash-attn 驱动。

**主要能力：**

- **极快的转录**：使用 Whisper Large v3，在约 98 秒内转录 150 分钟（2.5 小时）的音频
- **CLI 工具**：简单易用的命令行界面
- **Flash Attention 2**：支持 Flash Attention 2 进行加速
- **多种模式**：支持 chunk 和 word 级别的时间戳
- **多模型支持**：支持 Whisper 和 distil-whisper

**性能基准（NVIDIA A100 - 80GB）：**

| 优化类型 | 转录 150 分钟音频时间 |
|---|---|
| large-v3 (Transformers) (fp32) | ~31 分钟 |
| large-v3 (Transformers) (fp16 + batching [24] + bettertransformer) | ~5 分钟 |
| large-v3 (Transformers) (fp16 + batching [24] + Flash Attention 2) | **~2 分钟** |
| distil-large-v2 (Transformers) (fp16 + batching [24] + Flash Attention 2) | **~1 分钟** |

### 适用场景

Insanely Fast Whisper 适合以下场景：

1. **音频转录**：快速将语音转录为文本
2. **视频字幕生成**：为视频生成字幕
3. **播客摘要**：生成播客摘要
4. **语言学习**：练习听力理解
5. **研究**：处理大量音频数据

### 安装方式

```bash
# 使用 pipx（推荐）
pipx install insanely-fast-whisper==0.0.15 --force

# 如果安装了 Python 3.11.XX，pipx 可能会解析版本错误
pipx install insanely-fast-whisper --force --pip-args="--ignore-requires-python"

# 使用 pip
pip install insanely-fast-whisper --ignore-requires-python
```

**依赖：**

- Python 3.8+
- PyTorch
- Transformers
- Optimum

### 代码示例

#### 基本使用

```bash
# 转录音频文件
insanely-fast-whisper --file-name audio.mp3

# 使用 Flash Attention 2
insanely-fast-whisper --file-name audio.mp3 --flash True

# 使用 distil-whisper 模型
insanely-fast-whisper --model-name distil-whisper/large-v2 --file-name audio.mp3
```

#### CLI 选项

```bash
# 查看所有选项
insanely-fast-whisper --help

# 常用选项
--file-name FILE_NAME              # 音频文件路径或 URL
--device-id DEVICE_ID              # GPU 设备 ID（CUDA:"0"，Mac:"mps"）
--transcript-path TRANSCRIPT_PATH  # 保存转录输出的路径（默认: output.json）
--model-name MODEL_NAME            # 预训练模型/检查点名称（默认: openai/whisper-large-v3）
--task {transcribe,translate}      # 执行的任务（默认: transcribe）
--language LANGUAGE                # 输入音频语言（默认: "None" (自动检测)）
--batch-size BATCH_SIZE            # 并行批次数量（默认: 24）
--flash FLASH                      # 使用 Flash Attention 2（默认: False）
--timestamp {chunk,word}           # 时间戳类型（默认: chunk）
```

#### Python 使用

```python
import torch
from transformers import pipeline

pipe = pipeline(
    "automatic-speech-recognition",
    model="openai/whisper-large-v3",
    torch_dtype=torch.float16,
    device="cuda:0",
    model_kwargs={"attn_implementation": "flash_attention_2"},
)

outputs = pipe(
    "audio.mp3",
    chunk_length_s=30,
    batch_size=24,
    return_timestamps=True,
)

print(outputs)
```

### 评价

**优势：**

1. **极快的速度**：使用 Flash Attention 2，在 A100 上 2 分钟转录 150 分钟音频
2. **CLI 工具**：简单易用的命令行界面
3. **灵活配置**：支持多种模型和选项
4. **社区驱动**：社区贡献增强功能

**限制：**

1. **GPU 依赖**：在 NVIDIA GPU 和 Mac 上效果最好
2. **配置复杂**：Flash Attention 2 的安装可能需要额外配置
3. **特定硬件**：可能不支持所有硬件配置

**总结：** Insanely Fast Whisper 是一个性能极高的音频转录工具。对于需要快速处理大量音频数据的人来说，它提供了无与伦比的速度和质量。

---

## 12. onyx

**GitHub:** [onyx-dot-app/onyx](https://github.com/onyx-dot-app/onyx)

**Stars:** 5,000+ | **Language:** Python

### 核心功能

Onyx 是一个功能丰富的、可自托管的 Chat UI，适用于任何 LLM。它易于部署，可以在完全隔离的环境中运行。

Onyx 配备了高级功能，如 Agents、Web Search、RAG、MCP、Deep Research、连接到 40+ 知识源的 Connectors 等。

**主要特性：**

1. **🤖 自定义 Agents**：使用独特的指令、知识和操作构建 AI Agents
2. **🌍 Web Search**：使用 Google PSE、Exa、Serper 以及内置爬虫或 Firecrawl 浏览网络
3. **🔍 RAG**：用于上传文件和来自 connectors 的摄取文档的最佳混合搜索 + 知识图谱
4. **🔄 Connectors**：从 40+ 应用中拉取知识、元数据和访问信息
5. **🔬 Deep Research**：使用 agent 多步搜索获取深入答案
6. **▶️ Actions & MCP**：给 AI Agents 交互外部系统的能力
7. **💻 Code Interpreter**：执行代码来分析数据、渲染图表和创建文件
8. **🎨 图像生成**：根据用户提示生成图像
9. **👥 协作**：聊天共享、反馈收集、用户管理、使用分析等

**与其他 LLM 平台的对比：**

| 特性 | Onyx | 其他平台 |
|---|---|---|
| RAG | 高级混合搜索 + 知识图谱 | 基本 RAG |
| Connectors | 40+ 应用 | 有限数量 |
| Deep Research | 多步 agent 搜索 | 基础搜索 |
| Code Interpreter | 内置 | 可能需要插件 |
| Self-host | 完全自托管 | 可能受限 |

### 适用场景

Onyx 适合以下场景：

1. **企业知识管理**：使用 RAG 和 Connectors 管理企业知识
2. **聊天机器人**：为任何 LLM 构建聊天界面
3. **研究助手**：使用 Deep Research 进行深入研究
4. **内容创建**：使用 Agents 和 Actions 创建内容
5. **数据分析**：使用 Code Interpreter 分析数据

### 安装方式

#### 快速安装

```bash
# 使用安装脚本
curl -fsSL https://onyx.app/install_onyx.sh | bash
```

#### Docker 部署

```bash
# Docker
docker run -d \
  -p 3000:3000 \
  -v onyx_data:/app/data \
  --name onyx \
  onyxdotapp/onyx

# 或使用 docker-compose
docker-compose up -d
```

#### Kubernetes

查看 [文档](https://docs.onyx.app/deployment/local/kubernetes) 获取 Kubernetes 部署指南。

#### Terraform

查看 [文档](https://docs.onyx.app/deployment/local/terraform) 获取 Terraform 部署指南。

### 代码示例

#### 基本使用

1. 访问 `http://localhost:3000`
2. 配置 LLM 提供者（OpenAI、Anthropic、Gemini 或自托管）
3. 创建新的chat
4. 开始提问

#### 连接知识源

```python
# 示例：连接到 Notion
# 1. 获取 Notion 连接器
from onyx.connectors.notion import NotionConnector

# 2. 配置连接器
connector = NotionConnector(
    notion_workspace_id="your-workspace-id",
    notion_secret="your-secret",
)

# 3. 启动连接器
connector.start()
```

#### 创建自定义 Agent

```python
# 示例：创建研究 Agent
from onyx.agents import Agent

research_agent = Agent(
    name="Research Assistant",
    description="Helps with deep research and analysis",
    instructions="You are an expert research assistant...",
    tools=["web_search", "rag", "deep_search"],
)

# 运行 Agent
result = research_agent.run("Research the latest AI trends")
```

### 评价

**优势：**

1. **功能丰富**： Agents、Web Search、RAG、Deep Research 等
2. **可自托管**：完全自托管，数据完全控制
3. **支持多种 LLM**：与任何 LLM 兼容
4. **企业功能**：SSO、RBAC、文档权限等
5. **社区支持**：活跃的社区和 Discord

**限制：**

1. **资源需求**：需要足够的系统资源运行
2. **配置复杂**：某些功能可能需要高级配置
3. **学习曲线**：某些高级功能需要学习

**总结：** Onyx 提供了一个功能强大的聊天界面，适用于任何 LLM。它的 RAG 和 Deep Research 功能特别强大，使其成为企业知识管理和研究工作的不错选择。

---

## 总结

2026 年 3 月 27 日的 GitHub Trending 项目展示了 AI 和开发者工具领域令人兴奋的进展。从 AI 调研工具、深度伪造技术、自动化研究系统到前沿语音 AI 和下一代 CRM，这些项目代表了开源社区的惊人创新力。

**主要趋势：**

1. **AI agent 协同**：多个项目专注于多个 AI agent 的协同工作
2. **语音 AI 突破**：VibeVoice 等项目展示了语音 AI 的最新进展
3. **开发者工具进化**： Twenty、Onyx 等项目重新定义了传统工具
4. **自动化研究**：AI-Scientist-v2 等项目展示了 AI 辅助研究的潜力

这些开源项目为开发者、研究人员和企业提供了强大的工具，展示了开源社区推动技术进步的巨大能力。

**推荐订阅：**

- 每日开源速递：获取每日 GitHub Trending 汇总
- 加入 Discord 社区：与其他开发者交流
- 关注项目更新： watch 仓库获取最新版本

Current time: Saturday, March 28th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-27 22:00 UTC
