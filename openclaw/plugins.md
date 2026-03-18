---
inclusion: manual
---

# OpenClaw Plugins 模块指南

## 概述

Plugins 模块是 OpenClaw 的扩展系统核心，负责插件的发现、加载、注册和生命周期管理。它支持渠道插件、Provider 插件、工具插件等多种类型。

## 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Plugin System                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Plugin Loader                           │   │
│  │  - 发现  - 加载  - 验证  - 注册                      │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│  │  Channel  │    │ Provider  │    │   Tool    │          │
│  │  Plugins  │    │  Plugins  │    │  Plugins  │          │
│  └───────────┘    └───────────┘    └───────────┘          │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Plugin Registry                         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/plugins/loader.ts` | 插件加载器 |
| `src/plugins/registry.ts` | 插件注册表 |
| `src/plugins/discovery.ts` | 插件发现 |
| `src/plugins/types.ts` | 类型定义 |
| `src/plugins/runtime.ts` | 运行时上下文 |
| `src/plugins/runtime/` | 运行时子模块 |
| `src/plugins/hooks.ts` | 钩子系统 |
| `src/plugins/providers.ts` | Provider 管理 |
| `src/plugins/commands.ts` | 命令注册 |
| `src/plugin-sdk/` | 插件开发 SDK |
| `extensions/` | 内置扩展插件 |

## 插件类型

### 1. Channel Plugin（渠道插件）

```typescript
type ChannelPlugin = {
  id: ChannelId;
  meta: ChannelMeta;
  start(ctx: ChannelStartContext): Promise<void>;
  stop(): Promise<void>;
  send(payload: ChannelSendPayload): Promise<ChannelSendResult>;
};
```

### 2. Provider Plugin（模型提供商插件）

```typescript
type ProviderPlugin = {
  id: string;
  label: string;
  auth: ProviderAuthMethod[];
  catalog?: ProviderPluginCatalog;
  resolveDynamicModel?: (ctx) => ProviderRuntimeModel | null;
  // ...
};
```

### 3. Tool Plugin（工具插件）

```typescript
type OpenClawPluginToolFactory = (ctx: OpenClawPluginToolContext) => 
  AnyAgentTool | AnyAgentTool[] | null;
```

### 4. Speech Provider（语音插件）

```typescript
type SpeechProviderPlugin = {
  id: SpeechProviderId;
  synthesize(req: SpeechSynthesisRequest): Promise<SpeechSynthesisResult>;
  listVoices?(req: SpeechListVoicesRequest): Promise<SpeechVoiceOption[]>;
};
```

## 插件定义

### 插件入口

```typescript
// extensions/example/index.ts
import { definePluginEntry } from "openclaw/plugin-sdk/core";

export default definePluginEntry({
  id: "example",
  name: "Example Plugin",
  description: "An example plugin",
  
  register(api) {
    // 注册 Provider
    api.registerProvider(myProvider);
    
    // 注册渠道
    api.registerChannel(myChannel);
    
    // 注册工具
    api.registerTool("my-tool", myToolFactory);
    
    // 注册钩子
    api.registerHook("before-agent-start", myHook);
  },
});
```

### 插件元数据

```json
// extensions/example/openclaw.plugin.json
{
  "id": "example",
  "name": "Example Plugin",
  "version": "1.0.0",
  "providers": ["example-provider"],
  "channels": ["example-channel"],
  "configSchema": {
    "type": "object",
    "properties": {
      "apiKey": { "type": "string" }
    }
  }
}
```

## 插件加载流程

```
1. Discovery（发现）
   - 扫描 extensions/ 目录
   - 扫描已安装的 npm 包
   - 读取 openclaw.plugin.json

2. Load（加载）
   - 使用 jiti 加载 TypeScript
   - 验证插件结构
   - 解析配置 schema

3. Register（注册）
   - 调用 register(api)
   - 注册各类组件到 Registry

4. Activate（激活）
   - 根据配置启用/禁用
   - 初始化运行时状态
```

## Plugin Registry

```typescript
type PluginRegistry = {
  plugins: Map<string, PluginRecord>;
  channels: ChannelPlugin[];
  providers: ProviderPlugin[];
  tools: Map<string, OpenClawPluginToolFactory>;
  hooks: Map<string, HookHandler[]>;
  // ...
};

// 获取注册表
import { getActivePluginRegistry } from "./runtime.js";
const registry = getActivePluginRegistry();
```

## Plugin Runtime

```typescript
// 创建运行时
import { createPluginRuntime } from "./runtime/index.js";

const runtime = createPluginRuntime({
  config,
  agentDir,
  workspaceDir,
});

// 运行时 API
runtime.config;      // 配置访问
runtime.agent;       // Agent 操作
runtime.channel;     // 渠道操作
runtime.media;       // 媒体处理
runtime.tools;       // 工具注册
runtime.events;      // 事件系统
runtime.logging;     // 日志
runtime.state;       // 状态存储
```

## 钩子系统

```typescript
// 可用钩子
type HookName = 
  | "before-agent-start"
  | "after-agent-end"
  | "before-tool-call"
  | "after-tool-call"
  | "on-inbound-message"
  | "on-outbound-reply"
  | "on-compaction"
  | "on-gateway-start"
  | "on-gateway-stop";

// 注册钩子
api.registerHook("before-agent-start", async (ctx) => {
  // 在 Agent 启动前执行
});
```

## 配置

```typescript
// 插件配置
{
  plugins: {
    enabled: ["example"],
    disabled: ["another"],
    config: {
      example: {
        apiKey: "xxx"
      }
    }
  }
}
```

## 关键文件说明

| 文件 | 职责 |
|------|------|
| `loader.ts` | 插件加载和初始化 |
| `registry.ts` | 插件注册表管理 |
| `discovery.ts` | 插件发现逻辑 |
| `types.ts` | 所有插件类型定义 |
| `runtime.ts` | 运行时上下文 |
| `hooks.ts` | 钩子系统实现 |
| `providers.ts` | Provider 插件管理 |
| `commands.ts` | 命令插件管理 |
| `manifest.ts` | 插件清单解析 |

## 开发插件

1. 创建目录结构：
```
extensions/my-plugin/
├── index.ts
├── openclaw.plugin.json
└── package.json
```

2. 实现插件入口
3. 定义配置 schema
4. 注册到 pnpm workspace

## 常用命令

```bash
# 查看插件状态
pnpm openclaw plugins list

# 安装插件
pnpm openclaw plugins install <plugin>

# 启用/禁用
pnpm openclaw plugins enable <plugin>
pnpm openclaw plugins disable <plugin>
```

## 测试

```bash
# 插件测试
pnpm test -- src/plugins/

# 扩展测试
pnpm test:extensions
```
