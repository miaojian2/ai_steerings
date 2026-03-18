---
inclusion: manual
---

# Context Engine 上下文引擎

## 概述

Context Engine 是 OpenClaw 的可插拔上下文管理系统，负责管理对话历史、上下文组装、压缩和子 Agent 上下文传递。

## 目录结构

```
src/context-engine/
├── types.ts        # 接口定义
├── index.ts        # 导出
├── init.ts         # 初始化
├── registry.ts     # 引擎注册表
├── legacy.ts       # 遗留兼容
```

## 核心接口

### ContextEngine

```typescript
interface ContextEngine {
  readonly info: ContextEngineInfo;

  // 初始化
  bootstrap?(params: {
    sessionId: string;
    sessionKey?: string;
    sessionFile: string;
  }): Promise<BootstrapResult>;

  // 消息摄入
  ingest(params: {
    sessionId: string;
    sessionKey?: string;
    message: AgentMessage;
    isHeartbeat?: boolean;
  }): Promise<IngestResult>;

  // 批量摄入
  ingestBatch?(params: {
    sessionId: string;
    sessionKey?: string;
    messages: AgentMessage[];
    isHeartbeat?: boolean;
  }): Promise<IngestBatchResult>;

  // 回合后处理
  afterTurn?(params: {
    sessionId: string;
    sessionKey?: string;
    sessionFile: string;
    messages: AgentMessage[];
    prePromptMessageCount: number;
    autoCompactionSummary?: string;
    isHeartbeat?: boolean;
    tokenBudget?: number;
    runtimeContext?: ContextEngineRuntimeContext;
  }): Promise<void>;

  // 上下文组装
  assemble(params: {
    sessionId: string;
    sessionKey?: string;
    messages: AgentMessage[];
    tokenBudget?: number;
  }): Promise<AssembleResult>;

  // 上下文压缩
  compact(params: {
    sessionId: string;
    sessionKey?: string;
    sessionFile: string;
    tokenBudget?: number;
    force?: boolean;
    currentTokenCount?: number;
    compactionTarget?: "budget" | "threshold";
    customInstructions?: string;
    runtimeContext?: ContextEngineRuntimeContext;
  }): Promise<CompactResult>;

  // 子 Agent 支持
  prepareSubagentSpawn?(params: {
    parentSessionKey: string;
    childSessionKey: string;
    ttlMs?: number;
  }): Promise<SubagentSpawnPreparation | undefined>;

  onSubagentEnded?(params: {
    childSessionKey: string;
    reason: SubagentEndReason;
  }): Promise<void>;

  // 清理
  dispose?(): Promise<void>;
}
```

## 结果类型

### 组装结果

```typescript
type AssembleResult = {
  messages: AgentMessage[];      // 组装后的消息
  estimatedTokens: number;       // 估算 token 数
  systemPromptAddition?: string; // 系统提示词附加内容
};
```

### 压缩结果

```typescript
type CompactResult = {
  ok: boolean;
  compacted: boolean;
  reason?: string;
  result?: {
    summary?: string;
    firstKeptEntryId?: string;
    tokensBefore: number;
    tokensAfter?: number;
    details?: unknown;
  };
};
```

### 引导结果

```typescript
type BootstrapResult = {
  bootstrapped: boolean;
  importedMessages?: number;
  reason?: string;
};
```

## 引擎信息

```typescript
type ContextEngineInfo = {
  id: string;
  name: string;
  version?: string;
  ownsCompaction?: boolean;  // 是否自管理压缩
};
```

## 子 Agent 支持

```typescript
type SubagentSpawnPreparation = {
  rollback: () => void | Promise<void>;  // 失败时回滚
};

type SubagentEndReason = "deleted" | "completed" | "swept" | "released";
```

## 使用场景

### 1. 会话初始化

```typescript
const result = await engine.bootstrap({
  sessionId: "session-123",
  sessionKey: "user:channel:thread",
  sessionFile: "/path/to/session.jsonl",
});
```

### 2. 消息摄入

```typescript
await engine.ingest({
  sessionId: "session-123",
  message: { role: "user", content: "Hello" },
});
```

### 3. 上下文组装

```typescript
const assembled = await engine.assemble({
  sessionId: "session-123",
  messages: allMessages,
  tokenBudget: 8000,
});
// 使用 assembled.messages 作为模型输入
```

### 4. 上下文压缩

```typescript
const compacted = await engine.compact({
  sessionId: "session-123",
  sessionFile: "/path/to/session.jsonl",
  tokenBudget: 8000,
  force: false,
});
```

## 相关模块

- `src/agents/` - Agent 运行时使用 Context Engine
- `src/sessions/` - 会话存储
- `src/memory/` - 向量记忆（可作为 Context Engine 的数据源）
- `extensions/memory-core/` - 记忆核心扩展
- `extensions/memory-lancedb/` - LanceDB 向量存储
