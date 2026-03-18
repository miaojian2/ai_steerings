---
inclusion: manual
---

# OpenClaw Skills 模块指南

## 概述

Skills 是 OpenClaw 的技能系统，提供可扩展的工具集合。技能可以是内置的、托管的（ClawHub）或工作区本地的。每个技能包含工具定义、使用说明和配置。

## 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Skills System                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Skills Loader                           │   │
│  │  - 发现  - 加载  - 合并  - 注入到 Agent              │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│  │  Bundled  │    │  Managed  │    │ Workspace │          │
│  │  Skills   │    │  Skills   │    │  Skills   │          │
│  │ (skills/) │    │ (ClawHub) │    │ (~/.../   │          │
│  │           │    │           │    │  skills/) │          │
│  └───────────┘    └───────────┘    └───────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

### 内置技能 (skills/)

```
skills/
├── github/           # GitHub 操作
├── slack/            # Slack 操作
├── discord/          # Discord 操作
├── notion/           # Notion 操作
├── obsidian/         # Obsidian 笔记
├── trello/           # Trello 看板
├── weather/          # 天气查询
├── canvas/           # 画布操作
├── coding-agent/     # 编码代理
├── summarize/        # 摘要生成
├── openai-image-gen/ # 图像生成
├── openai-whisper/   # 语音转文字
├── spotify-player/   # Spotify 控制
├── voice-call/       # 语音通话
├── camsnap/          # 摄像头快照
├── peekaboo/         # 屏幕截图
└── ...
```

### 工作区技能

```
~/.openclaw/workspace/skills/
├── my-skill/
│   ├── SKILL.md      # 技能描述（必需）
│   ├── tools.md      # 工具说明
│   └── ...
```

## 技能结构

### SKILL.md（必需）

```markdown
# My Skill

这个技能用于...

## 工具

### tool_name
描述这个工具做什么...

**参数:**
- `param1`: 参数说明
- `param2`: 参数说明

**示例:**
\`\`\`
tool_name param1="value" param2="value"
\`\`\`
```

### tools.md（可选）

```markdown
# 工具详细说明

## tool_name

更详细的使用说明...
```

## 技能来源

### 1. Bundled Skills（内置）

位于 `skills/` 目录，随 OpenClaw 分发。

```typescript
// 配置允许列表
{
  skills: {
    bundled: {
      allowlist: ["github", "weather", "canvas"]
    }
  }
}
```

### 2. Managed Skills（托管）

从 ClawHub 安装的技能。

```bash
# 安装托管技能
openclaw skills install clawhub:my-skill

# 配置
{
  skills: {
    managed: {
      enabled: ["my-skill"]
    }
  }
}
```

### 3. Workspace Skills（工作区）

用户自定义技能，位于工作区目录。

```bash
# 创建工作区技能
mkdir -p ~/.openclaw/workspace/skills/my-skill
echo "# My Skill" > ~/.openclaw/workspace/skills/my-skill/SKILL.md
```

## 技能加载流程

```
1. 扫描技能目录
   - skills/ (bundled)
   - ~/.openclaw/managed-skills/ (managed)
   - ~/.openclaw/workspace/skills/ (workspace)

2. 解析 SKILL.md
   - 提取描述
   - 解析工具定义

3. 合并技能
   - workspace > managed > bundled (优先级)

4. 注入到 Agent
   - 添加到系统提示词
   - 注册工具
```

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/agents/skills.ts` | 技能加载和管理 |
| `src/agents/skills-install.ts` | 技能安装 |
| `src/agents/skills-status.ts` | 技能状态 |
| `src/cli/skills-cli.ts` | CLI 命令 |

## 配置

```typescript
{
  skills: {
    // 内置技能
    bundled: {
      allowlist: ["github", "weather"],  // 允许的技能
      denylist: ["deprecated-skill"],    // 禁用的技能
    },
    
    // 托管技能
    managed: {
      enabled: ["clawhub-skill"],
    },
    
    // 工作区技能
    workspace: {
      enabled: true,
      path: "~/.openclaw/workspace/skills",
    },
  }
}
```

## 常用命令

```bash
# 列出所有技能
openclaw skills list

# 查看技能状态
openclaw skills status

# 安装技能
openclaw skills install <skill>

# 卸载技能
openclaw skills uninstall <skill>

# 启用/禁用
openclaw skills enable <skill>
openclaw skills disable <skill>

# 搜索技能（ClawHub）
openclaw skills search <query>
```

## 创建自定义技能

1. 创建目录：
```bash
mkdir -p ~/.openclaw/workspace/skills/my-skill
```

2. 创建 SKILL.md：
```markdown
# My Custom Skill

这个技能帮助你...

## 使用方法

描述如何使用这个技能...

## 工具

### my_tool
执行某个操作...
```

3. 验证：
```bash
openclaw skills status
```

## 技能与工具的关系

- **技能 (Skill)**: 一组相关工具的集合 + 使用说明
- **工具 (Tool)**: 具体的可执行操作

技能通过 SKILL.md 描述工具的用途和用法，Agent 根据这些描述决定何时使用哪个工具。

## ClawHub

ClawHub 是 OpenClaw 的技能注册中心：

- 网址: https://clawhub.com
- 发布技能: 通过 ClawHub 网站
- 安装技能: `openclaw skills install clawhub:<skill>`

## 测试

```bash
# 技能相关测试
pnpm test -- src/agents/skills

# CLI 测试
pnpm test -- src/cli/skills-cli.test.ts
```
