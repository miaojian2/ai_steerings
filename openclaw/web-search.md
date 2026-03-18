---
inclusion: manual
---

# Web Search 网络搜索

## 概述

Web Search 模块为 Agent 提供网络搜索能力，支持多个搜索提供商（Brave、Perplexity 等），可作为工具供 Agent 调用。

## 目录结构

```
src/web-search/
├── runtime.ts    # 运行时 API
```

## 核心 API

### 解析搜索定义

```typescript
import { resolveWebSearchDefinition } from "./runtime.js";

const resolved = resolveWebSearchDefinition({
  config: openclawConfig,
  sandboxed: false,
  providerId: "brave",  // 可选，指定提供商
});

if (resolved) {
  const { provider, definition } = resolved;
  console.log(provider.id);           // "brave"
  console.log(definition.name);       // 工具名称
  console.log(definition.description);// 工具描述
}
```

### 执行搜索

```typescript
import { runWebSearch } from "./runtime.js";

const result = await runWebSearch({
  config: openclawConfig,
  args: { query: "OpenClaw AI assistant" },
});

console.log(result.provider);  // 使用的提供商
console.log(result.result);    // 搜索结果
```

### 列出提供商

```typescript
import { listWebSearchProviders } from "./runtime.js";

const providers = listWebSearchProviders({ config });
// [{ id: "brave", ... }, { id: "perplexity", ... }]
```

## 配置

在 `tools.web.search` 中配置：

```yaml
tools:
  web:
    search:
      enabled: true
      provider: brave
      apiKey: ${BRAVE_API_KEY}
      # 或使用 Perplexity
      # provider: perplexity
      # perplexity:
      #   apiKey: ${PERPLEXITY_API_KEY}
```

## 提供商接口

```typescript
type PluginWebSearchProviderEntry = {
  id: string;
  label?: string;
  envVars: string[];  // 支持的环境变量
  getCredentialValue: (config?: Record<string, unknown>) => string | undefined;
  createTool: (params: {
    config?: OpenClawConfig;
    searchConfig?: Record<string, unknown>;
    runtimeMetadata?: RuntimeWebSearchMetadata;
  }) => WebSearchProviderToolDefinition | null;
};
```

## 工具定义

```typescript
type WebSearchProviderToolDefinition = {
  name: string;
  description: string;
  inputSchema: object;
  execute: (args: Record<string, unknown>) => Promise<Record<string, unknown>>;
};
```

## 提供商自动检测

如果未配置 `provider`，运行时会自动检测可用的 API 密钥：

1. 检查配置中的 API 密钥
2. 检查环境变量
3. 使用第一个有凭证的提供商

## 支持的提供商

### Brave Search

- 环境变量：`BRAVE_API_KEY`
- 配置路径：`tools.web.search.apiKey`

### Perplexity

- 环境变量：`PERPLEXITY_API_KEY`
- 配置路径：`tools.web.search.perplexity.apiKey`

## 沙箱模式

在沙箱模式下，web search 默认启用（安全的只读操作）。

## 相关模块

- `src/agents/` - Agent 工具调用
- `src/plugins/` - 插件提供商注册
- `extensions/brave/` - Brave 搜索扩展
- `extensions/perplexity/` - Perplexity 扩展
