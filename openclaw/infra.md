---
inclusion: manual
---

# 基础设施 (src/infra/)

## 概述

基础设施模块提供底层工具函数、系统集成和通用能力，是整个项目的基础支撑。

## 目录结构

```
src/infra/
├── net/                   # 网络相关
│   ├── ssrf.ts           # SSRF 防护
│   ├── proxy-*.ts        # 代理支持
│   └── hostname.ts       # 主机名处理
├── outbound/              # 出站消息
│   ├── deliver.ts        # 消息投递
│   ├── message.ts        # 消息处理
│   └── targets.ts        # 目标解析
├── format-time/           # 时间格式化
├── tls/                   # TLS 相关
├── exec-*.ts              # 命令执行
├── file-*.ts              # 文件操作
├── gateway-*.ts           # Gateway 相关
├── install-*.ts           # 安装相关
├── update-*.ts            # 更新相关
└── *.ts                   # 其他工具
```

## 核心模块

### 网络安全 (net/)

```typescript
import { resolvePinnedHostname, isBlockedHostnameOrIp } from "./net/ssrf.js";

// SSRF 防护 - 解析并固定 IP
const pinned = await resolvePinnedHostname("example.com");

// 检查是否被阻止
const blocked = isBlockedHostnameOrIp("169.254.169.254");
```

### 命令执行 (exec-*.ts)

```typescript
import { execSafeCommand } from "./exec-safety.js";
import { resolveMergedSafeBinProfileFixtures } from "./exec-safe-bin-runtime-policy.js";

// 安全执行命令
const result = await execSafeCommand(command, args, options);

// 获取安全二进制配置
const safeBins = resolveMergedSafeBinProfileFixtures(cfg);
```

### 出站消息 (outbound/)

```typescript
import { deliverMessage } from "./outbound/deliver.js";
import { resolveTarget } from "./outbound/targets.js";

// 投递消息
await deliverMessage({
  channel: "telegram",
  target: "user-123",
  payload: { text: "Hello" }
});

// 解析目标
const target = resolveTarget(cfg, targetSpec);
```

### 文件操作

```typescript
import { readJsonFileWithFallback, writeJsonFileAtomically } from "./json-file.js";
import { withFileLock } from "./file-lock.js";

// 安全读取 JSON
const { value, exists } = await readJsonFileWithFallback(path, defaultValue);

// 原子写入
await writeJsonFileAtomically(path, data);

// 文件锁
await withFileLock(path, options, async () => {
  // 临界区操作
});
```

### Gateway 工具

```typescript
import { resolveGatewayLock, isGatewayRunning } from "./gateway-lock.js";
import { findGatewayProcesses } from "./gateway-processes.js";

// 检查 Gateway 锁
const lock = await resolveGatewayLock();

// 查找 Gateway 进程
const processes = await findGatewayProcesses();
```

### 更新检查

```typescript
import { checkForUpdates } from "./update-check.js";
import { runGlobalUpdate } from "./update-global.js";

// 检查更新
const update = await checkForUpdates();

// 执行更新
await runGlobalUpdate();
```

### 设备身份

```typescript
import { getDeviceIdentity, resolveDeviceId } from "./device-identity.js";

// 获取设备身份
const identity = await getDeviceIdentity();

// 解析设备 ID
const deviceId = resolveDeviceId();
```

### 路径工具

```typescript
import { resolveRequiredHomeDir } from "./home-dir.js";
import { isPathSafe, normalizePath } from "./path-safety.js";

// 获取 home 目录
const home = resolveRequiredHomeDir();

// 路径安全检查
const safe = isPathSafe(path, workspaceRoot);
```

### 重试和退避

```typescript
import { withRetry } from "./retry.js";
import { exponentialBackoff } from "./backoff.js";

// 带重试执行
const result = await withRetry(async () => {
  return await riskyOperation();
}, { maxRetries: 3 });

// 计算退避时间
const delay = exponentialBackoff(attempt);
```

### 系统事件

```typescript
import { emitSystemEvent, onSystemEvent } from "./system-events.js";

// 发送系统事件
emitSystemEvent("gateway.started", { port: 8080 });

// 监听系统事件
onSystemEvent("gateway.started", (data) => {
  console.log("Gateway started on port", data.port);
});
```

### 心跳系统

```typescript
import { HeartbeatRunner } from "./heartbeat-runner.js";

// 心跳运行器
const runner = new HeartbeatRunner({
  cfg: config,
  agentId: "default"
});

await runner.start();
```

## 工具函数

### 时间格式化 (format-time/)

```typescript
import { formatDuration, formatRelativeTime } from "./format-time/index.js";

formatDuration(3600000);  // "1h"
formatRelativeTime(date); // "2 hours ago"
```

### 错误处理

```typescript
import { isTransientError, wrapError } from "./errors.js";

// 检查是否是临时错误
if (isTransientError(err)) {
  // 可以重试
}

// 包装错误
throw wrapError(err, "Operation failed");
```

### 去重

```typescript
import { dedupeByKey } from "./dedupe.js";

const unique = dedupeByKey(items, item => item.id);
```

## 阅读建议

1. 从 `net/ssrf.ts` 了解网络安全
2. 阅读 `outbound/deliver.ts` 理解消息投递
3. 查看 `exec-safety.ts` 了解命令执行安全
4. 研究 `file-lock.ts` 了解并发控制
5. 查看 `gateway-*.ts` 了解 Gateway 工具
