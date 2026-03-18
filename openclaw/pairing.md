---
inclusion: manual
---

# 配对系统 (src/pairing/)

## 概述

配对系统管理用户与 OpenClaw 的配对流程，包括配对码生成、验证和允许列表管理。

## 目录结构

```
src/pairing/
├── pairing-store.ts       # 配对存储核心
├── pairing-challenge.ts   # 配对挑战
├── pairing-messages.ts    # 配对消息
├── pairing-labels.ts      # 配对标签
├── setup-code.ts          # 设置码生成
└── *.test.ts              # 测试文件
```

## 核心功能

### 配对存储 (pairing-store.ts)

```typescript
import {
  upsertChannelPairingRequest,
  approveChannelPairingCode,
  listChannelPairingRequests,
  readChannelAllowFromStore,
  addChannelAllowFromStoreEntry
} from "./pairing-store.js";

// 创建配对请求
const { code, created } = await upsertChannelPairingRequest({
  channel: "telegram",
  id: "user-123",
  accountId: "bot-456",
  meta: { name: "John" }
});
// code: "ABCD1234"

// 批准配对码
const result = await approveChannelPairingCode({
  channel: "telegram",
  code: "ABCD1234",
  accountId: "bot-456"
});

// 列出待处理的配对请求
const requests = await listChannelPairingRequests("telegram");

// 读取允许列表
const allowList = await readChannelAllowFromStore("telegram", process.env, "bot-456");

// 添加到允许列表
await addChannelAllowFromStoreEntry({
  channel: "telegram",
  entry: "user-123",
  accountId: "bot-456"
});
```

### 配对请求结构

```typescript
type PairingRequest = {
  id: string;           // 用户 ID
  code: string;         // 配对码
  createdAt: string;    // 创建时间
  lastSeenAt: string;   // 最后活动时间
  meta?: Record<string, string>;  // 元数据
};
```

### 配对码特性

- 8 字符长度
- 使用无歧义字符集（排除 0O1I）
- 1 小时有效期
- 最多 3 个待处理请求

## 存储位置

```
~/.openclaw/credentials/
├── telegram-pairing.json      # 配对请求
├── telegram-allowFrom.json    # 允许列表（默认账户）
└── telegram-bot456-allowFrom.json  # 允许列表（特定账户）
```

## 配对流程

1. 用户发送消息给 Bot
2. 系统检查 DM 策略
3. 如果是 `pairing` 策略，生成配对码
4. 用户在控制端输入配对码
5. 系统验证并添加到允许列表
6. 后续消息直接处理

## 配对挑战 (pairing-challenge.ts)

```typescript
import { createPairingChallenge, verifyPairingChallenge } from "./pairing-challenge.js";

// 创建挑战
const challenge = createPairingChallenge();

// 验证挑战
const valid = verifyPairingChallenge(challenge, response);
```

## 配对消息 (pairing-messages.ts)

```typescript
import { formatPairingMessage } from "./pairing-messages.js";

// 格式化配对提示消息
const message = formatPairingMessage({
  code: "ABCD1234",
  channel: "telegram"
});
```

## 设置码 (setup-code.ts)

```typescript
import { generateSetupCode, validateSetupCode } from "./setup-code.js";

// 生成设置码
const code = generateSetupCode();

// 验证设置码
const valid = validateSetupCode(code);
```

## 账户隔离

- 每个账户有独立的允许列表
- 默认账户兼容旧版存储
- 非默认账户完全隔离

```typescript
// 读取特定账户的允许列表
const allowList = await readChannelAllowFromStore(
  "telegram",
  process.env,
  "bot-456"  // 账户 ID
);
```

## 缓存机制

允许列表读取使用文件状态缓存：

```typescript
// 缓存基于文件的 mtime 和 size
// 文件未变化时直接返回缓存结果
```

## 阅读建议

1. 从 `pairing-store.ts` 了解核心存储逻辑
2. 阅读 `pairing-challenge.ts` 理解挑战机制
3. 查看 `pairing-messages.ts` 了解消息格式
4. 研究测试文件了解各种场景
