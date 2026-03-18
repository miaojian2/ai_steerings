---
inclusion: manual
---

# OpenClaw 全部技能详细指南

## 概述

OpenClaw 内置 60+ 个技能，分布在 `skills/` 目录（内置技能）和 `extensions/*/skills/` 目录（扩展技能）。技能覆盖密码管理、笔记文档、任务管理、邮件、消息、媒体处理、智能家居、开发工具、AI/LLM、实用工具、音乐、语音、飞书集成、工作流自动化等多个领域。

## 技能来源

| 来源 | 路径 | 说明 |
|------|------|------|
| **内置技能** | `skills/` | 随 OpenClaw 分发的核心技能 |
| **扩展技能** | `extensions/*/skills/` | 随扩展插件分发的技能 |
| **托管技能** | ClawHub | 从技能市场安装的技能 |
| **工作区技能** | `~/.openclaw/workspace/skills/` | 用户自定义技能 |
| **Agent 技能** | `.agents/skills/` | 仓库级别的 Agent 专用技能 |

## 技能系统在工程中的作用

### 架构位置

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenClaw Gateway                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Agent Runtime                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │   System    │  │   Skills    │  │    Tools    │     │   │
│  │  │   Prompt    │◄─┤   Loader    │─►│   Registry  │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│         ▲                    ▲                    ▲             │
│         │                    │                    │             │
│  ┌──────┴──────┐      ┌──────┴──────┐      ┌──────┴──────┐    │
│  │   Bundled   │      │  Extension  │      │  Workspace  │    │
│  │   Skills    │      │   Skills    │      │   Skills    │    │
│  │  (skills/)  │      │(extensions/)│      │ (~/.../     │    │
│  │             │      │             │      │  skills/)   │    │
│  └─────────────┘      └─────────────┘      └─────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 技能的工程作用

| 作用 | 说明 |
|------|------|
| **扩展 Agent 能力** | 技能为 Agent 提供特定领域的工具和知识，使其能够执行密码管理、笔记操作、智能家居控制等任务 |
| **提供上下文指导** | SKILL.md 中的描述和示例会注入到 Agent 的系统提示词中，指导 Agent 何时、如何使用该技能 |
| **依赖管理** | 技能声明所需的 CLI 工具、环境变量、配置项，系统自动检查并提示安装 |
| **平台适配** | 技能可声明支持的操作系统（darwin/linux/windows），实现跨平台兼容 |
| **模块化设计** | 技能独立于核心代码，便于添加、更新、禁用，不影响系统稳定性 |

### 技能运行时机

```
用户消息 ──► Agent 分析 ──► 匹配技能描述 ──► 调用技能工具 ──► 返回结果
                │
                ▼
        ┌───────────────────────────────────────────────────┐
        │              技能匹配流程                          │
        │  1. Agent 解析用户意图                            │
        │  2. 根据 SKILL.md 中的 description 匹配技能       │
        │  3. 检查技能依赖是否满足（bins/env/config）        │
        │  4. 执行技能定义的工具/脚本                       │
        │  5. 返回执行结果给 Agent                          │
        └───────────────────────────────────────────────────┘
```

### 技能触发方式

| 触发方式 | 说明 | 示例 |
|----------|------|------|
| **意图匹配** | Agent 根据用户消息自动匹配技能描述 | "帮我查一下天气" → weather 技能 |
| **显式调用** | 用户明确指定使用某技能 | "用 summarize 总结这个链接" |
| **触发短语** | SKILL.md 中定义的特定短语 | "transcribe this video" → summarize 技能 |
| **工具调用** | Agent 直接调用技能注册的工具 | `slack.sendMessage()` |

### 技能加载流程

```
Gateway 启动
    │
    ▼
扫描技能目录
    │
    ├── skills/ (bundled)
    ├── extensions/*/skills/ (extension)
    ├── ~/.openclaw/managed-skills/ (managed)
    └── ~/.openclaw/workspace/skills/ (workspace)
    │
    ▼
解析 SKILL.md
    │
    ├── 提取 name, description, metadata
    ├── 解析依赖要求 (requires)
    └── 加载安装配置 (install)
    │
    ▼
检查依赖
    │
    ├── 检查 bins（CLI 工具是否存在）
    ├── 检查 env（环境变量是否设置）
    └── 检查 config（配置项是否存在）
    │
    ▼
注册到 Agent
    │
    ├── 注入技能描述到系统提示词
    └── 注册技能工具到工具注册表
```

### 核心代码文件

| 文件 | 职责 |
|------|------|
| `src/agents/skills.ts` | 技能加载、解析、合并逻辑 |
| `src/agents/skills-install.ts` | 技能安装和依赖检查 |
| `src/agents/skills-status.ts` | 技能状态查询 |
| `src/cli/skills-cli.ts` | CLI 命令实现 |
| `src/agents/agent-runtime.ts` | Agent 运行时，注入技能 |

---

## 技能分类总览

### 内置技能 (skills/)

| 分类 | 技能数量 | 技能列表 |
|------|----------|----------|
| 密码/安全 | 1 | 1password |
| 笔记/文档 | 4 | apple-notes, bear-notes, notion, obsidian |
| 任务/提醒 | 3 | apple-reminders, things-mac, trello |
| 邮件 | 2 | himalaya, gog |
| 消息 | 4 | bluebubbles, imsg, discord, slack |
| 媒体/音频 | 10 | camsnap, gifgrep, openai-image-gen, nano-banana-pro, openai-whisper, openai-whisper-api, sag, sherpa-onnx-tts, video-frames, peekaboo |
| 智能家居 | 4 | openhue, eightctl, blucli, sonoscli |
| 开发工具 | 6 | coding-agent, github, gh-issues, skill-creator, tmux, mcporter |
| AI/LLM | 3 | gemini, oracle, model-usage |
| 实用工具 | 11 | blogwatcher, clawhub, goplaces, healthcheck, nano-pdf, node-connect, ordercli, session-logs, weather, xurl, summarize |
| 音乐 | 3 | songsee, spotify-player, sonoscli |
| 语音 | 1 | voice-call |
| WhatsApp | 1 | wacli |
| 画布 | 1 | canvas |

### 扩展技能 (extensions/*/skills/)

| 扩展 | 技能 | 说明 |
|------|------|------|
| **feishu** | feishu-doc | 飞书文档读写操作 |
| **feishu** | feishu-drive | 飞书云盘文件管理 |
| **feishu** | feishu-wiki | 飞书知识库导航 |
| **feishu** | feishu-perm | 飞书权限管理 |
| **diffs** | diffs | 生成可分享的 diff 视图 |
| **open-prose** | prose | OpenProse VM 多代理工作流编排 |
| **acpx** | acp-router | ACP 代理路由（Pi/Claude/Codex/OpenCode/Gemini） |
| **lobster** | lobster | 多步骤工作流执行与审批 |

### Agent 技能 (.agents/skills/)

| 技能 | 说明 |
|------|------|
| parallels-discord-roundtrip | macOS Parallels 烟雾测试与 Discord 端到端验证 |

---

## 详细技能说明

### 🔐 密码/安全

#### 1password
- **描述**: 通过 1Password CLI 管理密码和安全笔记
- **工程作用**: 为 Agent 提供安全的密码和敏感信息访问能力，避免在对话中暴露凭据
- **运行时机**: 当用户请求获取密码、API Key、安全笔记等敏感信息时触发
- **依赖**: `op` CLI
- **安装**: `brew install 1password-cli`
- **主要功能**: 读取密码、创建条目、搜索保险库
- **示例**: `op item get "GitHub" --fields password`

---

### 📝 笔记/文档

#### apple-notes
- **描述**: 通过 AppleScript 管理 Apple Notes
- **工程作用**: 集成 macOS 原生笔记应用，实现笔记的创建、搜索和管理
- **运行时机**: 当用户请求创建笔记、搜索笔记内容、列出笔记文件夹时触发
- **平台**: macOS
- **主要功能**: 创建笔记、搜索笔记、列出文件夹
- **示例**: 使用 `osascript` 调用 Notes.app

#### bear-notes
- **描述**: 通过 x-callback-url 管理 Bear 笔记
- **工程作用**: 集成 Bear 笔记应用，支持 Markdown 笔记的创建和管理
- **运行时机**: 当用户请求在 Bear 中创建、搜索、打开笔记或管理标签时触发
- **平台**: macOS
- **依赖**: Bear.app
- **主要功能**: 创建/搜索/打开笔记、管理标签
- **示例**: `open "bear://x-callback-url/create?title=Test&text=Content"`

#### notion
- **描述**: 通过 Notion API 管理页面和数据库
- **工程作用**: 集成 Notion 工作空间，实现页面和数据库的 CRUD 操作
- **运行时机**: 当用户请求创建 Notion 页面、查询数据库、更新内容时触发
- **依赖**: `NOTION_API_KEY` 环境变量
- **主要功能**: 创建页面、查询数据库、更新内容
- **示例**: 使用 curl 调用 Notion API

#### obsidian
- **描述**: 管理 Obsidian 笔记库
- **工程作用**: 集成 Obsidian 知识库，支持 Markdown 笔记和双向链接管理
- **运行时机**: 当用户请求在 Obsidian 中创建、搜索、编辑笔记或管理链接时触发
- **平台**: macOS/Linux
- **主要功能**: 创建/搜索/编辑笔记、管理链接
- **配置**: 需要设置 vault 路径

---

### ✅ 任务/提醒

#### apple-reminders
- **描述**: 通过 AppleScript 管理 Apple Reminders
- **工程作用**: 集成 macOS 原生提醒事项应用，实现任务管理
- **运行时机**: 当用户请求创建提醒、列出待办事项、标记任务完成时触发
- **平台**: macOS
- **主要功能**: 创建提醒、列出任务、标记完成
- **示例**: 使用 `osascript` 调用 Reminders.app

#### things-mac
- **描述**: 通过 `things` CLI 管理 Things 3
- **工程作用**: 集成 Things 3 任务管理应用，支持 GTD 工作流
- **运行时机**: 当用户请求添加任务、查看收件箱/今日/即将到来的任务、搜索任务时触发
- **平台**: macOS
- **依赖**: `things` CLI (Go)
- **安装**: `go install github.com/ossianhempel/things3-cli/cmd/things@latest`
- **主要功能**:
  - 读取: `things inbox`, `things today`, `things search "query"`
  - 写入: `things add "Title" --when today --deadline 2026-01-02`
  - 更新: `things update --id <UUID> --auth-token <TOKEN> --completed`
- **注意**: 需要 Full Disk Access 权限

#### trello
- **描述**: 通过 Trello REST API 管理看板
- **工程作用**: 集成 Trello 看板，实现项目和任务的可视化管理
- **运行时机**: 当用户请求查看看板、创建卡片、移动卡片、添加评论时触发
- **依赖**: `TRELLO_API_KEY`, `TRELLO_TOKEN`, `jq`
- **主要功能**: 列出看板/列表/卡片、创建卡片、移动卡片
- **示例**: `curl "https://api.trello.com/1/members/me/boards?key=$TRELLO_API_KEY&token=$TRELLO_TOKEN"`

---

### 📧 邮件

#### himalaya
- **描述**: 通过 himalaya CLI 管理邮件
- **工程作用**: 提供通用邮件客户端能力，支持 IMAP/SMTP 协议
- **运行时机**: 当用户请求列出邮件、读取邮件内容、发送邮件、搜索邮件时触发
- **依赖**: `himalaya` CLI
- **安装**: `brew install himalaya`
- **主要功能**: 列出邮件、读取邮件、发送邮件、搜索
- **示例**: `himalaya list`, `himalaya read <id>`

#### gog (Gmail)
- **描述**: 通过 gog CLI 管理 Gmail
- **工程作用**: 专门针对 Gmail 的邮件管理，支持 Gmail 特有功能
- **运行时机**: 当用户请求管理 Gmail 邮件、搜索、发送、管理标签时触发
- **依赖**: `gog` CLI
- **安装**: `go install github.com/steipete/gog/cmd/gog@latest`
- **主要功能**: 列出邮件、搜索、发送、管理标签
- **示例**: `gog list --limit 20`, `gog search "from:boss"`

---

### 💬 消息

#### bluebubbles
- **描述**: 通过 BlueBubbles 服务器发送 iMessage
- **工程作用**: 在非 macOS 环境下提供 iMessage 访问能力（通过 BlueBubbles 服务器中转）
- **运行时机**: 当用户请求通过 BlueBubbles 发送/读取 iMessage 时触发
- **依赖**: BlueBubbles 服务器配置
- **主要功能**: 发送消息、读取消息、管理对话
- **配置**: `channels.bluebubbles.serverUrl`, `channels.bluebubbles.password`

#### imsg
- **描述**: 通过 imsg CLI 发送 iMessage
- **工程作用**: 在 macOS 上直接访问 iMessage，实现消息发送和历史搜索
- **运行时机**: 当用户请求发送 iMessage、读取消息历史、搜索聊天记录时触发
- **平台**: macOS
- **依赖**: `imsg` CLI
- **安装**: `brew install steipete/tap/imsg`
- **主要功能**: 发送消息、读取消息、搜索历史
- **示例**: `imsg send "+1234567890" "Hello!"`

#### discord
- **描述**: 通过 Discord API 管理消息
- **工程作用**: 扩展 Discord 渠道能力，提供消息管理和反应功能
- **运行时机**: 当用户在 Discord 渠道中请求发送消息、添加反应、编辑/删除消息时触发
- **依赖**: Discord 渠道配置
- **主要功能**: 发送消息、反应、管理频道
- **动作**: `sendMessage`, `react`, `editMessage`, `deleteMessage`

#### slack
- **描述**: 通过 Slack API 管理消息
- **工程作用**: 扩展 Slack 渠道能力，提供完整的消息和频道管理功能
- **运行时机**: 当用户在 Slack 渠道中请求发送/编辑/删除消息、添加反应、置顶消息时触发
- **依赖**: Slack 渠道配置
- **主要功能**: 发送/编辑/删除消息、反应、置顶、成员信息
- **动作组**: `reactions`, `messages`, `pins`, `memberInfo`, `emojiList`
- **示例**:
```json
{"action": "sendMessage", "to": "channel:C123", "content": "Hello"}
{"action": "react", "channelId": "C123", "messageId": "1712023032.1234", "emoji": "✅"}
```

---

### 🎬 媒体/音频

#### camsnap
- **描述**: 通过摄像头拍摄快照
- **工程作用**: 提供摄像头访问能力，用于拍照和视觉输入
- **运行时机**: 当用户请求拍照、获取摄像头画面时触发
- **平台**: macOS
- **依赖**: `camsnap` CLI
- **安装**: `brew install steipete/tap/camsnap`
- **示例**: `camsnap --output /tmp/photo.jpg`

#### gifgrep
- **描述**: 搜索和下载 GIF
- **工程作用**: 提供 GIF 搜索能力，用于在对话中插入动图
- **运行时机**: 当用户请求搜索 GIF、发送动图时触发
- **依赖**: `gifgrep` CLI
- **安装**: `brew install steipete/tap/gifgrep`
- **示例**: `gifgrep "happy dance" --limit 5`

#### openai-image-gen
- **描述**: 通过 OpenAI DALL-E 生成图像
- **工程作用**: 提供 AI 图像生成能力，根据文本描述创建图像
- **运行时机**: 当用户请求生成图像、创建插图、编辑图片时触发
- **依赖**: `OPENAI_API_KEY`
- **主要功能**: 生成图像、编辑图像
- **示例**: 使用 OpenAI API 生成图像

#### nano-banana-pro
- **描述**: 通过 Nano Banana Pro 设备进行音频处理
- **工程作用**: 集成 Nano Banana Pro 硬件设备，提供专业音频处理能力
- **运行时机**: 当用户使用 Nano Banana Pro 设备进行音频输入/输出时触发
- **依赖**: Nano Banana Pro 硬件
- **主要功能**: 音频输入/输出处理

#### openai-whisper
- **描述**: 本地 Whisper 模型语音转文字
- **工程作用**: 提供本地语音识别能力，无需网络即可转录音频
- **运行时机**: 当用户请求转录音频文件、语音转文字时触发（本地处理）
- **依赖**: `whisper` CLI
- **安装**: `pip install openai-whisper`
- **示例**: `whisper audio.mp3 --model base`

#### openai-whisper-api
- **描述**: 通过 OpenAI Whisper API 语音转文字
- **工程作用**: 提供云端语音识别能力，处理速度快、准确率高
- **运行时机**: 当用户请求转录音频文件、语音转文字时触发（云端处理）
- **依赖**: `OPENAI_API_KEY`
- **示例**: 使用 OpenAI API 转录音频

#### sag (Screen Audio Grab)
- **描述**: 录制屏幕和系统音频
- **工程作用**: 提供屏幕录制和系统音频捕获能力
- **运行时机**: 当用户请求录制屏幕、捕获系统音频时触发
- **平台**: macOS
- **依赖**: `sag` CLI
- **安装**: `brew install steipete/tap/sag`
- **示例**: `sag record --duration 30 --output recording.mp4`

#### sherpa-onnx-tts
- **描述**: 本地 TTS 语音合成
- **工程作用**: 提供本地文字转语音能力，无需网络即可生成语音
- **运行时机**: 当用户请求将文字转为语音、朗读内容时触发
- **依赖**: `sherpa-onnx` 模型
- **主要功能**: 文字转语音

#### video-frames
- **描述**: 通过 ffmpeg 提取视频帧
- **工程作用**: 提供视频帧提取能力，用于视频分析和缩略图生成
- **运行时机**: 当用户请求从视频中提取帧、生成缩略图时触发
- **依赖**: `ffmpeg`
- **安装**: `brew install ffmpeg`
- **示例**:
```bash
{baseDir}/scripts/frame.sh /path/to/video.mp4 --out /tmp/frame.jpg
{baseDir}/scripts/frame.sh /path/to/video.mp4 --time 00:00:10 --out /tmp/frame-10s.jpg
```

#### peekaboo
- **描述**: 屏幕截图和窗口捕获
- **工程作用**: 提供屏幕截图能力，用于视觉输入和问题诊断
- **运行时机**: 当用户请求截图、捕获窗口画面时触发
- **平台**: macOS
- **依赖**: `peekaboo` CLI
- **安装**: `brew install steipete/tap/peekaboo`
- **示例**: `peekaboo capture --output screenshot.png`

---

### 🏠 智能家居

#### openhue
- **描述**: 控制 Philips Hue 灯光
- **工程作用**: 集成 Philips Hue 智能灯光系统，实现灯光自动化控制
- **运行时机**: 当用户请求开关灯、调节亮度、设置颜色、创建场景时触发
- **依赖**: `openhue` CLI
- **安装**: `brew install openhue/tap/openhue`
- **主要功能**: 开关灯、调节亮度、设置颜色
- **示例**: `openhue light on "Living Room"`, `openhue light brightness 50`

#### eightctl
- **描述**: 控制 Eight Sleep 智能床垫
- **工程作用**: 集成 Eight Sleep 智能床垫，实现睡眠温度控制
- **运行时机**: 当用户请求查看床垫状态、设置温度、控制加热时触发
- **依赖**: `eightctl` CLI
- **安装**: `go install github.com/steipete/eightctl/cmd/eightctl@latest`
- **主要功能**: 查看状态、设置温度、控制加热
- **示例**: `eightctl status`, `eightctl set-temp 72`

#### blucli
- **描述**: 蓝牙设备控制
- **工程作用**: 提供蓝牙设备管理能力，实现设备连接和状态查询
- **运行时机**: 当用户请求列出蓝牙设备、连接/断开设备、查看设备状态时触发
- **平台**: macOS
- **依赖**: `blucli` CLI
- **主要功能**: 列出设备、连接/断开、查看状态
- **示例**: `blucli list`, `blucli connect "AirPods"`

#### sonoscli
- **描述**: 控制 Sonos 音箱
- **工程作用**: 集成 Sonos 多房间音响系统，实现音乐播放和音箱控制
- **运行时机**: 当用户请求播放/暂停音乐、调节音量、分组音箱、搜索音乐时触发
- **依赖**: `sonos` CLI (Go)
- **安装**: `go install github.com/steipete/sonoscli/cmd/sonos@latest`
- **主要功能**: 发现设备、播放控制、音量调节、分组
- **示例**:
```bash
sonos discover
sonos play --name "Kitchen"
sonos volume set 15 --name "Kitchen"
sonos smapi search --service "Spotify" --category tracks "query"
```
- **故障排除**:
  - `no route to host`: 需要在 macOS 隐私设置中启用本地网络访问
  - `bind: operation not permitted`: 可能在沙箱环境中运行

---

### 🛠️ 开发工具

#### coding-agent
- **描述**: 编码代理，辅助代码编写和调试
- **工程作用**: 提供代码生成、审查和调试能力，辅助开发工作
- **运行时机**: 当用户请求编写代码、审查代码、调试问题时触发
- **主要功能**: 代码生成、代码审查、调试辅助

#### github
- **描述**: 通过 GitHub CLI 管理仓库
- **工程作用**: 集成 GitHub 平台，实现仓库、PR、Issues 的完整管理
- **运行时机**: 当用户请求管理 GitHub 仓库、创建 PR、查看 Issues、管理 Actions 时触发
- **依赖**: `gh` CLI
- **安装**: `brew install gh`
- **主要功能**: 管理 PR、Issues、Actions、Releases
- **示例**: `gh pr list`, `gh issue create`

#### gh-issues
- **描述**: GitHub Issues 专用管理
- **工程作用**: 专注于 GitHub Issues 的管理，提供更细粒度的 Issue 操作
- **运行时机**: 当用户请求创建、搜索、更新 GitHub Issues 时触发
- **依赖**: `gh` CLI
- **主要功能**: 创建/搜索/更新 Issues
- **示例**: `gh issue list --label bug`

#### skill-creator
- **描述**: 创建新的 OpenClaw 技能
- **工程作用**: 辅助开发者创建新技能，生成标准化的技能模板
- **运行时机**: 当用户请求创建新技能、生成技能模板时触发
- **主要功能**: 生成技能模板、配置依赖

#### tmux
- **描述**: 远程控制 tmux 会话
- **工程作用**: 提供 tmux 会话控制能力，用于管理长时间运行的进程和交互式终端
- **运行时机**: 当用户请求监控 tmux 会话、发送按键、捕获输出时触发
- **平台**: macOS/Linux
- **依赖**: `tmux`
- **主要功能**: 发送按键、捕获输出、管理会话
- **使用场景**:
  - ✅ 监控 Claude/Codex 会话
  - ✅ 向交互式终端发送输入
  - ✅ 抓取长时间运行进程的输出
  - ❌ 运行一次性命令（使用 `exec`）
- **示例**:
```bash
tmux capture-pane -t shared -p | tail -20
tmux send-keys -t shared "y" Enter
tmux send-keys -t shared C-c
```

#### mcporter
- **描述**: MCP 服务器移植工具
- **工程作用**: 辅助移植和配置 MCP (Model Context Protocol) 服务器
- **运行时机**: 当用户请求移植 MCP 服务器、配置 MCP 连接时触发
- **主要功能**: 移植和配置 MCP 服务器

---

### 🤖 AI/LLM

#### gemini
- **描述**: 通过 Google Gemini API 进行对话
- **工程作用**: 提供 Google Gemini 模型访问能力，支持多模态理解
- **运行时机**: 当用户请求使用 Gemini 模型进行对话、分析图像时触发
- **依赖**: `GEMINI_API_KEY`
- **主要功能**: 文本生成、多模态理解

#### oracle
- **描述**: 通用 LLM 查询工具
- **工程作用**: 提供统一的 LLM 查询接口，支持多种模型提供商
- **运行时机**: 当用户请求向 LLM 提问、获取 AI 回答时触发
- **主要功能**: 向配置的 LLM 提问

#### model-usage
- **描述**: 查看模型使用统计
- **工程作用**: 提供模型使用量和成本统计，帮助用户监控 API 消耗
- **运行时机**: 当用户请求查看 token 使用量、API 成本统计时触发
- **主要功能**: 查看 token 使用量、成本统计

---

### 🔧 实用工具

#### blogwatcher
- **描述**: 监控博客更新
- **工程作用**: 提供博客订阅和更新监控能力，实现信息聚合
- **运行时机**: 当用户请求订阅博客、检查博客更新时触发
- **依赖**: `blogwatcher` CLI
- **主要功能**: 订阅博客、检查更新

#### clawhub
- **描述**: ClawHub 技能市场客户端
- **工程作用**: 提供技能市场访问能力，实现技能的搜索、安装和发布
- **运行时机**: 当用户请求搜索、安装、发布技能时触发
- **主要功能**: 搜索/安装/发布技能
- **示例**: `openclaw skills search weather`

#### goplaces
- **描述**: 地点搜索和导航
- **工程作用**: 提供地点搜索和路线规划能力
- **运行时机**: 当用户请求搜索地点、获取导航路线时触发
- **依赖**: `goplaces` CLI
- **主要功能**: 搜索地点、获取路线

#### healthcheck
- **描述**: 系统健康检查
- **工程作用**: 提供系统和服务健康状态检查能力
- **运行时机**: 当用户请求检查系统状态、验证服务可用性时触发
- **主要功能**: 检查服务状态、依赖可用性

#### nano-pdf
- **描述**: PDF 处理工具
- **工程作用**: 提供 PDF 文件处理能力，支持文本提取和文件操作
- **运行时机**: 当用户请求提取 PDF 文本、合并/拆分 PDF 时触发
- **依赖**: `nano-pdf` CLI
- **主要功能**: 提取文本、合并/拆分 PDF

#### node-connect
- **描述**: Node 设备连接管理
- **工程作用**: 管理 OpenClaw Node 设备的连接状态
- **运行时机**: 当用户请求连接/断开 Node 设备时触发
- **主要功能**: 连接/断开 Node 设备

#### ordercli
- **描述**: 订单管理 CLI
- **工程作用**: 提供订单查询和管理能力
- **运行时机**: 当用户请求查看订单、管理订单状态时触发
- **主要功能**: 查看/管理订单

#### session-logs
- **描述**: 会话日志查看
- **工程作用**: 提供 Agent 会话历史查看能力，用于调试和审计
- **运行时机**: 当用户请求查看会话历史、调试 Agent 行为时触发
- **主要功能**: 查看 Agent 会话历史
- **路径**: `~/.openclaw/agents/<agentId>/sessions/*.jsonl`

#### weather
- **描述**: 天气查询
- **工程作用**: 提供天气信息查询能力，无需 API Key
- **运行时机**: 当用户询问天气、温度、预报时触发
- **依赖**: `curl`
- **主要功能**: 当前天气、天气预报
- **使用场景**:
  - ✅ 查询天气、温度、预报
  - ❌ 历史天气、严重天气警报
- **示例**:
```bash
curl "wttr.in/London?format=3"           # 一行摘要
curl "wttr.in/London"                     # 3天预报
curl "wttr.in/London?format=j1"           # JSON 输出
```

#### xurl
- **描述**: X (Twitter) API CLI 工具
- **工程作用**: 提供 X (Twitter) 平台完整访问能力，支持发推、搜索、互动等
- **运行时机**: 当用户请求发推、回复、搜索推文、管理关注、发送 DM 时触发
- **依赖**: `xurl` CLI
- **安装**: `brew install xdevplatform/tap/xurl` 或 `npm install -g @xdevplatform/xurl`
- **主要功能**: 发推、回复、搜索、关注、DM、上传媒体
- **安全注意**: 
  - 永远不要在 Agent 会话中使用 `--verbose`
  - 永远不要读取或发送 `~/.xurl` 文件
- **示例**:
```bash
xurl auth oauth2                          # 认证
xurl post "Hello world!"                  # 发推
xurl reply POST_ID "Nice post!"           # 回复
xurl search "golang" -n 10                # 搜索
xurl timeline -n 20                       # 时间线
xurl dm @user "message"                   # 私信
```

#### summarize
- **描述**: URL/文件/视频摘要工具
- **工程作用**: 提供内容摘要能力，支持 URL、本地文件、YouTube 视频
- **运行时机**: 当用户请求摘要链接、总结文章、转录视频时触发
- **触发短语**: "summarize this", "what's this link about", "transcribe this video"
- **依赖**: `summarize` CLI
- **安装**: `brew install steipete/tap/summarize`
- **主要功能**: 摘要 URL、本地文件、YouTube 视频
- **示例**:
```bash
summarize "https://example.com" --model google/gemini-3-flash-preview
summarize "https://youtu.be/xxx" --youtube auto --extract-only
```
- **配置**: `~/.summarize/config.json`

---

### 🎵 音乐

#### songsee
- **描述**: 音频频谱图和特征可视化
- **工程作用**: 提供音频分析和可视化能力，生成频谱图和特征面板
- **运行时机**: 当用户请求分析音频、生成频谱图、可视化音频特征时触发
- **依赖**: `songsee` CLI
- **安装**: `brew install steipete/tap/songsee`
- **主要功能**: 生成频谱图、多面板可视化
- **示例**:
```bash
songsee track.mp3
songsee track.mp3 --viz spectrogram,mel,chroma,hpss
songsee track.mp3 --start 12.5 --duration 8 -o slice.jpg
```

#### spotify-player
- **描述**: Spotify 终端播放控制
- **工程作用**: 提供 Spotify 播放控制能力，实现音乐搜索和播放
- **运行时机**: 当用户请求播放/暂停音乐、搜索歌曲、切换设备时触发
- **依赖**: `spogo` (首选) 或 `spotify_player`
- **安装**: `brew install steipete/tap/spogo` 或 `brew install spotify_player`
- **要求**: Spotify Premium 账户
- **示例**:
```bash
spogo auth import --browser chrome        # 导入认证
spogo search track "query"                # 搜索
spogo play|pause|next|prev                # 播放控制
spogo device list                         # 设备列表
```

---

### 📞 语音

#### voice-call
- **描述**: 语音通话功能
- **工程作用**: 提供语音通话能力，支持多种电话服务提供商
- **运行时机**: 当用户请求发起语音通话、查询通话状态时触发
- **依赖**: voice-call 插件启用
- **支持提供商**: Twilio, Telnyx, Plivo, Mock
- **动作**: `initiate_call`, `continue_call`, `speak_to_user`, `end_call`, `get_status`
- **CLI**: `openclaw voicecall call --to "+15555550123" --message "Hello"`

---

### 📱 WhatsApp

#### wacli
- **描述**: WhatsApp CLI 工具
- **工程作用**: 提供 WhatsApp 消息发送和历史搜索能力（用于向第三方发送消息，非用户聊天）
- **运行时机**: 仅当用户明确要求向他人发送 WhatsApp 消息或同步/搜索历史时触发
- **注意**: 不要用于正常用户聊天，OpenClaw 会自动路由 WhatsApp 对话
- **依赖**: `wacli` CLI
- **安装**: `brew install steipete/tap/wacli` 或 `go install github.com/steipete/wacli/cmd/wacli@latest`
- **示例**:
```bash
wacli auth                                # QR 登录
wacli chats list --limit 20               # 列出聊天
wacli messages search "query"             # 搜索消息
wacli send text --to "+14155551212" --message "Hello!"
```

---

### 🎨 画布

#### canvas
- **描述**: 画布操作工具
- **工程作用**: 提供画布创建和编辑能力，用于可视化内容创作
- **运行时机**: 当用户请求创建画布、添加元素、编辑画布内容时触发
- **主要功能**: 创建/编辑画布、添加元素

---

## 技能结构

每个技能目录包含：

```
skills/<skill-name>/
├── SKILL.md          # 技能描述（必需）
├── scripts/          # 辅助脚本（可选）
├── references/       # 参考文档（可选）
└── assets/           # 资源文件（可选）
```

### SKILL.md 格式

```yaml
---
name: skill-name
description: 技能描述
homepage: https://example.com
metadata:
  openclaw:
    emoji: "🔧"
    os: ["darwin", "linux"]           # 可选，平台限制
    requires:
      bins: ["cli-tool"]              # 必需的 CLI 工具
      anyBins: ["tool1", "tool2"]     # 任一工具即可
      env: ["API_KEY"]                # 必需的环境变量
      config: ["config.path"]         # 必需的配置
    install:
      - id: brew
        kind: brew
        formula: tap/formula
        bins: ["cli-tool"]
        label: "Install via brew"
      - id: go
        kind: go
        module: github.com/user/repo@latest
        bins: ["cli-tool"]
        label: "Install via go"
---

# 技能名称

技能描述和使用说明...
```

## 安装方式

技能依赖的安装方式：

| 方式 | 示例 |
|------|------|
| brew | `brew install formula` |
| go | `go install module@latest` |
| npm | `npm install -g package` |
| pip | `pip install package` |
| uv | `uv tool install package` |
| 手动 | 下载二进制文件 |

## 常用命令

```bash
# 列出所有技能
openclaw skills list

# 查看技能状态
openclaw skills status

# 安装技能
openclaw skills install <skill>

# 启用/禁用技能
openclaw skills enable <skill>
openclaw skills disable <skill>

# 搜索技能（ClawHub）
openclaw skills search <query>
```

## 配置

```typescript
{
  skills: {
    bundled: {
      allowlist: ["github", "weather"],  // 允许的技能
      denylist: ["deprecated-skill"],    // 禁用的技能
    },
    managed: {
      enabled: ["clawhub-skill"],
    },
    workspace: {
      enabled: true,
      path: "~/.openclaw/workspace/skills",
    },
  }
}
```

## 技能加载优先级

```
workspace > managed > extension > bundled
```

工作区技能优先级最高，可覆盖同名的托管、扩展或内置技能。

---

## 扩展技能详细说明

### 📄 飞书集成 (extensions/feishu)

#### feishu-doc
- **描述**: 飞书文档读写操作
- **工程作用**: 提供飞书云文档的完整 CRUD 能力，支持 Markdown 内容写入和表格创建
- **运行时机**: 当用户提及飞书文档、云文档、docx 链接时触发
- **路径**: `extensions/feishu/skills/feishu-doc/`
- **主要动作**:
  - `read`: 读取文档内容
  - `write`: 替换整个文档（支持 Markdown）
  - `append`: 追加内容
  - `create`: 创建新文档
  - `list_blocks`: 获取完整块数据（含表格、图片）
  - `create_table`: 创建表格
  - `upload_image`: 上传图片
  - `upload_file`: 上传附件
- **Token 提取**: 从 URL `https://xxx.feishu.cn/docx/ABC123def` 提取 `doc_token` = `ABC123def`
- **配置**: `channels.feishu.tools.doc: true`
- **权限**: `docx:document`, `docx:document:readonly`, `drive:drive`

#### feishu-drive
- **描述**: 飞书云盘文件管理
- **工程作用**: 提供飞书云空间的文件和文件夹管理能力
- **运行时机**: 当用户提及云空间、文件夹、云盘时触发
- **路径**: `extensions/feishu/skills/feishu-drive/`
- **主要动作**:
  - `list`: 列出文件夹内容
  - `info`: 获取文件信息
  - `create_folder`: 创建文件夹
  - `move`: 移动文件
  - `delete`: 删除文件
- **文件类型**: `doc`, `docx`, `sheet`, `bitable`, `folder`, `file`, `mindnote`, `shortcut`
- **配置**: `channels.feishu.tools.drive: true`
- **权限**: `drive:drive` (完整访问) 或 `drive:drive:readonly` (只读)
- **注意**: Bot 没有根文件夹，只能访问已共享的文件/文件夹

#### feishu-wiki
- **描述**: 飞书知识库导航
- **工程作用**: 提供飞书知识库的空间和节点管理能力
- **运行时机**: 当用户提及知识库、wiki、wiki 链接时触发
- **路径**: `extensions/feishu/skills/feishu-wiki/`
- **主要动作**:
  - `spaces`: 列出知识空间
  - `nodes`: 列出节点
  - `get`: 获取节点详情
  - `create`: 创建节点
  - `move`: 移动节点
  - `rename`: 重命名节点
- **Wiki-Doc 工作流**: 
  1. 获取节点: `{ "action": "get", "token": "wiki_token" }` → 返回 `obj_token`
  2. 读取文档: `feishu_doc { "action": "read", "doc_token": "obj_token" }`
  3. 写入文档: `feishu_doc { "action": "write", "doc_token": "obj_token", "content": "..." }`
- **配置**: `channels.feishu.tools.wiki: true`
- **依赖**: 需要同时启用 `feishu_doc`
- **权限**: `wiki:wiki` 或 `wiki:wiki:readonly`

#### feishu-perm
- **描述**: 飞书权限管理
- **工程作用**: 提供飞书文档和文件的权限管理能力
- **运行时机**: 当用户提及分享、权限、协作者时触发
- **路径**: `extensions/feishu/skills/feishu-perm/`
- **主要动作**:
  - `list`: 列出协作者
  - `add`: 添加协作者
  - `remove`: 移除协作者
- **成员类型**: `email`, `openid`, `userid`, `unionid`, `openchat`, `opendepartmentid`
- **权限级别**: `view` (只读), `edit` (可编辑), `full_access` (完全访问)
- **配置**: `channels.feishu.tools.perm: true` (默认禁用)
- **权限**: `drive:permission`
- **注意**: 默认禁用，因为权限管理是敏感操作

---

### 🔀 Diff 工具 (extensions/diffs)

#### diffs
- **描述**: 生成可分享的 diff 视图
- **工程作用**: 提供真实的、可分享的 diff 展示能力，替代手动编辑摘要
- **运行时机**: 当需要展示代码变更、生成 diff 视图时触发
- **路径**: `extensions/diffs/skills/diffs/`
- **输入**: `before` + `after` 文本，或统一的 `patch` 字符串
- **模式**:
  - `mode=view`: 生成交互式 Gateway 托管的查看器 URL
  - `mode=file`: 生成渲染的文件工件（PNG/PDF）
  - `mode=both`: 同时生成 URL 和文件
- **选项**: `fileFormat` (png/pdf), `fileQuality` (standard/hq/print), `fileScale`, `fileMaxWidth`
- **使用**: 生成后可通过 `canvas present` 或 `canvas navigate` 展示

---

### 🪶 OpenProse (extensions/open-prose)

#### prose
- **描述**: OpenProse VM 多代理工作流编排
- **工程作用**: 提供 AI 会话编程语言，用于编排多代理工作流
- **运行时机**: 当用户使用 `prose` 命令、运行 `.prose` 文件、提及 OpenProse 时触发
- **路径**: `extensions/open-prose/skills/prose/`
- **命令路由**:
  - `prose help`: 加载帮助文档
  - `prose run <file>`: 加载 VM 并执行程序
  - `prose compile <file>`: 验证程序
  - `prose update`: 运行迁移
  - `prose examples`: 显示或运行示例
- **状态模式**:
  - `filesystem` (默认): 文件系统状态，支持恢复和调试
  - `in-context`: 上下文内状态，适合简单程序
  - `sqlite` (实验性): SQLite 状态
  - `postgres` (实验性): PostgreSQL 状态
- **OpenClaw 运行时映射**:
  - Task tool → `sessions_spawn`
  - File I/O → `read`/`write`
  - Remote fetch → `web_fetch`
- **示例**: 37 个示例程序在 `examples/` 目录

---

### 🔌 ACP 路由 (extensions/acpx)

#### acp-router
- **描述**: ACP 代理路由器
- **工程作用**: 将自然语言请求路由到 Pi/Claude Code/Codex/OpenCode/Gemini CLI 或 ACP 运行时
- **运行时机**: 当用户请求在 Pi/Claude Code/Codex/OpenCode/Gemini 中运行任务时触发
- **路径**: `extensions/acpx/skills/acp-router/`
- **AgentId 映射**:
  - "pi" → `agentId: "pi"`
  - "claude" / "claude code" → `agentId: "claude"`
  - "codex" → `agentId: "codex"`
  - "opencode" → `agentId: "opencode"`
  - "gemini" / "gemini cli" → `agentId: "gemini"`
  - "kimi" / "kimi cli" → `agentId: "kimi"`
- **路径选择**:
  1. **OpenClaw ACP 运行时路径** (默认): 使用 `sessions_spawn` / ACP 运行时工具
  2. **直接 acpx 路径** ("电话游戏"): 通过 `exec` 使用 `acpx` CLI 直接驱动
- **示例**:
```json
{
  "task": "Say hi.",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

---

### 🦞 Lobster (extensions/lobster)

#### lobster
- **描述**: 多步骤工作流执行与审批
- **工程作用**: 执行带有审批检查点的多步骤工作流
- **运行时机**: 当用户需要可重复的自动化、需要人工审批的操作、多工具调用作为一个确定性操作时触发
- **路径**: `extensions/lobster/`
- **使用场景**:
  - ✅ "整理我的邮件" — 多步骤，可能发送回复
  - ✅ "每天早上检查邮件，回复前询问" — 带审批的定时工作流
  - ✅ "监控这个 PR 并通知我变更" — 有状态、循环
  - ❌ "发送消息" — 单一操作，直接使用 message 工具
  - ❌ "天气怎么样？" — 简单查询
- **基本用法**:
```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' --max 20 | email.triage"
}
```
- **审批处理**: 如果工作流需要审批，返回 `requiresApproval` 对象，使用 `resume` 动作继续
- **关键特性**:
  - 确定性: 相同输入 → 相同输出
  - 审批门: `approve` 命令暂停执行
  - 可恢复: 使用 token 继续执行
  - 结构化输出: 始终返回带 `protocolVersion` 的 JSON 信封

---

## Agent 技能详细说明

### 🖥️ Parallels 测试 (.agents/skills)

#### parallels-discord-roundtrip
- **描述**: macOS Parallels 烟雾测试与 Discord 端到端验证
- **工程作用**: 验证 macOS Parallels 环境下的 Discord 双向消息传递
- **运行时机**: 当需要运行 macOS Parallels 烟雾测试并验证 Discord 端到端通信时触发
- **路径**: `.agents/skills/parallels-discord-roundtrip/`
- **测试覆盖**:
  - 在新鲜 macOS 快照上安装
  - 引导 + 网关健康检查
  - 客户端 `message send` 到 Discord
  - 主机在 Discord 上看到消息
  - 主机发布新 Discord 消息
  - 客户端 `message read` 看到新消息
- **运行命令**:
```bash
pnpm test:parallels:macos \
  --discord-token-env OPENCLAW_PARALLELS_DISCORD_TOKEN \
  --discord-guild-id <guild_id> \
  --discord-channel-id <channel_id> \
  --json
```
- **通过标准**:
  - fresh lane 或 upgrade lane 通过
  - 摘要报告 `discord=pass`
  - 客户端出站 nonce 出现在频道历史中
  - 主机入站 nonce 出现在 `openclaw message read` 输出中
