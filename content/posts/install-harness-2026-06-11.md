---
title: "Harness Open Source 安装指南 - 2026-06-11"
date: 2026-06-11T23:00:00+08:00
description: "下一代开源 DevOps 平台 Harness (Drone 继任者) 的快速安装与上手指南"
tags: [install, harness, devops, ci-cd, docker]
categories: [tech-tutorial]
draft: false
---

# Harness Open Source 安装指南

> Harness 最近入选 Fortune 2026 美国最具创新公司榜 #24（与 OpenAI、Anthropic 同列），开源仓库 36.6k+ Star。本文带你 10 分钟把 Harness 跑起来。

## 一、Harness 是什么

**Harness Open Source** 是 Harness 公司（harness.io）推出的**端到端开源开发平台**，是经典 CI 工具 **Drone** 的官方继任者。它在 Drone 的 CI/CD 能力之上，新增了：

- **代码托管**（SCM）—— 内置 Git 仓库，告别外部 GitHub 依赖
- **CI/CD 流水线** —— 容器化构建，YAML 配置
- **Gitspaces** —— 云端开发环境（类似 GitHub Codespaces）
- **制品仓库**（Artifact Registry）—— 镜像、二进制包统一管理

技术栈：Go 60.7% + TypeScript 34.6%，Apache-2.0 协议。

> 相比 Drone：Harness 把"仓库 + 流水线 + 环境 + 制品"四件套打包成一套开箱即用的平台，对小团队和个人开发者特别友好——你不再需要单独搭 GitLab + Drone + Harbor + S3。

---

## 二、环境要求

| 组件 | 最低版本 | 建议版本 |
|---|---|---|
| Docker | 20.10+ | 24.0+ |
| Docker Compose | v2 | v2.20+ |
| 系统 | Linux / macOS / WSL2 | Linux x86_64 |
| 内存 | 2 GB | 4 GB+ |
| 磁盘 | 10 GB | 20 GB+ |

需要开启 Docker socket 挂载权限（用于执行流水线内的容器构建任务）。

---

## 三、极速安装（Docker 单命令）

官方提供开箱即用的 Docker 镜像，一条命令起服务：

```bash
docker run -d \
  -p 3000:3000 \
  -p 3022:3022 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /tmp/harness:/data \
  --name harness \
  --restart always \
  harness/harness
```

参数说明：
- `-p 3000:3000`：Web UI 端口
- `-p 3022:3022`：内置 Git SSH 端口
- `/var/run/docker.sock`：挂载 Docker socket，让 Harness 能为每个流水线启动构建容器
- `/tmp/harness`：数据库 + 仓库数据卷（**强烈建议改成命名卷或 bind mount**，否则容器重建数据全丢）

> ⚠️ 启动需 30-60 秒，期间 `http://localhost:3000` 会返回 502，等看到日志 `service ready` 再访问。

---

## 四、首次登录与初始化

启动后浏览器打开 `http://localhost:3000`：

1. **默认账号**：`admin` / `changeit`（首次登录强制改密）
2. 创建一个新用户或继续用 admin
3. 进入 **Account Settings → Default Settings**，把 Harness AI 关掉（开源版不依赖）

### 4.1 创建第一个代码仓库

进入 **Repositories → New Repository**：

```bash
# 本地推送到内置 Git
git remote add harness http://localhost:3000/admin/<repo>.git
git push harness main
```

按提示输入账号密码即可。仓库自动启用 CI。

### 4.2 创建一个 Pipeline

在仓库根目录加 `.harness/pipeline.yml`：

```yaml
kind: pipeline
name: hello-harness
steps:
  - name: build
    image: alpine
    commands:
      - echo "Hello from Harness!"
      - uname -a
      - cat /etc/os-release
  - name: test
    image: node:20
    commands:
      - node -v
      - npm -v
trigger:
  branch:
    - main
```

Push 后 Harness 自动触发，看到绿色 ✅ 就是成功。

---

## 五、生产级部署（Docker Compose）

个人试用上面的单命令就够了；想长期跑推荐 Compose：

```yaml
# docker-compose.yml
version: "3.9"

services:
  harness:
    image: harness/harness:latest
    container_name: harness
    restart: always
    ports:
      - "3000:3000"
      - "3022:3022"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - harness_data:/data
    environment:
      - HARNESS_LOG_LEVEL=info
      # 反向代理时取消注释
      # - HARNESS_PUBLIC_BASE_URL=https://ci.example.com

volumes:
  harness_data:
    name: harness_data
```

启动：

```bash
docker compose up -d
docker compose logs -f harness
```

升级：

```bash
docker compose pull harness
docker compose up -d
```

---

## 六、K8s 部署（可选）

Helm 方式（适合团队）：

```bash
helm repo add harness https://harness.github.io/harness
helm install harness harness/harness \
  -n ci-cd --create-namespace \
  --set server.replicaCount=2 \
  --set server.persistence.size=20Gi
```

生产建议至少 2 副本 + external DB（PostgreSQL）+ 外部存储（Ceph / S3 兼容），具体参考官方 [Reference Architecture](https://developer.harness.io)。

---

## 七、常见问题

### Q1: 启动报 `bind: address already in use`？
3000/3022 被占用。换端口：
```bash
docker run -d -p 9000:3000 -p 9022:3022 ... harness/harness
```

### Q2: 流水线一直 Pending？
通常是 Docker socket 挂载问题。检查：
```bash
ls -la /var/run/docker.sock
# 必须存在且容器内可访问
```

### Q3: 忘了 admin 密码？
```bash
docker exec -it harness ./gitness user reset-password admin
```

### Q4: 怎么对接外部 GitHub/GitLab？
进入 **Repositories → Add Repository → GitHub/GitLab**，填入 OAuth 或 PAT 即可，Harness 会作为外部仓库的 CI runner。

### Q5: 和 Drone 还兼容吗？
**部分兼容**。Harness 读 Drone 的 `.drone.yml`，但 Harness 专属的 `.harness/pipeline.yml` 才支持 SCM + Gitspaces + Registry 全家桶。建议新项目直接用 Harness 语法。

---

## 八、评价

**优点**：
- 一条命令起服务，5 分钟看到 Web UI，对个人开发者极友好
- 容器隔离的构建环境，资源利用率高（vs Jenkins master/agent 模式）
- Drone 用户无缝迁移，学习成本几乎为零
- 真正"开箱即用"的端到端平台，省一堆自建组件

**缺点**：
- 项目仍处于快速迭代期，部分 UI 翻译不完整
- 企业级特性（高级权限、审计、SSO）需要 Harness 商业版
- 中文社区资料偏少，问题排查主要靠英文 GitHub Issues

**适合谁**：
- 想从 GitHub Actions / Jenkins / GitLab CI 找替代品的个人/小团队
- 需要自托管 CI 但不想维护多个组件的 DevOps 工程师
- 想"一个 docker run 跑完整个 DevOps 平台"的极简主义者

**不适合谁**：
- 需要复杂审批流、SOX 合规审计的大型企业（用商业版 Harness）
- 已深度绑定 GitHub / GitLab 生态且不想迁出的团队

---

## 九、参考资料

- 官网：https://harness.io
- 开源仓库：https://github.com/harness/harness
- 官方文档：https://developer.harness.io/docs/open-source
- Docker Hub：https://hub.docker.com/r/harness/harness
- 中文社区讨论：InfoQ / 掘金搜 "Harness" 有不少实测文章

---

> 📅 本文基于 Harness `v2.28.2`（2026-04-20 发布）撰写。如果命令有变以官方仓库 README 为准。
