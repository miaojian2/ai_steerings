---
inclusion: manual
---

# 会话管理 (src/sessions/)

## 概述

会话管理模块处理对话上下文、会话 ID 解析和消息历史。

## 目录结构

```
src/sessions/
├── session-id.ts          # 会话 ID 生成和解析
├── session-id-resolution.ts # 会话 ID 解析
├── session-key-utils.ts   # 会话键工具
├── session-label.ts       # 会话标签
├── transcript-events.ts   # 对话记录事件
├── model-overrides.ts     # 模型覆盖
├── send-policy.ts         # 发送策略
├── level-overrides.ts     # 级别覆盖
├── input-provenance.ts    # 输入来源
└── *.test.ts              # 测试文件
```

## 核心功能

### 会话 ID (session-id.ts)

```typescript
import { generateSessionId, parseSessionId } from "./session-id.js";

// 生成会话 ID
const sessionId = generateSessionId();

// 解析会话 ID
const parsed = parseSessionId(sessionId);
```

### 会话 ID 解析 (session-id-resolution.ts)

```typescript
import { resolveSessionId } from "./session-id-resolution.js";

// 从各种输入解析会话 ID
const sessionId = resolveSessionId({
  explicit: "session-123",
  channel: "telegram",
  peer: { kind: "direct", id: "user-456" }
});
```

### 会话键工具 (session-key-utils.ts)

```typescript
import { normalizeSessionKey, parseSessionKey } from "./session-key-utils.js";

// 规范化会话键
const normalized = normalizeSessionKey(rawKey);

// 解析会话键
const parts = parseSessionKey(sessionKey);
// { agentId, channel, accountId, peerId }
```

### 对话记录事件 (transcript-events.ts)

```typescript
import { TranscriptEvent, formatTranscriptEvent } from "./transcript-events.js";

// 对话事件类型
type TranscriptEvent = {
  type: "user" | "assistant" | "tool" | "system";
  content: string;
  timestamp: number;
  metadata?: Record<string, unknown>;
};

// 格式化事件
const formatted = formatTranscriptEvent(event);
```

### 模型覆盖 (model-overrides.ts)

```typescript
import { resolveModelOverrides } from "./model-overrides.js";

// 解析会话级别的模型覆盖
const overrides = resolveModelOverrides({
  sessionConfig: session.config,
  agentConfig: agent.config
});
```

### 发送策略 (send-policy.ts)

```typescript
import { resolveSendPolicy } from "./send-policy.js";

// 解析发送策略
const policy = resolveSendPolicy({
  channel: "telegram",
  peer: { kind: "group", id: "group-123" }
});
```

### 会话标签 (session-label.ts)

```typescript
import { formatSessionLabel } from "./session-label.js";

// 格式化会话标签（用于显示）
const label = formatSessionLabel(sessionKey);
// "telegram:user-123"
```

### 输入来源 (input-provenance.ts)

```typescript
import { InputProvenance } from "./input-provenance.js";

// 输入来源类型
type InputProvenance = {
  channel: string;
  accountId?: string;
  peerId?: string;
  messageId?: string;
};
```

## 会话存储

会话数据存储在 `~/.openclaw/sessions/` 目录：

```
~/.openclaw/sessions/
├── default/               # Agent ID
│   ├── main.jsonl        # 主会话
│   └── telegram-user123.jsonl  # 特定会话
└── agent-2/
    └── ...
```

## 会话配置

```json
{
  "session": {
    "dmScope": "per-peer",
    "identityLinks": {
      "telegram:user-123": ["discord:user-456"]
    }
  }
}
```

### DM 作用域

- `main`: 所有 DM 共享会话
- `per-peer`: 每个对话对象独立会话
- `per-channel-peer`: 每个渠道+对话对象独立会话
- `per-account-channel-peer`: 完全隔离

### 身份链接

允许跨渠道关联同一用户：

```json
{
  "identityLinks": {
    "telegram:user-123": ["discord:user-456", "slack:U789"]
  }
}
```

## 阅读建议

1. 从 `session-id.ts` 了解 ID 生成
2. 阅读 `session-key-utils.ts` 理解会话键
3. 查看 `transcript-events.ts` 了解对话记录
4. 研究 `src/config/sessions/` 了解存储实现
