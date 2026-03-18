---
inclusion: manual
---

# Link Understanding 链接理解

## 概述

Link Understanding 模块负责从消息中提取链接并获取其内容摘要，用于增强 Agent 的上下文理解能力。

## 目录结构

```
src/link-understanding/
├── runner.ts       # 主运行器
├── detect.ts       # 链接检测
├── apply.ts        # 应用到上下文
├── format.ts       # 输出格式化
├── defaults.ts     # 默认配置
```

## 核心功能

### 链接检测

```typescript
import { extractLinksFromMessage } from "./detect.js";

const links = extractLinksFromMessage(
  "Check out https://example.com and https://docs.openclaw.ai",
  { maxLinks: 5 }
);
// ["https://example.com", "https://docs.openclaw.ai"]
```

### 运行链接理解

```typescript
import { runLinkUnderstanding } from "./runner.js";

const result = await runLinkUnderstanding({
  cfg: config,
  ctx: messageContext,
  message: "See https://example.com for details",
});

console.log(result.urls);     // 检测到的 URL
console.log(result.outputs);  // 每个 URL 的内容摘要
```

## 配置

在 `tools.links` 中配置：

```yaml
tools:
  links:
    enabled: true
    maxLinks: 5
    timeoutSeconds: 30
    scope:
      channels:
        - whatsapp
        - telegram
      chatTypes:
        - dm
        - group
    models:
      - type: cli
        command: readability
        args:
          - "--url"
          - "{{LinkUrl}}"
```

## 链接模型配置

```typescript
type LinkModelConfig = {
  type?: "cli";           // 目前只支持 CLI
  command: string;        // 命令
  args?: string[];        // 参数（支持模板变量）
  timeoutSeconds?: number;
};
```

### 模板变量

- `{{LinkUrl}}` - 当前处理的 URL
- 其他 MsgContext 变量

## 作用域控制

通过 `scope` 配置控制在哪些场景启用：

```typescript
type LinkToolsConfig = {
  enabled?: boolean;
  maxLinks?: number;
  timeoutSeconds?: number;
  scope?: {
    channels?: string[];
    chatTypes?: ("dm" | "group" | "channel")[];
    sessionKeys?: string[];
  };
  models?: LinkModelConfig[];
};
```

## 结果类型

```typescript
type LinkUnderstandingResult = {
  urls: string[];      // 检测到的所有 URL
  outputs: string[];   // 成功获取的内容摘要
};
```

## 默认值

```typescript
const DEFAULT_LINK_TIMEOUT_SECONDS = 30;
```

## 相关模块

- `src/media-understanding/` - 媒体理解（类似的作用域控制）
- `src/web-search/` - Web 搜索
- `src/agents/` - Agent 上下文增强
