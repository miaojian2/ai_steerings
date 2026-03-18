---
inclusion: manual
---

# Media Understanding 媒体理解

## 概述

Media Understanding 模块处理音频转录、视频描述和图像描述，支持多个提供商，为 Agent 提供多模态理解能力。

## 目录结构

```
src/media-understanding/
├── types.ts                    # 类型定义
├── runtime.ts                  # 运行时 API
├── runner.ts                   # 主运行器
├── apply.ts                    # 应用到上下文
├── resolve.ts                  # 配置解析
├── scope.ts                    # 作用域控制
├── defaults.ts                 # 默认配置
├── format.ts                   # 输出格式化
├── attachments.ts              # 附件处理
├── attachments.cache.ts        # 附件缓存
├── attachments.normalize.ts    # 附件规范化
├── attachments.select.ts       # 附件选择
├── transcribe-audio.ts         # 音频转录
├── audio-preflight.ts          # 音频预检
├── audio-transcription-runner.ts
├── video.ts                    # 视频处理
├── echo-transcript.ts          # 回显转录
├── concurrency.ts              # 并发控制
├── errors.ts                   # 错误处理
├── fs.ts                       # 文件系统操作
├── output-extract.ts           # 输出提取
├── providers/                  # 提供商实现
│   ├── index.ts                # 提供商注册
│   ├── image.ts                # 图像描述
│   ├── image-runtime.ts        # 图像运行时
│   ├── shared.ts               # 共享工具
│   ├── openai-compatible-audio.ts
│   ├── openai/                 # OpenAI Whisper
│   ├── google/                 # Google
│   ├── groq/                   # Groq Whisper
│   ├── deepgram/               # Deepgram
│   ├── mistral/                # Mistral
│   └── moonshot/               # Moonshot
```

## 核心类型

### 媒体能力

```typescript
type MediaUnderstandingCapability = "image" | "audio" | "video";
type MediaUnderstandingKind =
  | "audio.transcription"
  | "video.description"
  | "image.description";
```

### 媒体附件

```typescript
type MediaAttachment = {
  path?: string;
  url?: string;
  mime?: string;
  index: number;
  alreadyTranscribed?: boolean;
};
```

### 理解输出

```typescript
type MediaUnderstandingOutput = {
  kind: MediaUnderstandingKind;
  attachmentIndex: number;
  text: string;
  provider: string;
  model?: string;
};
```

### 提供商接口

```typescript
type MediaUnderstandingProvider = {
  id: string;
  capabilities?: MediaUnderstandingCapability[];
  transcribeAudio?: (req: AudioTranscriptionRequest) => Promise<AudioTranscriptionResult>;
  describeVideo?: (req: VideoDescriptionRequest) => Promise<VideoDescriptionResult>;
  describeImage?: (req: ImageDescriptionRequest) => Promise<ImageDescriptionResult>;
  describeImages?: (req: ImagesDescriptionRequest) => Promise<ImagesDescriptionResult>;
};
```

## 音频转录

```typescript
type AudioTranscriptionRequest = {
  buffer: Buffer;
  fileName: string;
  mime?: string;
  apiKey: string;
  baseUrl?: string;
  headers?: Record<string, string>;
  model?: string;
  language?: string;
  prompt?: string;
  query?: Record<string, string | number | boolean>;
  timeoutMs: number;
  fetchFn?: typeof fetch;
};

type AudioTranscriptionResult = {
  text: string;
  model?: string;
};
```

## 图像描述

```typescript
type ImageDescriptionRequest = {
  buffer: Buffer;
  fileName: string;
  mime?: string;
  prompt?: string;
  maxTokens?: number;
  timeoutMs: number;
  profile?: string;
  preferredProfile?: string;
  agentDir: string;
  cfg: OpenClawConfig;
  model: string;
  provider: string;
};
```

## 视频描述

```typescript
type VideoDescriptionRequest = {
  buffer: Buffer;
  fileName: string;
  mime?: string;
  apiKey: string;
  baseUrl?: string;
  headers?: Record<string, string>;
  model?: string;
  prompt?: string;
  timeoutMs: number;
  fetchFn?: typeof fetch;
};
```

## 决策追踪

```typescript
type MediaUnderstandingDecision = {
  capability: MediaUnderstandingCapability;
  outcome: MediaUnderstandingDecisionOutcome;
  attachments: MediaUnderstandingAttachmentDecision[];
};

type MediaUnderstandingDecisionOutcome =
  | "success"
  | "skipped"
  | "disabled"
  | "no-attachment"
  | "scope-deny";
```

## 作用域控制

```typescript
// 在 scope.ts 中
function resolveMediaUnderstandingScope(params: {
  scope?: MediaUnderstandingScope;
  sessionKey?: string;
  channel?: string;
  chatType?: "dm" | "group" | "channel";
}): "allow" | "deny";
```

## 配置示例

```yaml
tools:
  media:
    audio:
      enabled: true
      provider: openai
      model: whisper-1
      language: auto
    image:
      enabled: true
      provider: openai
      model: gpt-4o
    video:
      enabled: true
      provider: google
    scope:
      channels:
        - whatsapp
        - telegram
```

## 支持的提供商

### 音频转录

- OpenAI Whisper
- Groq Whisper
- Deepgram
- Google
- Moonshot

### 图像描述

- OpenAI GPT-4V
- Google Gemini
- Mistral

### 视频描述

- Google Gemini

## 相关模块

- `src/agents/` - Agent 多模态输入处理
- `src/media/` - 媒体文件处理
- `src/providers/` - 认证管理
