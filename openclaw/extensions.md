---
inclusion: manual
---

# OpenClaw Extensions 模块指南

## 概述

Extensions 目录包含所有内置扩展插件，包括渠道适配器、模型提供商、功能增强等。每个扩展都是独立的 npm workspace 包。

## 目录结构

```
extensions/
├── .npmignore
├── shared/              # 共享工具库
│
├── # 渠道扩展
├── telegram/
├── discord/
├── slack/
├── whatsapp/
├── signal/
├── imessage/
├── bluebubbles/
├── googlechat/
├── msteams/
├── matrix/
├── irc/
├── line/
├── feishu/
├── mattermost/
├── nextcloud-talk/
├── nostr/
├── synology-chat/
├── tlon/
├── twitch/
├── zalo/
├── zalouser/
│
├── # 模型提供商
├── openai/
├── anthropic/
├── google/
├── amazon-bedrock/
├── microsoft/           # Azure OpenAI
├── ollama/
├── openrouter/
├── together/
├── mistral/
├── xai/
├── nvidia/
├── huggingface/
├── qianfan/
├── volcengine/
├── byteplus/
├── minimax/
├── moonshot/
├── modelstudio/
├── kilocode/
├── kimi-coding/
├── venice/
├── vllm/
├── sglang/
│
├── # 功能扩展
├── brave/               # Brave 搜索
├── firecrawl/           # 网页抓取
├── elevenlabs/          # TTS
├── memory-core/         # 记忆核心
├── memory-lancedb/      # LanceDB 记忆
├── diagnostics-otel/    # OpenTelemetry
├── diffs/               # Diff 工具
├── llm-task/            # LLM 任务
├── open-prose/          # 写作助手
├── openshell/           # Shell 增强
├── phone-control/       # 手机控制
├── talk-voice/          # 语音对话
├── voice-call/          # 语音通话
├── thread-ownership/    # 线程所有权
├── device-pair/         # 设备配对
│
├── # 代理/网关
├── cloudflare-ai-gateway/
├── vercel-ai-gateway/
├── copilot-proxy/
├── github-copilot/
└── ...
```

## 扩展结构

每个扩展的标准结构：

```
extensions/example/
├── index.ts              # 插件入口
├── openclaw.plugin.json  # 插件元数据
├── package.json          # npm 包配置
├── *.ts                  # 实现文件
└── *.test.ts             # 测试文件
```

## 扩展类型

### 1. 渠道扩展

```typescript
// extensions/telegram/index.ts
import { definePluginEntry } from "openclaw/plugin-sdk/core";
import { createTelegramChannel } from "./channel.js";

export default definePluginEntry({
  id: "telegram",
  name: "Telegram Channel",
  register(api) {
    api.registerChannel(createTelegramChannel());
  },
});
```

### 2. Provider 扩展

```typescript
// extensions/openai/index.ts
import { definePluginEntry } from "openclaw/plugin-sdk/core";
import { buildOpenAIProvider } from "./openai-provider.js";

export default definePluginEntry({
  id: "openai",
  name: "OpenAI Provider",
  register(api) {
    api.registerProvider(buildOpenAIProvider());
    api.registerSpeechProvider(buildOpenAISpeechProvider());
    api.registerImageGenerationProvider(buildOpenAIImageGenerationProvider());
  },
});
```

### 3. 功能扩展

```typescript
// extensions/brave/index.ts
import { definePluginEntry } from "openclaw/plugin-sdk/core";

export default definePluginEntry({
  id: "brave",
  name: "Brave Search",
  register(api) {
    api.registerWebSearchProvider(braveSearchProvider);
  },
});
```

## 插件元数据

```json
// openclaw.plugin.json
{
  "id": "telegram",
  "name": "Telegram Channel",
  "version": "1.0.0",
  "channels": ["telegram"],
  "configSchema": {
    "type": "object",
    "properties": {
      "botToken": {
        "type": "string",
        "description": "Telegram Bot Token"
      }
    }
  }
}
```

## Package.json 配置

```json
{
  "name": "@openclaw/telegram",
  "version": "2026.3.14",
  "type": "module",
  "main": "index.ts",
  "dependencies": {
    "grammy": "^1.x"
  },
  "devDependencies": {
    "openclaw": "workspace:*"
  },
  "peerDependencies": {
    "openclaw": "*"
  }
}
```

## 主要扩展说明

### 渠道扩展

| 扩展 | 依赖库 | 说明 |
|------|--------|------|
| telegram | grammy | Telegram Bot API |
| discord | discord.js | Discord Bot |
| slack | @slack/bolt | Slack Socket Mode |
| whatsapp | @whiskeysockets/baileys | WhatsApp Web |
| signal | signal-cli | Signal REST API |
| matrix | matrix-js-sdk | Matrix 协议 |

### Provider 扩展

| 扩展 | 说明 |
|------|------|
| openai | OpenAI API + Codex OAuth |
| anthropic | Anthropic API + setup-token |
| google | Google AI (Gemini) |
| amazon-bedrock | AWS Bedrock |
| ollama | 本地 Ollama |
| openrouter | OpenRouter 聚合 |

### 功能扩展

| 扩展 | 说明 |
|------|------|
| brave | Brave 搜索 API |
| elevenlabs | ElevenLabs TTS |
| memory-lancedb | 向量记忆存储 |
| voice-call | 语音通话 |

## 开发新扩展

1. 创建目录：
```bash
mkdir extensions/my-extension
```

2. 创建 package.json：
```json
{
  "name": "@openclaw/my-extension",
  "version": "2026.3.14",
  "type": "module",
  "main": "index.ts",
  "devDependencies": {
    "openclaw": "workspace:*"
  }
}
```

3. 创建 openclaw.plugin.json
4. 实现 index.ts
5. 添加到 pnpm-workspace.yaml（已包含 `extensions/*`）

## 测试扩展

```bash
# 测试所有扩展
pnpm test:extensions

# 测试特定扩展
pnpm test -- extensions/telegram/

# 使用测试脚本
node scripts/test-extension.mjs telegram
```

## 发布扩展

内置扩展随 openclaw 主包发布。第三方扩展可以：

1. 发布到 npm（`@openclaw/` scope 需要权限）
2. 作为本地插件安装
3. 通过 ClawHub 分发

## 注意事项

- 扩展依赖放在自己的 package.json，不要加到根目录
- 使用 `openclaw` 作为 devDependencies 或 peerDependencies
- 避免使用 `workspace:*` 在 dependencies 中
- 测试文件放在扩展目录内
