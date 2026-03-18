---
inclusion: manual
---

# 语音合成 TTS (src/tts/)

## 概述

TTS 模块提供文本转语音功能，支持多个提供商（OpenAI、ElevenLabs、Microsoft Edge）。

## 目录结构

```
src/tts/
├── tts.ts              # 主入口和核心逻辑
├── tts-core.ts         # 核心工具函数
├── runtime.ts          # 运行时集成
├── provider-registry.ts # 提供商注册
├── provider-types.ts   # 提供商类型定义
└── providers/          # 具体提供商实现
    ├── openai.ts
    ├── elevenlabs.ts
    └── microsoft.ts
```

## 核心功能

### 文本转语音 (tts.ts)

```typescript
import { textToSpeech, resolveTtsConfig } from "./tts.js";

// 基本使用
const result = await textToSpeech({
  text: "Hello, world!",
  cfg: config,
  channel: "telegram"  // 可选，影响输出格式
});

if (result.success) {
  console.log(result.audioPath);  // 音频文件路径
}
```

### 配置解析

```typescript
import { resolveTtsConfig, resolveTtsPrefsPath } from "./tts.js";

// 解析 TTS 配置
const ttsConfig = resolveTtsConfig(cfg);

// 获取用户偏好路径
const prefsPath = resolveTtsPrefsPath(ttsConfig);
```

### 自动模式

```typescript
// TTS 自动模式
type TtsAutoMode = "off" | "always" | "inbound" | "tagged";

// 检查是否启用
const enabled = isTtsEnabled(config, prefsPath);

// 设置自动模式
setTtsAutoMode(prefsPath, "always");
```

## 提供商

### OpenAI

```typescript
// 配置
{
  "messages": {
    "tts": {
      "provider": "openai",
      "openai": {
        "apiKey": "$env:OPENAI_API_KEY",
        "model": "gpt-4o-mini-tts",
        "voice": "alloy",
        "speed": 1.0,
        "instructions": "Speak clearly and naturally"
      }
    }
  }
}
```

支持的声音：alloy, echo, fable, onyx, nova, shimmer

### ElevenLabs

```typescript
// 配置
{
  "messages": {
    "tts": {
      "provider": "elevenlabs",
      "elevenlabs": {
        "apiKey": "$env:ELEVENLABS_API_KEY",
        "voiceId": "pMsXgVXv3BLzUgSXRplE",
        "modelId": "eleven_multilingual_v2",
        "voiceSettings": {
          "stability": 0.5,
          "similarityBoost": 0.75
        }
      }
    }
  }
}
```

### Microsoft Edge (免费)

```typescript
// 配置
{
  "messages": {
    "tts": {
      "provider": "microsoft",
      "microsoft": {
        "voice": "en-US-MichelleNeural",
        "lang": "en-US",
        "rate": "+0%",
        "pitch": "+0Hz"
      }
    }
  }
}
```

## 指令覆盖

支持在文本中使用指令控制 TTS：

```
[[tts:voice=nova,speed=1.2]]
Hello, this will be spoken with Nova voice at 1.2x speed.
[[/tts]]
```

```typescript
import { parseTtsDirectives } from "./tts-core.js";

const result = parseTtsDirectives(text, modelOverrides, baseUrl);
// result.cleanedText - 清理后的文本
// result.overrides - 解析出的覆盖参数
```

## 自动应用到回复

```typescript
import { maybeApplyTtsToPayload } from "./tts.js";

// 自动为回复添加 TTS 音频
const payload = await maybeApplyTtsToPayload({
  payload: replyPayload,
  cfg: config,
  channel: "telegram",
  kind: "final",
  inboundAudio: true  // 入站消息是否包含音频
});
```

## 渠道适配

不同渠道使用不同的输出格式：

- Telegram/Feishu/WhatsApp: Opus 格式（语音气泡）
- 其他渠道: MP3 格式

```typescript
const VOICE_BUBBLE_CHANNELS = new Set(["telegram", "feishu", "whatsapp"]);
```

## 文本摘要

长文本自动摘要后再转语音：

```typescript
import { summarizeText } from "./tts-core.js";

const summary = await summarizeText({
  text: longText,
  targetLength: 1500,
  cfg: config,
  config: ttsConfig,
  timeoutMs: 30000
});
```

## 阅读建议

1. 从 `tts.ts` 了解主要 API
2. 阅读 `provider-registry.ts` 理解提供商注册
3. 查看 `providers/` 了解具体实现
4. 研究 `tts-core.ts` 了解工具函数
5. 查看测试文件了解使用场景
