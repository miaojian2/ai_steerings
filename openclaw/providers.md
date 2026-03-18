---
inclusion: manual
---

# OpenClaw Providers 模块指南

## 概述

Providers 模块负责 LLM 模型提供商的抽象和集成。它定义了统一的接口，让 OpenClaw 可以无缝切换不同的 AI 模型提供商（OpenAI、Anthropic、Google、Azure 等）。

## 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                      Plugin System                          │
│  api.registerProvider(buildOpenAIProvider())                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    ProviderPlugin 接口                       │
│  - id, label, auth[], catalog, resolveDynamicModel...       │
└─────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │   OpenAI    │     │  Anthropic  │     │   Google    │
   │  Extension  │     │  Extension  │     │  Extension  │
   └─────────────┘     └─────────────┘     └─────────────┘
```

## 目录结构

| 路径 | 说明 |
|------|------|
| `src/providers/` | 核心共享代码（OAuth、模型定义等） |
| `src/plugins/types.ts` | ProviderPlugin 接口定义 |
| `src/plugin-sdk/provider-*.ts` | Provider SDK 工具函数 |
| `extensions/<provider>/` | 各提供商的具体实现 |

## 核心接口：ProviderPlugin

```typescript
type ProviderPlugin = {
  // 基础信息
  id: string;                    // 提供商 ID，如 "openai"
  label: string;                 // 显示名称
  docsPath?: string;             // 文档路径
  envVars?: string[];            // 相关环境变量

  // 认证方法
  auth: ProviderAuthMethod[];    // 支持的认证方式（API key、OAuth 等）

  // 模型目录
  catalog?: ProviderPluginCatalog;  // 模型发现和配置

  // 动态模型解析
  resolveDynamicModel?: (ctx) => ProviderRuntimeModel | null;
  prepareDynamicModel?: (ctx) => Promise<void>;
  normalizeResolvedModel?: (ctx) => ProviderRuntimeModel | null;

  // 运行时认证
  prepareRuntimeAuth?: (ctx) => Promise<ProviderPreparedRuntimeAuth | null>;

  // 能力声明
  capabilities?: Partial<ProviderCapabilities>;

  // Thinking 策略
  supportsXHighThinking?: (ctx) => boolean;
  resolveDefaultThinkingLevel?: (ctx) => ThinkLevel;

  // 使用量/计费
  resolveUsageAuth?: (ctx) => Promise<ProviderResolvedUsageAuth | null>;
  fetchUsageSnapshot?: (ctx) => Promise<ProviderUsageSnapshot | null>;

  // 其他钩子...
};
```

## 认证方法

每个 Provider 可以支持多种认证方式：

```typescript
type ProviderAuthMethod = {
  id: string;           // 如 "api-key", "oauth", "setup-token"
  label: string;        // 显示名称
  kind: ProviderAuthKind;  // "oauth" | "api_key" | "token" | "device_code"
  run: (ctx: ProviderAuthContext) => Promise<ProviderAuthResult>;
  runNonInteractive?: (ctx) => Promise<OpenClawConfig | null>;
};
```

常用认证工具函数：
- `createProviderApiKeyAuthMethod()` - 创建 API Key 认证方法
- `ensureApiKeyFromOptionEnvOrPrompt()` - 从选项/环境变量/提示获取 API Key
- `upsertAuthProfile()` - 保存认证配置

## 扩展实现示例

### 基本结构

```
extensions/openai/
├── index.ts                    # 插件入口
├── openai-provider.ts          # Provider 实现
├── openai-codex-provider.ts    # Codex OAuth 变体
├── media-understanding-provider.ts  # 媒体理解
├── shared.ts                   # 共享工具
├── openclaw.plugin.json        # 插件元数据
└── package.json
```

### 插件入口 (index.ts)

```typescript
import { definePluginEntry } from "openclaw/plugin-sdk/core";
import { buildOpenAIProvider } from "./openai-provider.js";

export default definePluginEntry({
  id: "openai",
  name: "OpenAI Provider",
  register(api) {
    api.registerProvider(buildOpenAIProvider());
    api.registerSpeechProvider(buildOpenAISpeechProvider());
    api.registerMediaUnderstandingProvider(openaiMediaUnderstandingProvider);
    api.registerImageGenerationProvider(buildOpenAIImageGenerationProvider());
  },
});
```

### Provider 实现

```typescript
export function buildOpenAIProvider(): ProviderPlugin {
  return {
    id: "openai",
    label: "OpenAI",
    envVars: ["OPENAI_API_KEY"],
    auth: [
      createProviderApiKeyAuthMethod({
        providerId: "openai",
        methodId: "api-key",
        envVar: "OPENAI_API_KEY",
        defaultModel: "openai/gpt-5.4",
        // ...
      }),
    ],
    resolveDynamicModel: (ctx) => {
      // 处理未在目录中的模型 ID
    },
    normalizeResolvedModel: (ctx) => {
      // 标准化模型配置（如 API 类型转换）
    },
    capabilities: {
      providerFamily: "openai",
    },
  };
}
```

## 关键概念

### 1. 模型解析流程

```
用户请求模型 "openai/gpt-5.4"
        │
        ▼
1. 查找已发现/静态模型目录
        │ (未找到)
        ▼
2. 调用 resolveDynamicModel()
        │ (需要网络)
        ▼
3. 调用 prepareDynamicModel() (异步预热)
        │
        ▼
4. 再次调用 resolveDynamicModel()
        │
        ▼
5. 调用 normalizeResolvedModel() (最终标准化)
```

### 2. 运行时模型 (ProviderRuntimeModel)

```typescript
type ProviderRuntimeModel = {
  id: string;           // 模型 ID
  name: string;         // 显示名称
  api: string;          // API 类型 ("openai-responses", "anthropic", ...)
  provider: string;     // 提供商 ID
  baseUrl?: string;     // API 基础 URL
  reasoning?: boolean;  // 是否支持推理
  input: string[];      // 输入类型 ["text", "image"]
  contextWindow: number;
  maxTokens: number;
  cost: { input, output, cacheRead, cacheWrite };
};
```

### 3. 认证配置存储

认证信息存储在 `~/.openclaw/credentials/` 下的 auth profile store 中：

```typescript
type AuthProfileCredential =
  | ApiKeyCredential      // { type: "api_key", apiKey, ... }
  | OAuthCredential       // { type: "oauth", access, refresh, expires, ... }
  | TokenCredential;      // { type: "token", token, ... }
```

## 常用 SDK 导入

```typescript
// 核心类型
import { definePluginEntry, type ProviderPlugin } from "openclaw/plugin-sdk/core";

// 认证工具
import {
  createProviderApiKeyAuthMethod,
  upsertAuthProfile,
  ensureApiKeyFromOptionEnvOrPrompt,
} from "openclaw/plugin-sdk/provider-auth";

// 模型工具
import {
  normalizeModelCompat,
  normalizeProviderId,
} from "openclaw/plugin-sdk/provider-models";

// 使用量
import { fetchClaudeUsage } from "openclaw/plugin-sdk/provider-usage";
```

## 现有 Provider 扩展

| 扩展 | 说明 |
|------|------|
| `extensions/openai/` | OpenAI + Codex OAuth |
| `extensions/anthropic/` | Anthropic (setup-token + API key) |
| `extensions/google/` | Google AI (Gemini) |
| `extensions/amazon-bedrock/` | AWS Bedrock |
| `extensions/azure-openai/` | Azure OpenAI |
| `extensions/ollama/` | Ollama 本地模型 |
| `extensions/openrouter/` | OpenRouter 聚合 |
| `extensions/together/` | Together AI |
| `extensions/mistral/` | Mistral AI |
| `extensions/xai/` | xAI (Grok) |
| `extensions/github-copilot/` | GitHub Copilot |

## 开发建议

1. **新增 Provider**：复制一个类似的扩展（如 `extensions/openai/`）作为模板
2. **测试**：在 `extensions/<provider>/` 下添加 `*.test.ts` 文件
3. **文档**：更新 `docs/providers/` 下的对应文档
4. **认证**：优先使用 `createProviderApiKeyAuthMethod()` 简化 API Key 认证
5. **模型目录**：实现 `catalog` 钩子动态发现可用模型

## 调试技巧

```bash
# 查看已配置的 providers
pnpm openclaw models list

# 测试特定模型
pnpm openclaw agent --model openai/gpt-5.4 --message "test"

# 查看认证状态
pnpm openclaw models auth status

# 诊断问题
pnpm openclaw doctor
```
