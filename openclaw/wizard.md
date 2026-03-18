---
inclusion: manual
---

# Wizard 设置向导

## 概述

Wizard 模块提供交互式设置向导，用于引导用户完成 OpenClaw 的初始配置。

## 目录结构

```
src/wizard/
├── session.ts              # 向导会话管理
├── prompts.ts              # 提示定义
├── clack-prompter.ts       # Clack 提示器封装
├── setup.ts                # 主设置流程
├── setup.types.ts          # 设置类型
├── setup.completion.ts     # 完成处理
├── setup.finalize.ts       # 最终化
├── setup.gateway-config.ts # Gateway 配置
├── setup.secret-input.ts   # 密钥输入
```

## 向导会话

```typescript
import { createWizardSession, WizardSession } from "./session.js";

const session = createWizardSession({
  title: "OpenClaw Setup",
  steps: ["auth", "channels", "gateway"],
});

session.start();
session.nextStep();
session.complete();
```

## Clack 提示器

基于 @clack/prompts 的封装：

```typescript
import { ClackPrompter } from "./clack-prompter.js";

const prompter = new ClackPrompter();

// 文本输入
const name = await prompter.text({
  message: "Enter your name:",
  placeholder: "John Doe",
});

// 选择
const provider = await prompter.select({
  message: "Select provider:",
  options: [
    { value: "openai", label: "OpenAI" },
    { value: "anthropic", label: "Anthropic" },
  ],
});

// 确认
const confirmed = await prompter.confirm({
  message: "Continue?",
});

// 密码
const apiKey = await prompter.password({
  message: "Enter API key:",
});
```

## 设置流程

### 主设置

```typescript
import { runSetup } from "./setup.js";

await runSetup({
  config: openclawConfig,
  runtime: runtimeEnv,
  nonInteractive: false,
});
```

### Gateway 配置

```typescript
import { setupGatewayConfig } from "./setup.gateway-config.js";

await setupGatewayConfig({
  prompter,
  config,
  existingConfig,
});
```

### 密钥输入

```typescript
import { promptSecretInput } from "./setup.secret-input.js";

const secret = await promptSecretInput({
  prompter,
  message: "Enter API key:",
  envVar: "OPENAI_API_KEY",
  configPath: "providers.openai.apiKey",
});
```

## 设置类型

```typescript
type SetupOptions = {
  config: OpenClawConfig;
  runtime: RuntimeEnv;
  nonInteractive?: boolean;
  skipChannels?: boolean;
  skipGateway?: boolean;
};

type SetupResult = {
  completed: boolean;
  configUpdated: boolean;
  channelsConfigured: string[];
  gatewayConfigured: boolean;
};
```

## 完成处理

```typescript
import { handleSetupCompletion } from "./setup.completion.js";

await handleSetupCompletion({
  result: setupResult,
  config,
  runtime,
});
```

## 最终化

```typescript
import { finalizeSetup } from "./setup.finalize.js";

await finalizeSetup({
  config,
  runtime,
  installDaemon: true,
  startGateway: true,
});
```

## 相关模块

- `src/commands/onboard*.ts` - 引导命令
- `src/commands/setup*.ts` - 设置命令
- `src/cli/` - CLI 框架
- `src/terminal/` - 终端工具
