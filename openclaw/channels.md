---
inclusion: manual
---

# OpenClaw Channels 模块指南

## 概述

Channels 模块负责消息渠道的抽象和管理，让 OpenClaw 可以连接到各种消息平台（WhatsApp、Telegram、Discord、Slack 等）。它定义了统一的渠道接口，处理消息路由、会话映射和渠道特定的功能。

## 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Channel Manager                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Channel Registry                        │   │
│  │  - 渠道注册  - 状态管理  - 生命周期控制              │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐          │
│  │ Telegram  │    │  Discord  │    │  WhatsApp │   ...    │
│  │  Plugin   │    │  Plugin   │    │  Plugin   │          │
│  └───────────┘    └───────────┘    └───────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/channels/registry.ts` | 渠道注册表 |
| `src/channels/plugins/` | 渠道插件接口 |
| `src/channels/plugins/types.ts` | 渠道类型定义 |
| `src/channels/session.ts` | 会话管理 |
| `src/channels/allowlists/` | 允许列表管理 |
| `src/channels/transport/` | 消息传输层 |
| `src/channels/web/` | Web 渠道 |
| `extensions/<channel>/` | 各渠道具体实现 |

## 支持的渠道

| 渠道 | ID | 扩展位置 |
|------|-----|---------|
| Telegram | `telegram` | `extensions/telegram/` |
| WhatsApp | `whatsapp` | `extensions/whatsapp/` |
| Discord | `discord` | `extensions/discord/` |
| Slack | `slack` | `extensions/slack/` |
| Signal | `signal` | `extensions/signal/` |
| iMessage | `imessage` | `extensions/imessage/` |
| BlueBubbles | `bluebubbles` | `extensions/bluebubbles/` |
| Google Chat | `googlechat` | `extensions/googlechat/` |
| Microsoft Teams | `msteams` | `extensions/msteams/` |
| Matrix | `matrix` | `extensions/matrix/` |
| IRC | `irc` | `extensions/irc/` |
| LINE | `line` | `extensions/line/` |
| Feishu | `feishu` | `extensions/feishu/` |
| WebChat | `web` | `src/channels/web/` |

## 核心接口

### ChannelPlugin

```typescript
type ChannelPlugin = {
  id: ChannelId;
  meta: ChannelMeta;
  
  // 生命周期
  start(ctx: ChannelStartContext): Promise<void>;
  stop(): Promise<void>;
  
  // 消息发送
  send(payload: ChannelSendPayload): Promise<ChannelSendResult>;
  
  // 状态
  getStatus(): ChannelStatus;
  
  // 可选功能
  typing?: (target: string) => Promise<void>;
  react?: (messageId: string, emoji: string) => Promise<void>;
};
```

### ChannelMeta

```typescript
type ChannelMeta = {
  id: string;
  label: string;              // 显示名称
  selectionLabel: string;     // 选择器标签
  docsPath: string;           // 文档路径
  blurb: string;              // 简短描述
  systemImage: string;        // 图标
};
```

### 消息入站处理

```typescript
// 入站消息信封
type InboundEnvelope = {
  channel: ChannelId;
  senderId: string;
  senderName?: string;
  chatId: string;
  chatType: "dm" | "group";
  messageId: string;
  text: string;
  attachments?: Attachment[];
  replyToMessageId?: string;
};

// 处理流程
inboundMessage → allowlistCheck → sessionResolve → agentDispatch
```

## 会话映射

```typescript
// 会话键生成规则
function resolveSessionKey(envelope: InboundEnvelope): string {
  // DM: channel:senderId
  // Group: channel:chatId
  
  if (envelope.chatType === "dm") {
    return `${envelope.channel}:${envelope.senderId}`;
  }
  return `${envelope.channel}:${envelope.chatId}`;
}
```

## 允许列表

```typescript
// 配置
{
  channels: {
    telegram: {
      allowFrom: ["123456789", "+1234567890"],
      groups: {
        "-1001234567890": { requireMention: true }
      }
    }
  }
}

// DM 策略
type DmPolicy = 
  | "pairing"  // 需要配对码（默认）
  | "open";    // 开放访问
```

## 渠道配置示例

### Telegram

```typescript
{
  channels: {
    telegram: {
      botToken: "123456:ABCDEF",
      allowFrom: ["*"],  // 或具体用户 ID
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

### Discord

```typescript
{
  channels: {
    discord: {
      token: "xxx",
      guilds: {
        "123456789": {
          channels: ["987654321"],
          requireMention: true
        }
      }
    }
  }
}
```

### WhatsApp

```typescript
{
  channels: {
    whatsapp: {
      allowFrom: ["+1234567890"],
      groups: {
        "*": { requireMention: true }
      }
    }
  }
}
```

## 关键文件说明

| 文件 | 职责 |
|------|------|
| `registry.ts` | 渠道元数据注册表 |
| `plugins/types.ts` | 渠道插件接口定义 |
| `session.ts` | 会话键解析和管理 |
| `allow-from.ts` | 允许列表匹配 |
| `mention-gating.ts` | @提及检测 |
| `typing.ts` | 输入状态管理 |
| `targets.ts` | 发送目标解析 |
| `sender-label.ts` | 发送者标签格式化 |

## 常用命令

```bash
# 查看渠道状态
pnpm openclaw channels status

# 登录渠道（如 WhatsApp）
pnpm openclaw channels login

# 发送消息
pnpm openclaw message send --to "+1234567890" --message "Hello"

# 读取消息
pnpm openclaw message read --channel telegram
```

## 添加新渠道

1. 在 `extensions/` 下创建新目录
2. 实现 `ChannelPlugin` 接口
3. 创建 `openclaw.plugin.json` 元数据
4. 更新 `.github/labeler.yml`
5. 添加文档到 `docs/channels/`

## 测试

```bash
# 渠道测试
pnpm test:channels

# 特定渠道
pnpm test -- extensions/telegram/
```
