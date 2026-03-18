---
inclusion: manual
---

# Cron 定时任务系统

## 概述

Cron 模块提供定时任务调度功能，支持一次性任务、周期性任务和 cron 表达式。可以触发 Agent 执行、发送消息或调用 webhook。

## 目录结构

```
src/cron/
├── types.ts              # 类型定义
├── types-shared.ts       # 共享类型
├── service.ts            # CronService 主类
├── service/              # 服务实现
│   ├── state.ts          # 服务状态管理
│   ├── ops.ts            # 操作实现
│   ├── jobs.ts           # 任务管理
│   ├── timer.ts          # 定时器管理
│   ├── store.ts          # 存储层
│   └── locked.ts         # 锁机制
├── store.ts              # 任务存储
├── store-migration.ts    # 存储迁移
├── schedule.ts           # 调度计算
├── parse.ts              # 表达式解析
├── normalize.ts          # 数据规范化
├── stagger.ts            # 错峰调度
├── delivery.ts           # 消息投递
├── run-log.ts            # 运行日志
├── session-reaper.ts     # 会话清理
├── isolated-agent/       # 隔离 Agent 执行
│   ├── run.ts            # 执行逻辑
│   ├── session.ts        # 会话管理
│   ├── delivery-dispatch.ts
│   └── delivery-target.ts
```

## 核心类型

### 调度类型

```typescript
type CronSchedule =
  | { kind: "at"; at: string }                    // 一次性：指定时间
  | { kind: "every"; everyMs: number; anchorMs?: number }  // 周期性
  | { kind: "cron"; expr: string; tz?: string; staggerMs?: number };  // cron 表达式
```

### 任务定义

```typescript
type CronJob = {
  id: string;
  name?: string;
  enabled: boolean;
  schedule: CronSchedule;
  sessionTarget: CronSessionTarget;  // "main" | "isolated" | "current" | "session:xxx"
  wakeMode: CronWakeMode;            // "next-heartbeat" | "now"
  payload: CronPayload;
  delivery: CronDelivery;
  failureAlert?: CronFailureAlert | false;
  state: CronJobState;
  createdAtMs: number;
  updatedAtMs: number;
};
```

### 任务载荷

```typescript
type CronPayload =
  | { kind: "systemEvent"; text: string }  // 系统事件
  | {
      kind: "agentTurn";                   // Agent 执行
      message: string;
      model?: string;
      fallbacks?: string[];
      thinking?: string;
      timeoutSeconds?: number;
      deliver?: boolean;
      channel?: CronMessageChannel;
      to?: string;
    };
```

### 投递配置

```typescript
type CronDelivery = {
  mode: "none" | "announce" | "webhook";
  channel?: ChannelId | "last";
  to?: string;
  accountId?: string;
  bestEffort?: boolean;
  failureDestination?: CronFailureDestination;
};
```

## CronService API

```typescript
class CronService {
  // 生命周期
  async start(): Promise<void>;
  stop(): void;
  async status(): Promise<CronStatus>;

  // 任务管理
  async list(opts?: { includeDisabled?: boolean }): Promise<CronJob[]>;
  async listPage(opts?: CronListPageOptions): Promise<CronListPageResult>;
  async add(input: CronJobCreate): Promise<CronJob>;
  async update(id: string, patch: CronJobPatch): Promise<CronJob>;
  async remove(id: string): Promise<void>;

  // 执行
  async run(id: string, mode?: "due" | "force"): Promise<CronRunOutcome>;
  async enqueueRun(id: string, mode?: "due" | "force"): Promise<void>;
  getJob(id: string): CronJob | undefined;

  // 唤醒
  wake(opts: { mode: "now" | "next-heartbeat"; text: string }): void;
}
```

## 任务状态

```typescript
type CronJobState = {
  nextRunAtMs?: number;
  runningAtMs?: number;
  lastRunAtMs?: number;
  lastRunStatus?: "ok" | "error" | "skipped";
  lastError?: string;
  lastDurationMs?: number;
  consecutiveErrors?: number;
  lastFailureAlertAtMs?: number;
  lastDeliveryStatus?: CronDeliveryStatus;
};
```

## 隔离 Agent 执行

`isolated-agent/` 子模块处理在隔离环境中执行 Agent 任务：

- 独立会话管理
- 技能快照
- 投递目标解析
- 子 Agent 跟进

## 失败告警

```typescript
type CronFailureAlert = {
  after?: number;           // 连续失败次数后告警
  channel?: CronMessageChannel;
  to?: string;
  cooldownMs?: number;      // 告警冷却时间
  mode?: "announce" | "webhook";
  accountId?: string;
};
```

## 相关模块

- `src/agents/` - Agent 运行时
- `src/sessions/` - 会话管理
- `src/channels/` - 消息渠道（投递目标）
- `src/gateway/` - Gateway 集成
