---
inclusion: manual
---

# 配置系统 (src/config/)

## 概述

配置系统是 OpenClaw 的核心基础设施，负责管理所有运行时配置、验证、迁移和持久化。

## 目录结构

```
src/config/
├── config.ts          # 主导出入口
├── schema.ts          # JSON Schema 生成和查询
├── io.ts              # 配置文件读写
├── paths.ts           # 配置路径解析
├── validation.ts      # 配置验证
├── defaults.ts        # 默认值
├── types.ts           # 类型定义主入口
├── types.*.ts         # 各模块类型定义
├── zod-schema.ts      # Zod schema 定义
├── zod-schema.*.ts    # 各模块 Zod schema
├── legacy-migrate.ts  # 旧版配置迁移
├── legacy.*.ts        # 迁移规则
├── sessions/          # 会话配置管理
└── *.test.ts          # 测试文件
```

## 核心组件

### 配置 Schema (schema.ts)

- `OpenClawSchema`: 基于 Zod 的完整配置 schema
- `buildConfigSchema()`: 生成 JSON Schema（支持插件扩展）
- `lookupConfigSchema()`: 按路径查询 schema 节点
- UI hints 系统：为 Control UI 提供字段元数据

### 配置 I/O (io.ts)

```typescript
// 读取配置
const config = await loadConfig();
const snapshot = await readConfigFileSnapshot();

// 写入配置
await writeConfigFile(config);

// 运行时快照
const runtime = getRuntimeConfigSnapshot();
setRuntimeConfigSnapshot(newConfig);
```

### 类型定义 (types.*.ts)

按模块组织的类型定义：
- `types.agents.ts` - Agent 配置
- `types.channels.ts` - 渠道配置
- `types.gateway.ts` - Gateway 配置
- `types.models.ts` - 模型配置
- `types.secrets.ts` - 密钥配置
- `types.tools.ts` - 工具配置
- 等等...

### 配置验证 (validation.ts)

```typescript
// 验证配置对象
const result = validateConfigObject(config);
const resultWithPlugins = validateConfigObjectWithPlugins(config, plugins);
```

### 路径解析 (paths.ts)

```typescript
import { resolveConfigPath, resolveStateDir } from "./paths.js";

const configPath = resolveConfigPath();  // ~/.openclaw/openclaw.json
const stateDir = resolveStateDir();      // ~/.openclaw/
```

## 配置结构

主要配置节点：
- `agents` - Agent 列表和默认设置
- `channels` - 消息渠道配置
- `gateway` - Gateway 服务配置
- `providers` - LLM 提供商配置
- `plugins` - 插件配置
- `tools` - 工具配置
- `secrets` - 密钥管理
- `session` - 会话设置
- `messages` - 消息处理设置

## 会话配置 (sessions/)

```
src/config/sessions/
├── store.ts           # 会话存储
├── session-key.ts     # 会话键生成
├── disk-budget.ts     # 磁盘配额管理
├── pruning.ts         # 会话清理
└── artifacts.ts       # 会话产物管理
```

## 迁移系统

支持从旧版配置自动迁移：

```typescript
import { migrateLegacyConfig } from "./legacy-migrate.js";

const migrated = await migrateLegacyConfig(oldConfig);
```

迁移规则分布在 `legacy.migrations.part-*.ts` 文件中。

## 环境变量

配置支持环境变量替换：

```typescript
// 在配置中使用
{
  "gateway": {
    "auth": {
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

相关代码在 `env-substitution.ts` 和 `env-vars.ts`。

## 密钥输入 (Secret Input)

支持多种密钥输入方式：
- 直接值
- 环境变量引用 (`$env:VAR_NAME`)
- 文件引用 (`$file:/path/to/secret`)
- 1Password 引用

详见 `types.secrets.ts` 和 `zod-schema.sensitive.ts`。

## 阅读建议

1. 从 `config.ts` 了解导出结构
2. 阅读 `types.ts` 理解配置类型
3. 查看 `schema.ts` 了解 schema 生成
4. 研究 `io.ts` 理解读写流程
5. 查看测试文件了解各种边界情况
