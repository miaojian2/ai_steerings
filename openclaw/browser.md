---
inclusion: manual
---

# 浏览器控制 (src/browser/)

## 概述

浏览器控制模块提供 Chrome/Chromium 浏览器的自动化控制能力，支持 CDP (Chrome DevTools Protocol) 和 Playwright。

## 目录结构

```
src/browser/
├── client.ts              # 客户端 API
├── client-actions.ts      # 浏览器操作
├── server.ts              # 控制服务器
├── server-context.ts      # 服务器上下文
├── cdp.ts                 # CDP 协议封装
├── chrome.ts              # Chrome 启动管理
├── chrome-mcp.ts          # MCP 集成
├── profiles.ts            # 浏览器配置文件
├── config.ts              # 配置解析
├── control-auth.ts        # 认证控制
├── pw-*.ts                # Playwright 相关
├── routes/                # HTTP 路由
└── *.test.ts              # 测试文件
```

## 核心功能

### 客户端 API (client.ts)

```typescript
import {
  browserStatus,
  browserStart,
  browserStop,
  browserTabs,
  browserSnapshot
} from "./client.js";

// 获取浏览器状态
const status = await browserStatus(baseUrl);

// 启动浏览器
await browserStart(baseUrl, { profile: "default" });

// 获取标签页列表
const tabs = await browserTabs(baseUrl);

// 获取页面快照
const snapshot = await browserSnapshot(baseUrl, {
  format: "ai",
  labels: true
});
```

### 浏览器操作 (client-actions.ts)

```typescript
import {
  browserNavigate,
  browserClick,
  browserType,
  browserScroll
} from "./client-actions.js";

// 导航
await browserNavigate(baseUrl, "https://example.com");

// 点击元素
await browserClick(baseUrl, { ref: "button-1" });

// 输入文本
await browserType(baseUrl, { ref: "input-1", text: "Hello" });

// 滚动
await browserScroll(baseUrl, { direction: "down" });
```

### 页面快照

支持两种快照格式：

```typescript
// ARIA 格式 - 结构化无障碍树
const ariaSnapshot = await browserSnapshot(baseUrl, { format: "aria" });

// AI 格式 - 优化的文本表示
const aiSnapshot = await browserSnapshot(baseUrl, {
  format: "ai",
  labels: true,      // 添加元素标签
  interactive: true, // 只包含可交互元素
  maxChars: 10000    // 限制字符数
});
```

### 配置文件管理 (profiles.ts)

```typescript
import {
  browserProfiles,
  browserCreateProfile,
  browserDeleteProfile
} from "./client.js";

// 列出配置文件
const profiles = await browserProfiles(baseUrl);

// 创建配置文件
await browserCreateProfile(baseUrl, {
  name: "work",
  color: "blue",
  userDataDir: "/path/to/profile"
});

// 删除配置文件
await browserDeleteProfile(baseUrl, "work");
```

## 服务器端

### 控制服务器 (server.ts)

```typescript
import { createBrowserControlServer } from "./server.js";

const server = createBrowserControlServer({
  port: 9222,
  auth: { token: "secret" }
});
```

### 路由 (routes/)

```
routes/
├── agent.ts           # Agent 操作路由
├── agent.act.ts       # 执行操作
├── agent.snapshot.ts  # 快照获取
├── basic.ts           # 基础操作
├── tabs.ts            # 标签页管理
└── dispatcher.ts      # 请求分发
```

## CDP 集成 (cdp.ts)

```typescript
import { connectCdp, sendCdpCommand } from "./cdp.js";

// 连接到 CDP
const client = await connectCdp(cdpUrl);

// 发送命令
const result = await sendCdpCommand(client, "Page.navigate", {
  url: "https://example.com"
});
```

## Playwright 集成 (pw-*.ts)

```typescript
// pw-session.ts - 会话管理
// pw-ai.ts - AI 辅助操作
// pw-tools-core.ts - 核心工具
// pw-role-snapshot.ts - 角色快照
```

## 配置

```json
{
  "browser": {
    "enabled": true,
    "cdpPort": 9222,
    "headless": false,
    "profiles": {
      "default": {
        "color": "blue",
        "userDataDir": "~/.openclaw/browser/default"
      }
    }
  }
}
```

## 安全考虑

### 认证 (control-auth.ts)

- 支持 token 认证
- 支持 Gateway 认证集成
- CSRF 保护

### 沙箱

- 支持 `--no-sandbox` 模式（Docker 环境）
- 用户数据目录隔离

## 阅读建议

1. 从 `client.ts` 了解客户端 API
2. 阅读 `server.ts` 理解服务器架构
3. 查看 `cdp.ts` 了解 CDP 协议
4. 研究 `routes/` 了解 HTTP API
5. 查看 `pw-*.ts` 了解 Playwright 集成
