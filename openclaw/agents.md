---
inclusion: manual
---

# OpenClaw Agents 模块指南

## 概述

Agents 模块是 AI 代理的运行时核心，负责管理 AI 会话、工具执行、上下文构建、模型调用等。它基于 `pi-agent` 库构建，提供了完整的 Agent 生命周期管理。

## 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent Runtime                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Pi Embedded Runner                      │   │
│  │  - 会话管理  - 工具执行  - 流式输出  - 上下文构建   │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│  │   Tools   │    │  Models   │    │  Skills   │          │
│  │  (bash,   │    │  Config   │    │ (bundled, │          │
│  │  browser) │    │  + Auth   │    │  managed) │          │
│  └───────────┘    └───────────┘    └───────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/agents/pi-embedded-runner.ts` | 嵌入式 Agent 运行器入口 |
| `src/agents/pi-embedded-runner/` | 运行器子模块 |
| `src/agents/pi-embedded-subscribe.ts` | 流式订阅处理 |
| `src/agents/tools/` | 工具定义 |
| `src/agents/bash-tools*.ts` | Bash 工具实现 |
| `src/agents/skills*.ts` | 技能系统 |
| `src/agents/auth-profiles*.ts` | 认证配置管理 |
| `src/agents/models-config*.ts` | 模型配置 |
| `src/agents/sandbox*.ts` | 沙箱执行 |
| `src/agents/subagent*.ts` | 子代理系统 |
| `src/agents/workspace*.ts` | 工作区管理 |

## 核心概念

### 1. Pi Embedded Runner

Agent 的核心运行器：

```typescript
import { runEmbeddedPiAgent } from "./pi-embedded-runner/run.js";

const result = await runEmbeddedPiAgent({
  config,
  sessionKey: "main",
  message: "Hello",
  tools: [...],
  onBlockReply: (text) => { /* 流式输出 */ },
  onToolCall: (tool, args) => { /* 工具调用 */ },
});
```

### 2. 工具系统

```typescript
// 工具定义
type AnyAgentTool = {
  name: string;
  description: string;
  parameters: JSONSchema;
  execute: (args: unknown, ctx: ToolContext) => Promise<ToolResult>;
};

// 内置工具
const builtInTools = [
  "bash",           // Shell 命令执行
  "browser",        // 浏览器控制
  "read",           // 文件读取
  "write",          // 文件写入
  "edit",           // 文件编辑
  "sessions_list",  // 会话列表
  "sessions_send",  // 跨会话消息
  "cron_*",         // 定时任务
  "canvas_*",       // 画布操作
  "nodes_*",        // 节点操作
];
```

### 3. 技能系统

```typescript
// 技能目录结构
// ~/.openclaw/workspace/skills/<skill>/
//   ├── SKILL.md      # 技能描述
//   ├── tools.md      # 工具说明
//   └── ...

// 技能类型
type SkillSource = 
  | "bundled"    // 内置技能
  | "managed"    // 托管技能（ClawHub）
  | "workspace"; // 工作区技能
```

### 4. 认证配置

```typescript
// 认证存储
// ~/.openclaw/credentials/auth-profiles.json

type AuthProfileCredential = 
  | { type: "api_key"; apiKey: string; }
  | { type: "oauth"; access: string; refresh?: string; expires?: number; }
  | { type: "token"; token: string; };

// 使用
import { ensureAuthProfileStore, upsertAuthProfile } from "./auth-profiles.js";
```

### 5. 沙箱执行

```typescript
// 沙箱模式
type SandboxMode = 
  | "off"       // 禁用
  | "non-main"  // 非主会话启用
  | "all";      // 全部启用

// 配置
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        toolAllowlist: ["bash", "read", "write"],
      }
    }
  }
}
```

### 6. 子代理系统

```typescript
// 子代理生成
import { spawnSubagent } from "./subagent-spawn.js";

const subagent = await spawnSubagent({
  parentSessionKey: "main",
  task: "Research topic X",
  model: "anthropic/claude-sonnet-4-6",
});
```

## 关键文件说明

| 文件 | 职责 |
|------|------|
| `pi-embedded-runner/run.ts` | Agent 主运行循环 |
| `pi-embedded-subscribe.ts` | 流式事件订阅 |
| `bash-tools.exec.ts` | Bash 命令执行 |
| `bash-tools.process.ts` | 进程管理 |
| `skills.ts` | 技能加载和管理 |
| `auth-profiles.ts` | 认证配置存储 |
| `models-config.ts` | 模型配置解析 |
| `model-auth.ts` | 模型认证解析 |
| `sandbox.ts` | 沙箱上下文解析 |
| `workspace.ts` | 工作区文件加载 |
| `system-prompt.ts` | 系统提示词构建 |
| `context.ts` | 上下文构建 |
| `compaction.ts` | 上下文压缩 |

## 会话管理

```typescript
// 会话键格式
type SessionKey = 
  | "main"                           // 主会话
  | `telegram:${userId}`             // Telegram DM
  | `discord:${guildId}:${channelId}` // Discord 频道
  | `whatsapp:${jid}`;               // WhatsApp

// 会话目录
// ~/.openclaw/agents/<agentId>/sessions/<sessionKey>/
//   ├── session.jsonl    # 会话历史
//   └── ...
```

## 配置项

```typescript
{
  agents: {
    defaults: {
      model: "anthropic/claude-sonnet-4-6",
      workspace: "~/.openclaw/workspace",
      thinkingLevel: "low",
      sandbox: { mode: "off" },
    }
  },
  agent: {
    model: "...",  // 默认模型
  }
}
```

## 常用命令

```bash
# 运行 Agent
pnpm openclaw agent --message "Hello"

# 指定模型
pnpm openclaw agent --model openai/gpt-5.4 --message "Hello"

# 指定思考级别
pnpm openclaw agent --thinking high --message "Complex task"

# 本地模式（不通过 Gateway）
pnpm openclaw agent --local --message "Hello"
```

## 调试技巧

```bash
# 查看会话日志
cat ~/.openclaw/agents/default/sessions/main/session.jsonl | jq

# 查看技能状态
pnpm openclaw skills status

# 查看模型配置
pnpm openclaw models list
```

## 测试

```bash
# Agent 相关测试
pnpm test -- src/agents/

# 特定测试
pnpm test -- src/agents/pi-embedded-runner.test.ts
```
