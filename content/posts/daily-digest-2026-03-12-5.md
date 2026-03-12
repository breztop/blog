---
title: "每日开源速递 - 2026-03-12"
date: 2026-03-12T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-03-12

## 5. vectorize-io/hindsight - Agent Memory That Learns

### 核心功能
Hindsight 是一个代理记忆系统，旨在创建能够学习的更智能代理。它消除了替代技术（如 RAG 和知识图谱）的缺点，并在长期记忆任务上实现了最先进的性能。

### 适用场景
- AI 代理记忆管理
- 长期记忆系统
- 智能对话系统
- 个性化 AI 助手
- 企业级 AI 应用

### 安装方式
```bash
# 使用 Docker 运行
docker run --rm -it --pull always -p 8888:8888 -p 9999:9999 \
 -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY \
 -v $HOME/.hindsight-docker:/home/hindsight/.pg0 \
 ghcr.io/vectorize-io/hindsight:latest
```

### 代码示例
```python
from hindsight_client import Hindsight

client = Hindsight(base_url="http://localhost:8888")

# 保留信息
client.retain(bank_id="my-bank", content="Alice works at Google as a software engineer")

# 回忆信息
client.recall(bank_id="my-bank", query="What does Alice do?")

# 反思信息
client.reflect(bank_id="my-bank", query="Tell me about Alice")
```

### 评价
Hindsight 在 LongMemEval 基准测试上实现了最先进的性能，被独立研究合作伙伴在弗吉尼亚理工大学和《华盛顿邮报》复现。它已被财富 500 强企业和越来越多的 AI 初创公司使用。该系统通过四个检索策略并行工作：语义、关键词、图和时间，然后合并结果并按相关性排序。

---

Current time: Friday, March 13th, 2026 — 6:00 AM (Asia/Shanghai) / 2026-03-12 22:00 UTC