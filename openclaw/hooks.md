---
inclusion: manual
---

# Hooks 系统

## 概述

Hooks 是 OpenClaw 的事件驱动扩展系统，允许在特定事件发生时执行自定义逻辑。支持内置 hooks、npm 包 hooks、git 仓库 hooks 和工作区 hooks。

## 目录结构

```
src/hooks/
├── types.ts              # Hook 类型定义
├── hooks.ts              # Hook 导出和注册 API
├── internal-hooks.ts     # 内部 hook 实现
├── loader.ts             # Hook 加载器
├── module-loader.ts      # 模块加载
├── config.ts             # Hook 配置
├── frontmatter.ts        # HOOK.md frontmatter 解析
├── install.ts            # Hook 安装
├── workspace.ts          # 工作区 hook 管理
├── plugin-hooks.ts       # 插件 hook 集成
├── message-hooks.ts      # 消息相关 hooks
├── gmail*.ts             # Gmail 集成 hooks
├── bundled/              # 内置 hooks
│   ├── boot-md/          # 启动 markdown hook
│   ├── command-logger/   # 命令日志 hook
│   └── session-memory/   # 会话记忆 hook
```

## 核心类型

### Hook 定义

```typescript
type Hook = {
  name: string;
  description: string;
  source: "openclaw-bundled" | "openclaw-managed" | "openclaw-workspace" | "openclaw-plugin";
  pluginId?: string;
  filePath: string;      // HOOK.md 路径
  baseDir: string;       // hook 目录
  handlerPath: string;   // handler.ts/js 路径
};
```

### Hook 元数据

```typescript
type OpenClawHookMetadata = {
  always?: boolean;           // 是否始终启用
  hookKey?: string;           // hook 标识符
  emoji?: string;             // 显示图标
  homepage?: string;          // 主页链接
  events: string[];           // 处理的事件列表
  export?: string;            // 导出名称（默认 "default"）
  os?: string[];              // 支持的操作系统
  requires?: {
    bins?: string[];          // 必需的二进制文件
    anyBins?: string[];       // 任一二进制文件
    env?: string[];           // 必需的环境变量
    config?: string[];        // 必需的配置项
  };
  install?: HookInstallSpec[];
};
```

### Hook 安装规格

```typescript
type HookInstallSpec = {
  id?: string;
  kind: "bundled" | "npm" | "git";
  label?: string;
  package?: string;       // npm 包名
  repository?: string;    // git 仓库 URL
  bins?: string[];        // 提供的二进制文件
};
```

## Hook API

### 注册和触发

```typescript
import {
  registerHook,
  unregisterHook,
  clearHooks,
  triggerHook,
  createHookEvent,
  getRegisteredHookEventKeys,
} from "./hooks.js";

// 注册 hook
registerHook("session:start", async (event) => {
  console.log("Session started:", event.sessionId);
});

// 触发 hook
await triggerHook(createHookEvent("session:start", { sessionId: "xxx" }));
```

## 事件类型

常见的 hook 事件：

- `command:new` - 新命令执行
- `session:start` - 会话开始
- `session:end` - 会话结束
- `message:received` - 收到消息
- `message:sent` - 发送消息
- `agent:turn:start` - Agent 回合开始
- `agent:turn:end` - Agent 回合结束

## 内置 Hooks

### boot-md

启动时加载 markdown 文件到上下文。

### command-logger

记录命令执行日志。

### session-memory

会话记忆持久化。

## Hook 文件结构

每个 hook 需要：

1. `HOOK.md` - 描述文件（包含 frontmatter 元数据）
2. `handler.ts` 或 `handler.js` - 处理器实现

### HOOK.md 示例

```markdown
---
events:
  - session:start
  - session:end
requires:
  bins:
    - node
---

# My Custom Hook

This hook does something useful.
```

## 工作区 Hooks

工作区 hooks 位于项目的 `.openclaw/hooks/` 目录，自动加载。

## 相关模块

- `src/plugins/` - 插件系统（hooks 可以来自插件）
- `src/agents/` - Agent 运行时（触发 hook 事件）
- `src/sessions/` - 会话管理（会话相关 hooks）
