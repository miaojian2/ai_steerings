---
inclusion: manual
---

# Image Generation 图像生成

## 概述

Image Generation 模块提供统一的图像生成接口，支持多个提供商（OpenAI、Google 等），具有自动故障转移和模型回退功能。

## 目录结构

```
src/image-generation/
├── types.ts              # 类型定义
├── runtime.ts            # 运行时 API
├── provider-registry.ts  # 提供商注册表
├── providers/            # 提供商实现
│   ├── openai.ts         # OpenAI DALL-E
│   └── google.ts         # Google Imagen
```

## 核心类型

### 生成请求

```typescript
type ImageGenerationRequest = {
  provider: string;
  model: string;
  prompt: string;
  cfg: OpenClawConfig;
  agentDir?: string;
  authStore?: AuthProfileStore;
  count?: number;                    // 生成数量
  size?: string;                     // 尺寸（如 "1024x1024"）
  resolution?: ImageGenerationResolution;  // "1K" | "2K" | "4K"
  inputImages?: ImageGenerationSourceImage[];  // 编辑模式输入图像
};
```

### 生成结果

```typescript
type ImageGenerationResult = {
  images: GeneratedImageAsset[];
  model?: string;
  metadata?: Record<string, unknown>;
};

type GeneratedImageAsset = {
  buffer: Buffer;
  mimeType: string;
  fileName?: string;
  revisedPrompt?: string;  // 模型修改后的提示词
  metadata?: Record<string, unknown>;
};
```

### 提供商接口

```typescript
type ImageGenerationProvider = {
  id: string;
  aliases?: string[];
  label?: string;
  defaultModel?: string;
  models?: string[];
  supportedSizes?: string[];
  supportedResolutions?: ImageGenerationResolution[];
  supportsImageEditing?: boolean;
  generateImage: (req: ImageGenerationRequest) => Promise<ImageGenerationResult>;
};
```

## 运行时 API

### 生成图像

```typescript
import { generateImage } from "./runtime.js";

const result = await generateImage({
  cfg: config,
  prompt: "A cute cat wearing a hat",
  modelOverride: "openai/dall-e-3",  // 可选
  count: 1,
  size: "1024x1024",
});

console.log(result.images[0].buffer);  // 图像数据
console.log(result.provider);          // 使用的提供商
console.log(result.model);             // 使用的模型
console.log(result.attempts);          // 尝试记录（用于调试）
```

### 列出提供商

```typescript
import { listRuntimeImageGenerationProviders } from "./runtime.js";

const providers = listRuntimeImageGenerationProviders({ config });
// [{ id: "openai", label: "OpenAI", models: ["dall-e-3", "dall-e-2"], ... }]
```

## 模型配置

在配置文件中设置默认图像生成模型：

```yaml
agents:
  defaults:
    imageGenerationModel:
      primary: openai/dall-e-3
      fallbacks:
        - google/imagen-3.0-generate-001
```

## 故障转移

运行时自动处理模型故障转移：

1. 尝试主模型
2. 如果失败，依次尝试 fallback 模型
3. 记录所有尝试结果
4. 如果全部失败，抛出聚合错误

```typescript
type FallbackAttempt = {
  provider: string;
  model: string;
  error?: string;
  reason?: FailoverReason;
  status?: number;
  code?: string;
};
```

## 支持的提供商

### OpenAI

- 模型：`dall-e-3`, `dall-e-2`
- 尺寸：`1024x1024`, `1792x1024`, `1024x1792`
- 支持图像编辑

### Google

- 模型：`imagen-3.0-generate-001`
- 分辨率：1K, 2K, 4K

## 图像编辑

部分提供商支持基于输入图像的编辑：

```typescript
const result = await generateImage({
  cfg: config,
  prompt: "Add a rainbow in the sky",
  inputImages: [{
    buffer: originalImageBuffer,
    mimeType: "image/png",
  }],
});
```

## 相关模块

- `src/agents/` - Agent 工具调用图像生成
- `src/providers/` - 认证和 API 密钥管理
- `src/config/` - 模型配置
