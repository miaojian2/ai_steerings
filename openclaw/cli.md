---
inclusion: manual
---

# OpenClaw CLI 模块指南

## 概述

CLI 模块提供 OpenClaw 的命令行界面，包括所有用户交互命令、参数解析、输出格式化等。它是用户与 OpenClaw 交互的主要入口。

## 架构

```
┌─────────────────────────────────────────────────────────────┐
│                      CLI Entry                              │
│                    (openclaw.mjs)                           │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Program (Commander)                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ gateway │ │  agent  │ │ models  │ │channels │   ...    │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/cli/program.ts` | 主程序入口 |
| `src/cli/run-main.ts` | 运行入口 |
| `src/cli/gateway-cli.ts` | gateway 命令 |
| `src/cli/models-cli.ts` | models 命令 |
| `src/cli/channels-cli.ts` | channels 命令 |
| `src/cli/browser-cli.ts` | browser 命令 |
| `src/cli/skills-cli.ts` | skills 命令 |
| `src/cli/nodes-cli.ts` | nodes 命令 |
| `src/cli/config-cli.ts` | config 命令 |
| `src/cli/progress.ts` | 进度条/Spinner |
| `src/cli/prompt.ts` | 交互式提示 |

## 命令结构

### 主要命令

```bash
openclaw <command> [options]

Commands:
  gateway     # 网关管理
  agent       # Agent 交互
  models      # 模型管理
  channels    # 渠道管理
  message     # 消息发送/读取
  browser     # 浏览器控制
  skills      # 技能管理
  nodes       # 节点管理
  config      # 配置管理
  onboard     # 引导设置
  doctor      # 诊断修复
  update      # 更新
```

### 常用命令示例

```bash
# 网关
openclaw gateway run --port 18789
openclaw gateway status --deep
openclaw gateway restart

# Agent
openclaw agent --message "Hello"
openclaw agent --model openai/gpt-5.4 --thinking high

# 模型
openclaw models list
openclaw models auth login --provider openai
openclaw models auth status

# 渠道
openclaw channels status
openclaw channels login

# 消息
openclaw message send --to "+1234567890" --message "Hi"
openclaw message read --channel telegram

# 配置
openclaw config get agent.model
openclaw config set agent.model anthropic/claude-sonnet-4-6

# 技能
openclaw skills list
openclaw skills install <skill>
openclaw skills status
```

## 核心组件

### 1. Program 定义

```typescript
import { Command } from "commander";

const program = new Command()
  .name("openclaw")
  .description("Personal AI assistant")
  .version(version);

// 添加子命令
program.addCommand(gatewayCommand);
program.addCommand(agentCommand);
// ...
```

### 2. 命令选项

```typescript
// 通用选项模式
import { addCommonOptions } from "./command-options.js";

const cmd = new Command("example")
  .option("-v, --verbose", "Verbose output")
  .option("--json", "JSON output")
  .option("--config <path>", "Config file path");

addCommonOptions(cmd);
```

### 3. 进度显示

```typescript
import { createProgress, createSpinner } from "./progress.js";

// Spinner
const spin = createSpinner();
spin.start("Loading...");
// ... 操作
spin.stop("Done");

// 进度条
const progress = createProgress({ total: 100 });
progress.update(50);
progress.finish();
```

### 4. 交互式提示

```typescript
import { text, select, confirm } from "./prompt.js";

const name = await text({ message: "Enter name:" });
const choice = await select({
  message: "Select option:",
  options: [
    { value: "a", label: "Option A" },
    { value: "b", label: "Option B" },
  ],
});
const ok = await confirm({ message: "Continue?" });
```

### 5. 输出格式化

```typescript
import { formatTable } from "../terminal/table.js";

// 表格输出
console.log(formatTable({
  headers: ["Name", "Status"],
  rows: [
    ["Gateway", "Running"],
    ["Telegram", "Connected"],
  ],
}));

// JSON 输出
if (opts.json) {
  console.log(JSON.stringify(data, null, 2));
}
```

## 关键文件说明

| 文件 | 职责 |
|------|------|
| `program.ts` | Commander 程序定义 |
| `run-main.ts` | 入口和错误处理 |
| `deps.ts` | 依赖注入 |
| `progress.ts` | 进度条和 Spinner |
| `prompt.ts` | 交互式提示 |
| `banner.ts` | 启动横幅 |
| `cli-utils.ts` | 通用工具函数 |
| `command-options.ts` | 通用选项定义 |
| `gateway-rpc.ts` | Gateway RPC 客户端 |

## 开发新命令

```typescript
// src/cli/example-cli.ts
import { Command } from "commander";
import { createDefaultDeps } from "./deps.js";

export function createExampleCommand() {
  return new Command("example")
    .description("Example command")
    .option("-n, --name <name>", "Name option")
    .action(async (opts) => {
      const deps = createDefaultDeps();
      // 实现逻辑
      deps.runtime.log(`Hello, ${opts.name}`);
    });
}

// 在 program.ts 中注册
program.addCommand(createExampleCommand());
```

## 测试

```bash
# CLI 测试
pnpm test -- src/cli/

# 特定命令测试
pnpm test -- src/cli/gateway-cli.test.ts

# E2E 测试
pnpm test:e2e
```

## 调试技巧

```bash
# 查看帮助
openclaw --help
openclaw gateway --help

# 详细输出
openclaw gateway status --verbose

# JSON 输出（便于解析）
openclaw channels status --json | jq

# 调试模式
DEBUG=openclaw:* openclaw gateway run
```
