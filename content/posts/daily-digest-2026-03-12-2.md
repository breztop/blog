---
title: "每日开源速递 - 2026-03-12"
date: 2026-03-12T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-03-12

## 2. fishaudio/fish-speech - SOTA Open Source TTS

### 核心功能
Fish Audio S2 是一个最先进的开源文本转语音（TTS）系统，训练在超过 1000 万小时的音频数据上，支持 50 多种语言。它结合了强化学习对齐和双自回归架构，生成自然、真实且富有情感的语音。

### 适用场景
- 多语言语音合成
- 情感丰富的语音生成
- 个性化语音助手
- 无障碍技术
- 内容创作和播客制作

### 安装方式
```bash
# 通过 pip 安装
pip install fish-speech

# 或者使用 Docker
docker run --rm -it --pull always -p 8888:8888 -p 9999:9999 \
 -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY \
 -v $HOME/.hindsight-docker:/home/hindsight/.pg0 \
 ghcr.io/vectorize-io/hindsight:latest
```

### 代码示例
```python
from fish_speech import FishSpeech

# 初始化模型
model = FishSpeech()

# 生成语音
text = "Hello, this is a sample text to speech conversion."
audio = model.synthesize(text)

# 保存音频
audio.save("output.wav")
```

### 评价
Fish Audio S2 在 Seed-TTS Eval 上实现了最佳 WER（中文 0.54%，英文 0.99%），在 Audio Turing Test 上达到 0.515 后验均值，在 EmergentTTS-Eval 上获得 81.88% 的胜率。它支持细粒度的语音控制，使用自然语言标签如 [laugh]、[whispers] 和 [super happy]。

---

Current time: Friday, March 13th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-12 22:00 UTC