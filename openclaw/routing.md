---
inclusion: manual
---

# 路由系统 (src/routing/)

## 概述

路由系统负责将入站消息路由到正确的 Agent，管理会话键生成和账户解析。

## 目录结构

```
src/routing/
├── resolve-route.ts       # 路由解析核心
├── session-key.ts         # 会话键生成
├── account-id.ts          # 账户 ID 处理
├── account-lookup.ts      # 账户查找
├── bindings.ts            # 绑定配置
└── *.test.ts              # 测试文件
```

## 核心功能

### 路由解析 (resolve-route.ts)

```typescript
import { resolveAgentRoute } from "./resolve-route.js";

// 解析入站消息的路由
const route = resolveAgentRoute({
  cfg: config,
  channel: "telegram",
  accountId: "bot-123",
  peer: { kind: "direct", id: "user-456" },
  guildId: null,
  teamId: null,
  memberRoleIds: []
});

// 返回结果
// {
//   agentId: "default",
//   channel: "telegram",
//   accountId: "bot-123",
//   sessionKey: "default:telegram:bot-123:user-456",
//   mainSessionKey: "default:main",
//   lastRoutePolicy: "session",
//   matchedBy: "binding.peer"
// }
```

### 路由匹配优先级

1. `binding.peer` - 精确匹配对话对象
2. `binding.peer.parent` - 匹配父级对话（线程）
3. `binding.guild+roles` - Guild + 角色匹配
4. `binding.guild` - Guild 匹配
5. `binding.team` - Team 匹配
6. `binding.account` - 账户匹配
7. `binding.channel` - 渠道匹配
8. `default` - 默认 Agent

### 会话键生成 (session-key.ts)

```typescript
import {
  buildAgentSessionKey,
  buildAgentMainSessionKey,
  normalizeAccountId
} from "./session-key.js";

// 构建会话键
const sessionKey = buildAgentSessionKey({
  agentId: "default",
  channel: "telegram",
  accountId: "bot-123",
  peer: { kind: "direct", id: "user-456" },
  dmScope: "per-peer"
});

// 构建主会话键
const mainKey = buildAgentMainSessionKey({
  agentId: "default",
  mainKey: "main"
});
```

### DM 作用域

```typescript
type DmScope = "main" | "per-peer" | "per-channel-peer" | "per-account-channel-peer";
```

- `main`: 所有 DM 共享一个会话
- `per-peer`: 每个对话对象一个会话
- `per-channel-peer`: 每个渠道+对话对象一个会话
- `per-account-channel-peer`: 每个账户+渠道+对话对象一个会话

## 绑定配置 (bindings.ts)

```json
{
  "bindings": [
    {
      "match": {
        "channel": "discord",
        "guildId": "123456789",
        "roles": ["admin"]
      },
      "agentId": "admin-agent"
    },
    {
      "match": {
        "channel": "telegram",
        "peer": { "kind": "direct", "id": "user-123" }
      },
      "agentId": "vip-agent"
    }
  ]
}
```

### 绑定匹配条件

- `channel`: 渠道 ID
- `accountId`: 账户 ID（支持 `*` 通配符）
- `peer`: 对话对象 `{ kind, id }`
- `guildId`: Discord Guild ID
- `teamId`: Slack/Teams Team ID
- `roles`: Discord 角色 ID 列表

## 账户处理

### 账户 ID 规范化 (account-id.ts)

```typescript
import { normalizeAccountId } from "./account-id.js";

const normalized = normalizeAccountId("Bot-123");  // "bot-123"
```

### 账户查找 (account-lookup.ts)

```typescript
import { lookupAccount } from "./account-lookup.js";

const account = lookupAccount(cfg, "telegram", "bot-123");
```

## 对话类型

```typescript
type ChatType = "direct" | "group" | "channel" | "thread";
```

- `direct`: 私聊
- `group`: 群组
- `channel`: 频道
- `thread`: 线程

## 缓存机制

路由解析使用多级缓存优化性能：

- `agentLookupCacheByCfg`: Agent 查找缓存
- `evaluatedBindingsCacheByCfg`: 绑定评估缓存
- `resolvedRouteCacheByCfg`: 路由结果缓存

## 阅读建议

1. 从 `resolve-route.ts` 了解路由逻辑
2. 阅读 `session-key.ts` 理解会话键生成
3. 查看 `bindings.ts` 了解绑定配置
4. 研究测试文件了解各种路由场景
