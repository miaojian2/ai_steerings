---
inclusion: manual
---

# 插件 SDK (src/plugin-sdk/)

## 概述

插件 SDK 提供扩展开发所需的所有运行时 API，是 `openclaw/plugin-sdk` 包的实现。

## 目录结构

```
src/plugin-sdk/
├── index.ts               # 主导出入口
├── core.ts                # 核心类型和工具
├── runtime.ts             # 运行时上下文
├── entrypoints.ts         # 入口点定义
├── *-runtime.ts           # 各子系统运行时
├── *.ts                   # 渠道/提供商特定导出
└── *.test.ts              # 测试文件
```

## 核心导出

### 运行时上下文 (runtime.ts)

```typescript
import { getPluginRuntime } from "openclaw/plugin-sdk";

const runtime = getPluginRuntime();

// 访问配置
const cfg = runtime.config;

// 访问日志
runtime.log.info("Plugin initialized");

// 访问存储
await runtime.store.set("key", value);
```

### 渠道开发

```typescript
// channel-lifecycle.ts - 渠道生命周期
import { ChannelLifecycle } from "openclaw/plugin-sdk";

// channel-runtime.ts - 渠道运行时
import { getChannelRuntime } from "openclaw/plugin-sdk";

// channel-send-result.ts - 发送结果
import { ChannelSendResult } from "openclaw/plugin-sdk";
```

### 提供商开发

```typescript
// provider-auth.ts - 认证
import { ProviderAuthResult } from "openclaw/plugin-sdk";

// provider-stream.ts - 流式响应
import { ProviderStream } from "openclaw/plugin-sdk";

// provider-models.ts - 模型定义
import { ProviderModel } from "openclaw/plugin-sdk";
```

## 子系统运行时

### Agent 运行时 (agent-runtime.ts)

```typescript
import { getAgentRuntime } from "openclaw/plugin-sdk";

const agent = getAgentRuntime();
await agent.sendMessage(target, payload);
```

### Gateway 运行时 (gateway-runtime.ts)

```typescript
import { getGatewayRuntime } from "openclaw/plugin-sdk";

const gateway = getGatewayRuntime();
const status = await gateway.getStatus();
```

### 媒体运行时 (media-runtime.ts)

```typescript
import { getMediaRuntime } from "openclaw/plugin-sdk";

const media = getMediaRuntime();
const saved = await media.saveBuffer(buffer, "image/png");
```

### 安全运行时 (security-runtime.ts)

```typescript
import { getSecurityRuntime } from "openclaw/plugin-sdk";

const security = getSecurityRuntime();
const audit = await security.runAudit();
```

## 渠道特定导出

每个渠道有对应的类型导出：

```typescript
// telegram.ts
import { TelegramConfig, TelegramMessage } from "openclaw/plugin-sdk/telegram";

// discord.ts
import { DiscordConfig, DiscordMessage } from "openclaw/plugin-sdk/discord";

// slack.ts
import { SlackConfig, SlackMessage } from "openclaw/plugin-sdk/slack";

// 等等...
```

## 工具函数

### JSON 存储 (json-store.ts)

```typescript
import { readJsonFileWithFallback, writeJsonFileAtomically } from "openclaw/plugin-sdk";

const { value, exists } = await readJsonFileWithFallback(path, defaultValue);
await writeJsonFileAtomically(path, data);
```

### 文本分块 (text-chunking.ts)

```typescript
import { chunkText } from "openclaw/plugin-sdk";

const chunks = chunkText(longText, { maxChars: 4000 });
```

### 回复处理 (reply-payload.ts)

```typescript
import { buildReplyPayload } from "openclaw/plugin-sdk";

const payload = buildReplyPayload({
  text: "Hello",
  mediaUrl: "/path/to/image.png"
});
```

### Webhook 工具 (webhook-*.ts)

```typescript
import { validateWebhookSignature } from "openclaw/plugin-sdk";

const valid = validateWebhookSignature(request, secret);
```

## 配置路径 (config-paths.ts)

```typescript
import { resolvePluginConfigPath, resolvePluginDataPath } from "openclaw/plugin-sdk";

const configPath = resolvePluginConfigPath("my-plugin");
const dataPath = resolvePluginDataPath("my-plugin");
```

## 状态路径 (state-paths.ts)

```typescript
import { resolvePluginStatePath } from "openclaw/plugin-sdk";

const statePath = resolvePluginStatePath("my-plugin", "cache.json");
```

## 测试工具 (testing.ts)

```typescript
import { createTestRuntime, mockChannelSend } from "openclaw/plugin-sdk/testing";

const runtime = createTestRuntime();
const sendMock = mockChannelSend();
```

## 导入方式

```typescript
// 主入口
import { ... } from "openclaw/plugin-sdk";

// 渠道特定
import { ... } from "openclaw/plugin-sdk/telegram";
import { ... } from "openclaw/plugin-sdk/discord";

// 测试工具
import { ... } from "openclaw/plugin-sdk/testing";
```

## 阅读建议

1. 从 `index.ts` 了解所有导出
2. 阅读 `runtime.ts` 理解运行时上下文
3. 查看 `channel-lifecycle.ts` 了解渠道开发
4. 研究 `provider-*.ts` 了解提供商开发
5. 查看具体渠道文件了解类型定义
