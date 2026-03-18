---
inclusion: manual
---

# OpenClaw Gateway 模块指南

## 概述

Gateway 是 OpenClaw 的核心控制平面，通过 WebSocket 提供统一的服务接口，管理会话、渠道、工具、事件和配置。它是整个系统的心脏。

## 架构

```
                    HTTP/WS 请求
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Gateway Server                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  WebSocket  │  │    HTTP     │  │  Control UI │         │
│  │   Handler   │  │   Handler   │  │   (静态)    │         │
│  └──────┬──────┘  └──────┬──────┘  └─────────────┘         │
│         │                │                                  │
│         ▼                ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Server Methods (RPC)                    │   │
│  │  chat.*, sessions.*, config.*, nodes.*, cron.*...   │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│  │  Channels │    │   Agents  │    │   Nodes   │          │
│  │  Manager  │    │  Runtime  │    │  Registry │          │
│  └───────────┘    └───────────┘    └───────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/gateway/server.ts` | 服务器导出入口 |
| `src/gateway/server.impl.ts` | 服务器核心实现 |
| `src/gateway/server-methods.ts` | RPC 方法注册 |
| `src/gateway/server-methods/` | 各类 RPC 方法实现 |
| `src/gateway/protocol/` | WebSocket 协议定义 |
| `src/gateway/server/` | 服务器子模块 |
| `src/gateway/server-channels.ts` | 渠道管理器 |
| `src/gateway/server-chat.ts` | 聊天/Agent 事件处理 |
| `src/gateway/server-cron.ts` | 定时任务服务 |
| `src/gateway/server-http.ts` | HTTP 服务器 |
| `src/gateway/control-ui*.ts` | Control UI 相关 |
| `src/gateway/auth*.ts` | 认证相关 |

## 核心概念

### 1. Gateway Server

```typescript
type GatewayServer = {
  // 启动服务器
  start(): Promise<void>;
  // 停止服务器
  stop(): Promise<void>;
  // 广播事件
  broadcast(event: string, payload: unknown): void;
  // 获取运行状态
  getState(): GatewayRuntimeState;
};

// 启动网关
const server = await startGatewayServer({
  port: 18789,
  bind: "loopback",
  config: loadedConfig,
});
```

### 2. Server Methods (RPC)

Gateway 通过 WebSocket 暴露 RPC 方法：

```typescript
// 方法定义
type GatewayRequestHandler = (
  params: unknown,
  ctx: GatewayRequestContext
) => Promise<unknown>;

// 核心方法分类
const methods = {
  // 聊天相关
  "chat.send": handleChatSend,
  "chat.abort": handleChatAbort,
  
  // 会话相关
  "sessions.list": handleSessionsList,
  "sessions.patch": handleSessionsPatch,
  "sessions.reset": handleSessionsReset,
  
  // 配置相关
  "config.get": handleConfigGet,
  "config.patch": handleConfigPatch,
  
  // 节点相关
  "node.list": handleNodeList,
  "node.invoke": handleNodeInvoke,
  
  // 定时任务
  "cron.list": handleCronList,
  "cron.trigger": handleCronTrigger,
};
```

### 3. 事件系统

Gateway 广播各类事件：

```typescript
// 事件类型
const GATEWAY_EVENTS = {
  "agent.start": {},
  "agent.text": {},
  "agent.tool": {},
  "agent.end": {},
  "session.update": {},
  "channel.status": {},
  "node.connect": {},
  "node.disconnect": {},
  // ...
};
```

### 4. 认证模式

```typescript
type GatewayAuthMode = 
  | "none"           // 无认证（仅本地）
  | "password"       // 密码认证
  | "token"          // Token 认证
  | "tailscale";     // Tailscale 身份认证

// 配置示例
{
  gateway: {
    auth: {
      mode: "password",
      password: "xxx"
    }
  }
}
```

## 关键文件说明

| 文件 | 职责 |
|------|------|
| `server.impl.ts` | 服务器启动、初始化、生命周期管理 |
| `server-methods.ts` | RPC 方法路由和注册 |
| `server-channels.ts` | 渠道连接管理（启动/停止/状态） |
| `server-chat.ts` | Agent 事件处理和消息路由 |
| `server-cron.ts` | 定时任务调度 |
| `server-http.ts` | HTTP 服务器（健康检查、Webhook） |
| `control-ui.ts` | Control UI 静态资源服务 |
| `auth.ts` | 认证逻辑 |
| `client.ts` | Gateway 客户端（用于 CLI 连接） |

## 配置项

```typescript
{
  gateway: {
    port: 18789,              // 端口
    bind: "loopback",         // 绑定地址
    auth: {
      mode: "none",           // 认证模式
      password: "",           // 密码
      token: "",              // Token
    },
    tailscale: {
      mode: "off",            // off | serve | funnel
    },
    controlUi: {
      enabled: true,          // 是否启用 Control UI
    },
  }
}
```

## 常用命令

```bash
# 启动网关
pnpm openclaw gateway run --port 18789

# 开发模式（自动重载）
pnpm gateway:watch

# 查看状态
pnpm openclaw gateway status --deep

# 重启
pnpm openclaw gateway restart
```

## 调试技巧

```bash
# 查看网关日志
tail -f /tmp/openclaw-gateway.log

# macOS 统一日志
./scripts/clawlog.sh

# 检查端口
ss -ltnp | grep 18789

# 健康检查
curl http://localhost:18789/health
```

## 测试

```bash
# 网关测试
pnpm test:gateway

# 特定测试
pnpm test -- src/gateway/server.test.ts
```
