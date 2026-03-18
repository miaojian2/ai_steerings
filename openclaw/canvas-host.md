---
inclusion: manual
---

# Canvas Host 画布服务

## 概述

Canvas Host 是 OpenClaw 的本地 Web 服务器，用于托管交互式画布内容。支持实时重载、A2UI（App-to-UI）通信协议，主要用于移动端和桌面端的 WebView 交互。

## 目录结构

```
src/canvas-host/
├── server.ts           # HTTP 服务器实现
├── a2ui.ts             # A2UI 协议处理
├── file-resolver.ts    # 文件路径解析
├── a2ui/
│   └── index.html      # A2UI 默认页面
```

## 核心功能

### Canvas Host Server

```typescript
type CanvasHostServer = {
  port: number;
  rootDir: string;
  close: () => Promise<void>;
};

// 启动服务器
const server = await startCanvasHost({
  runtime: runtimeEnv,
  rootDir: "~/.openclaw/canvas",
  port: 0,              // 0 = 自动分配
  listenHost: "127.0.0.1",
  liveReload: true,
});
```

### Canvas Host Handler

可以将 handler 集成到现有 HTTP 服务器：

```typescript
const handler = await createCanvasHostHandler({
  runtime: runtimeEnv,
  rootDir: "/path/to/canvas",
  basePath: "/canvas",
  liveReload: true,
});

// 处理 HTTP 请求
const handled = await handler.handleHttpRequest(req, res);

// 处理 WebSocket 升级（实时重载）
const upgraded = handler.handleUpgrade(req, socket, head);
```

## A2UI 协议

A2UI（App-to-UI）是移动端/桌面端原生应用与 WebView 之间的通信协议。

### 发送用户动作

```javascript
// 在 WebView 中
window.openclawSendUserAction({
  name: "photo",
  surfaceId: "main",
  sourceComponentId: "demo.photo",
  context: { t: Date.now() },
});
```

### 接收动作状态

```javascript
window.addEventListener("openclaw:a2ui-action-status", (ev) => {
  const { id, ok, error } = ev.detail;
  console.log(`Action ${id}: ${ok ? "success" : error}`);
});
```

### 平台桥接

- iOS: `window.webkit.messageHandlers.openclawCanvasA2UIAction`
- Android: `window.openclawCanvasA2UIAction.postMessage()`

## 实时重载

Canvas Host 支持文件变更时自动重载：

1. 使用 chokidar 监听文件变更
2. 通过 WebSocket 通知客户端
3. 客户端收到 "reload" 消息后刷新页面

### WebSocket 路径

- `/canvas-ws` - 实时重载 WebSocket 端点

## 文件服务

### 默认根目录

```
~/.openclaw/canvas/
```

### 自动生成 index.html

如果根目录没有 `index.html`，会自动生成一个测试页面。

### MIME 类型检测

使用 `src/media/mime.ts` 自动检测文件 MIME 类型。

## 配置选项

```typescript
type CanvasHostOpts = {
  runtime: RuntimeEnv;
  rootDir?: string;       // 画布根目录
  port?: number;          // 监听端口（0 = 自动）
  listenHost?: string;    // 监听地址（默认 127.0.0.1）
  allowInTests?: boolean; // 测试环境是否启用
  liveReload?: boolean;   // 是否启用实时重载
};
```

## 环境变量

- `OPENCLAW_SKIP_CANVAS_HOST` - 设为 truthy 值禁用 Canvas Host

## 安全考虑

- 默认只监听 localhost
- 文件访问限制在 rootDir 内
- 路径遍历保护

## 相关模块

- `src/gateway/` - Gateway 集成 Canvas Host
- `apps/ios/` - iOS 应用 WebView 集成
- `apps/android/` - Android 应用 WebView 集成
- `apps/macos/` - macOS 应用 WebView 集成
