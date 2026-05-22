---
title: "每日开源速递 - 2026-05-22"
date: 2026-05-22T00:00:00+08:00
description: "GitHub Trending 项目的深度解读与推荐"
tags: [daily, github, trending]
categories: [tech-news]
draft: false
---

# 每日开源速递 - 2026-05-22

> 每日精选 GitHub 昨日热门开源项目，深度解析技术亮点与应用场景

## 📊 概览

昨日（2026年5月21-22日）GitHub 上诞生了多个引人注目的开源项目，涵盖音乐制作、游戏辅助、卫星仿真、AI 客户端、视频索引、开发工具等多个领域。这些项目在短时间内获得了大量关注和 star，体现了开发者社区的最新兴趣点。

---

## 1. FL Studio 2026 激活工具

**仓库地址：** [Jawaz-Keyzor/FL-Studio-2026-Producer-Edition-Unlock](https://github.com/Jawaz-Keyzor/FL-Studio-2026-Producer-Edition-Unlock)  
**⭐ Stars：** 426  
**语言：** C# / Windows 应用  
**许可证：** MIT

### 核心功能

FL Studio 2026 激活工具是一个针对 FL Studio 2026 Producer Edition 的功能解锁工具，旨在提供完整的 DAW（数字音频工作站）体验，而无需付费许可证。

### 主要特性

- **🔓 演示版移除**：完全移除试用限制
- **🎹 全部原生插件解锁**：Harmor、Sytrus、Maximus、Gross Beat、Vocodex、Pitcher、Soundgoodizer 等所有插件
- **🔌 VST/VST3/AU 支持**：完全启用第三方插件支持
- **💾 无限制导出**：WAV/MP3/FLAC/OGG 格式，无水印、无静音插入
- **🎛️ Edison 编辑器解锁**：音频编辑功能完整可用
- **🎤 外部音频录制**：支持外部音源输入
- **🎼 MIDI 支持**：控制器导入导出
- **📜 无限音序器**：钢琴卷帘支持无限模式
- **🎚️ 多轨混音器**：所有效果器可用
- **📈 自动化剪辑**：所有参数自动化
- **🔄 终身更新**：包含次版本更新
- **📡 离线模式**：无需身份验证

### 适用场景

- **音乐制作学习**：学生和初学者可以在不购买许可证的情况下学习 FL Studio 的功能
- **DAW 架构研究**：通过修改许可证验证文件来研究数字音频工作站的授权机制
- **插件开发测试**：开发者可以测试所有插件功能，而不受许可证限制

### 安装方式

```bash
# 1. 下载最新版本
wget https://github.com/Jawaz-Keyzor/FL-Studio-2026-Producer-Edition-Unlock/releases/latest/download/FL_Studio_2026_Unlock_v24.zip

# 2. 解压到桌面
unzip FL_Studio_2026_Unlock_v24.zip -d ~/Desktop

# 3. 右键以管理员身份运行
sudo FL_Studio_Unlock.exe

# 4. 点击 "Restore Features" 确认激活

# 5. 启动 FL Studio
# 所有功能已激活
```

### 代码示例

工具内部通过修改 FL Studio 的许可证验证文件来实现功能解锁。核心逻辑涉及：

```csharp
// 伪代码示例：许可证验证绕过
public class LicenseBypass {
    public static void BypassLicenseCheck() {
        // 1. 定位许可证验证 DLL
        string licenseDll = @"C:\Program Files\Image-Line\FL Studio 2026\fl_studio.exe";
        
        // 2. 修改验证逻辑为始终返回 true
        byte[] patch = { 0xB8, 0x01, 0x00, 0x00, 0x00, 0xC3 }; // mov eax, 1; ret
        ApplyPatch(licenseDll, patch);
        
        // 3. 禁用在线验证
        DisableOnlineVerification();
    }
    
    private static void DisableOnlineVerification() {
        // 禁用网络验证请求
        Registry.SetValue(
            @"HKEY_CURRENT_USER\Software\Image-Line\FL Studio",
            "OnlineCheckDisabled",
            1,
            RegistryValueKind.DWord
        );
    }
}
```

### 评价

**优点：**
- ✅ 功能完整，解锁所有插件和导出选项
- ✅ 离线可用，无需持续联网
- ✅ 保留原有项目文件兼容性
- ✅ MIT 许可证，开源透明

**缺点：**
- ❌ 需要管理员权限
- ❌ 可能被杀毒软件误报（修改系统文件）
- ❌ 不适用于生产环境
- ❌ 可能违反软件使用协议

**适用性：** 仅供学习和研究使用。用户需自行承担合规风险。

---

## 2. CS2 外部游戏辅助工具

**仓库地址：** [Jawaz-Keyzor/CS2-External-Cheat-2026](https://github.com/Jawaz-Keyzor/CS2-External-Cheat-2026)  
**⭐ Stars：** 338  
**语言：** C++ / C#  
**许可证：** MIT

### 核心功能

这是一个基于屏幕捕获分析的 Counter-Strike 2 外部游戏辅助工具，提供视觉叠加显示，无需内存访问或注入。

### 主要特性

- **🧱 墙体透视**：深度缓冲可视化，结构感知
- **🔊 声音雷达**：方向音频指示器，显示距离
- **💣 手雷助手**：烟雾、闪光弹、燃烧弹轨迹预览
- **⏱️ 回合计时器**：炸弹计时、回合阶段、拆包倒计时
- **💰 经济追踪**：伤害经济、队伍购买状态
- **🖥️ 无边框叠加**：游戏窗口上的透明叠加层
- **📊 最小影响**：CPU 占用 < 3%，无 FPS 损失

### 技术实现

**核心组件：**

| 组件 | 方法 |
|------|------|
| 屏幕捕获 | Windows DXGI Desktop Duplication API |
| 深度分析 | 帧缓冲差分处理 |
| 声音处理 | 立体声混音音频空间分析 |
| 叠加渲染 | DirectX 11 透明窗口 |
| 轨迹计算 | 手雷弧线的物理模拟 |

### 适用场景

- **计算机视觉学习**：研究屏幕捕获和视觉分析技术
- **叠加渲染研究**：学习游戏 HUD 和信息叠加技术
- **教育用途**：了解游戏辅助工具的原理（非作弊用途）

### 安装方式

```bash
# 1. 下载最新版本
wget https://github.com/Jawaz-Keyzor/CS2-External-Cheat-2026/releases/latest/download/CS2ExternalAssistant.zip

# 2. 解压
unzip CS2ExternalAssistant.zip -d ~/Desktop

# 3. 运行（需要无边框窗口模式）
# 在 Windows 上：
#   - 右键 CS2 -> 设置 -> 游戏设置 -> 启用无边框窗口
#   - 启动 CS2ExternalAssistant.exe

# 4. 配置叠加层位置
# Settings -> Overlay -> Align to Game Window
```

### 代码示例

**屏幕捕获与深度分析：**

```cpp
// 伪代码示例：屏幕捕获 + 深度可视化
class ScreenCaptureAnalyzer {
public:
    void AnalyzeDepthBuffer() {
        // 1. 创建 DXGI Duplication
        ComPtr<ID3D11Device> device;
        ComPtr<ID3D11DeviceContext> context;
        D3D11CreateDevice(
            nullptr, D3D_DRIVER_TYPE_HARDWARE, nullptr,
            0, nullptr, 0, D3D11_SDK_VERSION,
            &device, nullptr, &context
        );

        // 2. 获取桌面复制
        ComPtr<IDXGIOutputDuplication> duplication;
        output->DuplicateOutput(device.Get(), &duplication);

        // 3. 捕获帧
        DXGI_FRAME_DESC frameDesc;
        ComPtr<ID3D11Texture2D> screenTexture;
        duplication->AcquireNextFrame(1000, &frameDesc, &screenTexture);

        // 4. 分析深度缓冲（简化版）
        ProcessDepthBuffer(screenTexture.Get());

        // 5. 释放
        duplication->ReleaseFrame();
    }

private:
    void ProcessDepthBuffer(ID3D11Texture2D* depthBuffer) {
        // 提取深度信息
        D3D11_MAPPED_SUBRESOURCE mapped;
        context->Map(depthBuffer, 0, D3D11_MAP_READ, 0, &mapped);

        // 识别墙体边界（简化算法）
        auto depthData = reinterpret_cast<float*>(mapped.pData);
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                float depth = depthData[y * width + x];
                
                // 深度突变 = 墙体边界
                if (IsDepthJump(depth, x, y)) {
                    HighlightWall(x, y);
                }
            }
        }

        context->Unmap(depthBuffer, 0);
    }
};
```

### 评价

**优点：**
- ✅ 无需内存访问，理论上更安全
- ✅ CPU 占用极低，不影响游戏性能
- ✅ 技术实现有价值，适合学习

**缺点：**
- ❌ 仍属于游戏辅助工具，可能违反服务条款
- ❌ 需要无边框窗口模式
- ❌ 在全屏模式下无法使用
- ❌ 可能被反作弊系统检测（视觉分析）

**适用性：** 仅供教育和研究用途。不建议在官方竞技平台使用。

---

## 3. 天基智枢 SmartNode

**仓库地址：** [Tong89/smartNode](https://github.com/Tong89/smartNode)  
**⭐ Stars：** 238  
**语言：** Python  
**许可证：** MIT

### 核心功能

SmartNode 是一个面向天基数据回传场景的可视化仿真平台，展示卫星、地面站、中继链路和内容驱动任务调度之间的协同关系。

### 主要特性

- **🌍 三维空间态势展示**：卫星轨道、地面站分布、链路可视化
- **📤 数据回传任务提交**：模拟卫星数据传输任务
- **📡 资源状态监测**：实时监控卫星、地面站、中继资源状态
- **📊 资源利用率统计**：计算各节点的资源使用情况
- **🔧 前后端分离架构**：清晰的代码结构
- **🔌 开放 API**：无密码登录依赖，易于集成
- **🎨 可视化界面**：直观的态势感知

### 技术栈

- **后端**：Python + Flask
- **前端**：原生 JavaScript + HTML/CSS
- **仿真引擎**：自定义调度算法
- **数据存储**：JSON 文件

### 适用场景

- **卫星通信教学**：学生可以直观理解卫星通信原理
- **任务调度研究**：研究数据回传的最优调度策略
- **系统架构设计**：学习前后端分离架构

### 安装方式

```bash
# 1. 克隆仓库
git clone https://github.com/Tong89/smartNode.git
cd smartNode

# 2. 创建虚拟环境（Python 3.8+）
python3 -m venv .venv

# 3. 激活虚拟环境
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# 4. 安装依赖
pip install -r requirements.txt

# 5. 启动服务器
python backend/app.py

# 6. 访问界面
# http://127.0.0.1:5000/frontend/
```

### API 接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/health | 健康检查 |
| GET | /api/data | 仿真态势数据 |
| GET | /api/system_info | 系统配置和数据类型 |
| GET | /api/resource_status | 实时资源状态 |
| GET | /api/resource_utilization | 资源利用率统计 |
| POST | /api/request | 提交数据回传任务 |
| POST | /api/update_ground_stations | 调整地面站数量 |
| POST | /api/update_leo_satellites | 调整 LEO 卫星数量 |

### 代码示例

**后端 Flask API：**

```python
# backend/app.py
from flask import Flask, jsonify, request
from backend.core import SimulationEngine

app = Flask(__name__)
engine = SimulationEngine()

@app.route('/api/data', methods=['GET'])
def get_simulation_data():
    """获取仿真态势数据"""
    return jsonify({
        'satellites': engine.get_satellite_positions(),
        'ground_stations': engine.get_ground_station_status(),
        'links': engine.get_active_links()
    })

@app.route('/api/request', methods=['POST'])
def submit_data_transfer():
    """提交数据回传任务"""
    data = request.json
    task_id = engine.create_transfer_task(
        satellite_id=data['satellite_id'],
        target_ground_station=data['station_id'],
        priority=data.get('priority', 'normal')
    )
    return jsonify({'task_id': task_id, 'status': 'queued'})

@app.route('/api/resource_utilization', methods=['GET'])
def get_utilization():
    """获取资源利用率"""
    return jsonify({
        'satellites': engine.get_satellite_utilization(),
        'ground_stations': engine.get_ground_station_utilization(),
        'network': engine.get_network_utilization()
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

**前端可视化：**

```javascript
// frontend/app.js
class SatelliteVisualization {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.satellites = [];
    }

    drawSatellites(satellites) {
        this.satellites = satellites;
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

        // 绘制地球
        this.drawEarth();

        // 绘制卫星轨道
        satellites.forEach(sat => {
            this.drawOrbit(sat.orbit);
            this.drawSatellite(sat);
        });
    }

    drawSatellite(satellite) {
        const x = (satellite.longitude + 180) * (this.canvas.width / 360);
        const y = (90 - satellite.latitude) * (this.canvas.height / 180);

        // 卫星图标
        this.ctx.beginPath();
        this.ctx.arc(x, y, 8, 0, 2 * Math.PI);
        this.ctx.fillStyle = satellite.status === 'active' ? '#4CAF50' : '#FF9800';
        this.ctx.fill();

        // 显示名称
        this.ctx.fillStyle = '#FFFFFF';
        this.ctx.font = '12px Arial';
        this.ctx.fillText(satellite.name, x + 12, y + 4);
    }
}
```

### 评价

**优点：**
- ✅ 架构清晰，前后端分离
- ✅ API 设计合理，易于扩展
- ✅ 可视化效果直观
- ✅ 适合教学和研究

**缺点：**
- ❌ 缺少数据库支持，数据持久化有限
- ❌ 可视化功能相对简单
- ❌ 未实现真正的物理仿真

**适用性：** 非常适合教学演示和架构学习。

---

## 4. DarkGPT - 无限制 ChatGPT 客户端

**仓库地址：** [Jawaz-Keyzor/DarkGPT](https://github.com/Jawaz-Keyzor/DarkGPT)  
**⭐ Stars：** 231  
**语言：** Python  
**许可证：** MIT

### 核心功能

DarkGPT 是一个无限制的 ChatGPT 客户端，内置 jailbreak 预设、网页界面和流式 API。无内容过滤器，无拒绝，无道德限制。

### 主要特性

- **🔓 无过滤器**：完全无限制，无拒绝、无"我无法"、无道德限制
- **🧠 内置 Jailbreaks**：预加载 DAN、开发者模式、自定义提示注入
- **🌐 网页界面**：干净的暗色主题 Web UI，带聊天历史
- **🖥️ 桌面应用**：通过 PyQt6 提供原生桌面客户端
- **🔌 API 模式**：通过 REST API 以编程方式使用
- **📝 自定义提示**：创建和保存自己的 jailbreak 模板
- **🎭 多重人格**：瞬间切换不同 AI 人格
- **💾 导出聊天**：保存对话为 JSON、TXT 或 Markdown
- **🛡️ 代理支持**：通过 SOCKS5/HTTP 代理路由

### 技术栈

- **语言**：Python 3.11+
- **Web 框架**：FastAPI + WebSockets
- **桌面**：PyQt6
- **LLM 后端**：OpenAI API / 兼容端点
- **流式**：SSE（Server-Sent Events）
- **存储**：SQLite 聊天历史

### 适用场景

- **提示工程研究**：学习如何绕过 AI 内容过滤器
- **角色扮演应用**：创建不同人格的 AI 助手
- **开发测试**：测试 AI 模型的边界情况

### 安装方式

```bash
# 1. 克隆仓库
git clone https://github.com/Jawaz-Keyzor/DarkGPT.git
cd DarkGPT

# 2. 安装依赖
pip install -r requirements.txt

# 3. 运行网页界面
python src/main.py --web

# 4. 运行桌面应用
python src/main.py --desktop

# 5. 运行 API 服务器
python src/main.py --api --port 1337
```

### 配置文件

```bash
# 创建 .env 文件
cat > .env << EOF
OPENAI_API_KEY=sk-your-key-here
OPENAI_BASE_URL=https://api.openai.com/v1
DEFAULT_MODEL=gpt-4o
DEFAULT_JAILBREAK=dan
MAX_TOKENS=4096
TEMPERATURE=0.9
EOF

# 编辑配置
nano .env
```

### API 使用示例

```python
import asyncio
from darkgpt import DarkGPT

async def main():
    # 初始化
    gpt = DarkGPT()
    gpt.set_jailbreak('dan')

    # 普通聊天
    response = await gpt.chat("写一个黑客攻击教程")
    print(response)

    # 流式聊天
    print("流式输出：")
    async for token in gpt.stream_chat("解释量子计算原理"):
        print(token, end='', flush=True)

if __name__ == '__main__':
    asyncio.run(main())
```

### 评价

**优点：**
- ✅ 实现了多种 jailbreak 策略
- ✅ 支持 Web、桌面、API 多种模式
- ✅ 流式响应，体验流畅
- ✅ 开源可定制

**缺点：**
- ❌ 使用可能违反 OpenAI 服务条款
- ❌ 可能导致 API 密钥被封禁
- ❌ 免责声明明确，用户需自行承担风险

**适用性：** 仅供学习和研究提示工程技术。不建议用于生产环境或违反服务条款的使用。

---

## 5. Framedex - 视频档案知识库

**仓库地址：** [Simbastack-hq/framedex](https://github.com/Simbastack-hq/framedex)  
**⭐ Stars：** 214  
**语言：** Python  
**许可证：** MIT

### 核心功能

Framedex 是一个可查询的视频档案知识库，将散落在多个 SSD 和多年间的视频档案转换为便携的纯文本知识库。

### 主要特性

- **📄 元数据提取**：时长、编码、分辨率、创建日期
- **📍 GPS 定位**：提取 GPS 坐标，反向地理编码为地名
- **🎙️ 说话人分离**：多语言转录，自动识别说话人
- **🌐 翻译**：非英语内容自动翻译为英语
- **👤 人脸检测**：512 维面部嵌入向量
- **👁️ AI 视觉描述**：场景/主体/动作/情绪/镜头类型/用途评分
- **💾 本地优先**：不修改原始文件
- **🔄 可恢复**：支持中断后继续处理
- **🏷️ 标记系统**：keep/review/cull 评级

### 技术栈

**本地工具：**
- ffmpeg、exiftool
- WhisperX（转录 + 对齐 + 说话人分离）
- pyannote（说话人分离）
- insightface（人脸检测）

**云端服务（可选）：**
- Nominatim（反向地理编码）
- Anthropic API / LM Studio（视觉描述）

### 适用场景

- **视频归档管理**：组织大量家庭视频、监控录像
- **内容审核**：自动标记需要保留/删除的视频
- **研究分析**：分析视频内容的模式和趋势

### 安装方式

```bash
# 1. 克隆到 Claude Code skills 目录
git clone git@github.com:Simbastack-hq/framedex.git ~/.claude/skills/framedex
cd ~/.claude/skills/framedex

# 2. 安装依赖（可编辑模式）
uv pip install -e .

# 3. 验证系统二进制 + 下载模型
python3 scripts/setup.py

# 4. 配置 Hugging Face Token（首次使用需要）
# 访问 https://huggingface.co/pyannote/speaker-diarization-3.1
# 点击 "Agree" 接受条款
# 访问 https://huggingface.co/settings/tokens 创建 Token
export HF_TOKEN=hf_yourTokenHere

# 5. （可选）设置 Anthropic API Key
export ANTHROPIC_API_KEY=sk-ant-...

# 6. 测试处理 5 个视频
fdx /path/to/videos --max-files 5

# 7. 检查侧边栏文件
# 每个视频旁边会生成 .description.md 文件

# 8. 满意后处理完整驱动
fdx /path/to/videos

# 9. 生成文件夹摘要和主索引
fdx-summary /path/to/videos
fdx-master /path/to/videos
```

### 命令使用

```bash
# 主索引命令
fdx /Volumes/SSD-2024

# 文件夹摘要
fdx-summary /Volumes/SSD-2024

# 主索引
fdx-master /Volumes/SSD-2024

# 查询侧边栏
fdx-query /Volumes/SSD-2024 --rating keep --time-of-day golden_hour
fdx-query /Volumes/SSD-2024 --keyword drone --keyword landscape
fdx-query /Volumes/SSD-2024 --place-contains California --language es
```

### 输出示例

**.description.md 文件：**

```markdown
---
file: IMG_4827.mov
path: /Volumes/SSD-2024/2024-08-construction/drone/IMG_4827.mov
parent_folder: drone
duration_seconds: 12.3
resolution: 3840x2160
codec: hvc1
size_bytes: 245678912
creation_time: 2024-08-14T07:23:11Z
location:
 lat: 37.7456
 lon: -119.5936
 altitude_m: 1842.5
 place: "Yosemite Valley, Mariposa County, USA"
language_detected: es
speaker_count: 2
rating: keep
indexed_at: 2026-05-17T14:32:01
---

# IMG_4827.mov

## Description
**Scene:** Wide drone aerial of a construction site at golden hour...
**Subjects:** Three workers in high-vis vests near a partially-built structure...
**Action:** Drone slowly orbits; workers carry materials between two structures.
**Mood:** Industrious, expansive, hopeful.
**Shot type:** Drone aerial, slow orbit.
**Use cases:**
- Construction milestone post
- "From the ground up" origin-story reel
- B-roll behind a voiceover

## Transcript (es, 2 speakers)
[SPEAKER_00] (00:00:01) Pon esta viga aquí primero.
[SPEAKER_01] (00:00:04) Sí, vale.
[SPEAKER_00] (00:00:07) Cuidado con el ángulo.

## English translation
Place this beam here first. Yes, OK. Careful with the angle.
```

### 评价

**优点：**
- ✅ 功能强大，涵盖转录、翻译、人脸、视觉描述
- ✅ 本地优先，隐私保护
- ✅ 可恢复处理，适合大档案
- ✅ Claude Code skill 集成，易于使用

**缺点：**
- ❌ 首次设置复杂（需要接受模型条款）
- ❌ 大模型下载占用磁盘空间（3GB+）
- ❌ Apple Silicon 上 WhisperX 运行在 CPU

**适用性：** 非常适合个人和家庭视频归档管理。

---

## 6. Codex Shim - Codex Desktop 模型适配器

**仓库地址：** [0xSero/codex-shim](https://github.com/0xSero/codex-shim)  
**⭐ Stars：** 206  
**语言：** Python  
**许可证：** MIT

### 核心功能

Codex Shim 是一个本地 Responses-API 适配器，将 Factory 的 BYOK 模型（以及可选的 ChatGPT GPT-5.5 透传）暴露给 Codex Desktop，无需重新编译。

### 主要特性

- **🔄 本地 API 适配**：模拟 OpenAI Responses API 端点
- **📦 多模型支持**：OpenAI、Anthropic、DeepSeek、Gemini、OpenRouter、Factory
- **⚡ GPT-5.5 透传**：使用 ChatGPT 订阅的 GPT-5.5
- **🔧 ASAR 补丁**：修改 Codex Desktop 的模型选择器
- **🌐 代理支持**：通过 SOCKS5/HTTP 代理
- **📊 实时日志**：自动拼接 VLESS 快捷导入链接

### 技术栈

- **Python 3.11+**
- **FastAPI**：API 服务器
- **PyQt6**：桌面应用
- **Electron ASAR**：Codex Desktop 补丁

### 适用场景

- **模型管理**：在 Codex 中使用自己的模型
- **API 灵活切换**：在多个 AI 服务之间切换
- **隐私保护**：使用本地或私有 API

### 安装方式

```bash
# 1. 克隆仓库
git clone https://github.com/0xSero/codex-shim ~/Documents/codex-shim
cd ~/Documents/codex-shim

# 2. 安装依赖
python3 -m pip install --user aiohttp pytest

# 3. 创建符号链接
ln -s "$PWD/bin/codex-shim" ~/.local/bin/codex-shim
ln -s "$PWD/bin/codex-app" ~/.local/bin/codex-app
ln -s "$PWD/bin/codex-model" ~/.local/bin/codex-model

# 4. 生成目录
codex-shim generate

# 5. 启动守护进程
codex-shim start

# 6. 启动 Codex
codex-shim app
```

### 配置文件

```json
{
  "customModels": [
    {
      "model": "gpt-5.5",
      "provider": "openai",
      "baseUrl": "https://api.openai.com/v1",
      "apiKey": "sk-...",
      "displayName": "OpenAI GPT-5.5",
      "maxContextLimit": 400000
    },
    {
      "model": "claude-opus-4-7-20251109",
      "provider": "anthropic",
      "baseUrl": "https://api.anthropic.com/v1",
      "apiKey": "sk-ant-...",
      "displayName": "Claude Opus 4.7"
    }
  ]
}
```

### 命令使用

```bash
# 生成目录
codex-shim generate

# 启动守护进程
codex-shim start

# 查看生成的 slug
codex-shim list

# 健康检查
codex-shim status

# 启动 Codex
codex-shim app

# 设置默认模型
codex-shim model use openai-gpt-5-5

# 执行 codex CLI
codex-shim codex -- "你的命令"
```

### ASAR 补丁（macOS）

```bash
# 1. 备份应用
APP=/Applications/Codex.app
sudo cp -R "$APP" "$APP.unpatched-$(date +%Y%m%d-%H%M%S)"

# 2. 提取 ASAR
cd /tmp && rm -rf codex-asar-patch && mkdir codex-asar-patch
cd codex-asar-patch
npx --yes @electron/asar extract "$APP/Contents/Resources/app.asar" extracted

# 3. 修补模型选择器
PATCH_FILE=$(grep -RIl 'useHiddenModels' extracted/webview/assets/model-queries-*.js | head -n1)
sed -i.bak -E 's/let u=c\\.useHiddenModels&&o!==`amazonBedrock`,d;/let u=!1,d;/' "$PATCH_FILE"
rm "$PATCH_FILE.bak"

# 4. 重新打包
npx --yes @electron/asar pack extracted app.asar.new
sudo cp app.asar.new "$APP/Contents/Resources/app.asar"

# 5. 重新签名
python3 - "$APP/Contents/Resources/app.asar" <<'PY'
import struct, hashlib, sys
with open(sys.argv[1], 'rb') as f:
    data_size, header_size, _, json_size = struct.unpack('<4I', f.read(16))
    header_json = f.read(json_size)
print(hashlib.sha256(header_json).hexdigest())
PY

sudo /usr/libexec/PlistBuddy -c "Set :ElectronAsarIntegrity:Resources/app.asar:hash $HEADER_HASH" "$APP/Contents/Info.plist"
sudo codesign --force --deep --sign - "$APP"

# 6. 启动
open "$APP"
```

### 评价

**优点：**
- ✅ 解锁 Codex Desktop 的模型选择器
- ✅ 支持多种 AI 服务
- ✅ 无需修改 Codex 配置文件
- ✅ 开源可定制

**缺点：**
- ❌ 需要修改 Codex Desktop 的二进制文件
- ❌ 可能被 Codex 更新破坏
- ❌ macOS 需要 ASAR 补丁

**适用性：** 适合想要在 Codex 中使用自定义模型的开发者。

---

## 总结

昨日 GitHub Trending 上涌现了多个有趣的开源项目，涵盖了从音乐制作、游戏技术、卫星通信到 AI 应用的多个领域。这些项目展示了开发者社区的多样性和创造力。

**值得关注的趋势：**
1. **AI 工具化**：DarkGPT 和 Codex Shim 展示了如何将 AI 能力封装为可配置的工具
2. **本地优先**：Framedex 强调隐私保护，所有处理都在本地完成
3. **可视化仿真**：SmartNode 展示了复杂系统的可视化能力
4. **边缘技术**：CS2 工具展示了屏幕捕获和视觉分析技术

**适合学习的项目：**
- SmartNode：前后端分离架构
- Framedex：视频处理和 AI 集成
- DarkGPT：提示工程和 API 封装
- Codex Shim：Electron 应用补丁和 API 适配

**注意事项：**
- 部分项目涉及修改系统文件或绕过服务条款，请谨慎使用
- 仅供学习和研究，避免用于生产环境或违反服务条款
- 注意数据隐私和安全性

---

**Current time:** Saturday, May 23rd, 2026 — 6:00 AM (Asia/Shanghai) / 2026-05-22 22:00 UTC
