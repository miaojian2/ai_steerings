---
inclusion: manual
---

# Daemon 守护进程服务

## 概述

Daemon 模块负责将 OpenClaw Gateway 安装为系统服务，支持 macOS (launchd)、Linux (systemd) 和 Windows (Scheduled Task)。

## 目录结构

```
src/daemon/
├── service.ts              # 统一服务接口
├── service-types.ts        # 服务类型定义
├── service-runtime.ts      # 运行时信息
├── service-env.ts          # 环境变量处理
├── service-audit.ts        # 服务审计
├── launchd.ts              # macOS LaunchAgent
├── launchd-plist.ts        # plist 生成
├── launchd-restart-handoff.ts
├── systemd.ts              # Linux systemd
├── systemd-unit.ts         # unit 文件生成
├── systemd-hints.ts        # systemd 提示
├── systemd-linger.ts       # linger 支持
├── schtasks.ts             # Windows Scheduled Task
├── schtasks-exec.ts        # 任务执行
├── constants.ts            # 常量定义
├── paths.ts                # 路径解析
├── runtime-*.ts            # 运行时检测
├── program-args.ts         # 程序参数
├── cmd-argv.ts             # 命令行参数
├── diagnostics.ts          # 诊断工具
├── inspect.ts              # 服务检查
├── output.ts               # 输出格式化
├── node-service.ts         # Node.js 服务
├── exec-file.ts            # 执行文件
```

## 统一服务接口

```typescript
type GatewayService = {
  label: string;              // 服务标签（如 "LaunchAgent"）
  loadedText: string;         // 已加载状态文本
  notLoadedText: string;      // 未加载状态文本
  install: (args: GatewayServiceInstallArgs) => Promise<void>;
  uninstall: (args: GatewayServiceManageArgs) => Promise<void>;
  stop: (args: GatewayServiceControlArgs) => Promise<void>;
  restart: (args: GatewayServiceControlArgs) => Promise<GatewayServiceRestartResult>;
  isLoaded: (args: GatewayServiceEnvArgs) => Promise<boolean>;
  readCommand: (env: GatewayServiceEnv) => Promise<GatewayServiceCommandConfig | null>;
  readRuntime: (env: GatewayServiceEnv) => Promise<GatewayServiceRuntime>;
};

// 获取当前平台的服务
const service = resolveGatewayService();
```

## 平台支持

### macOS (LaunchAgent)

```typescript
import {
  installLaunchAgent,
  uninstallLaunchAgent,
  stopLaunchAgent,
  restartLaunchAgent,
  isLaunchAgentLoaded,
  readLaunchAgentProgramArguments,
  readLaunchAgentRuntime,
} from "./launchd.js";
```

plist 文件位置：`~/Library/LaunchAgents/ai.openclaw.gateway.plist`

### Linux (systemd)

```typescript
import {
  installSystemdService,
  uninstallSystemdService,
  stopSystemdService,
  restartSystemdService,
  isSystemdServiceEnabled,
  readSystemdServiceExecStart,
  readSystemdServiceRuntime,
} from "./systemd.js";
```

unit 文件位置：`~/.config/systemd/user/openclaw-gateway.service`

### Windows (Scheduled Task)

```typescript
import {
  installScheduledTask,
  uninstallScheduledTask,
  stopScheduledTask,
  restartScheduledTask,
  isScheduledTaskInstalled,
  readScheduledTaskCommand,
  readScheduledTaskRuntime,
} from "./schtasks.js";
```

## 安装参数

```typescript
type GatewayServiceInstallArgs = {
  env: GatewayServiceEnv;
  command: GatewayServiceCommandConfig;
  serviceEnv?: Record<string, string>;  // 服务环境变量
  force?: boolean;
};

type GatewayServiceCommandConfig = {
  program: string;
  args: string[];
};
```

## 运行时信息

```typescript
type GatewayServiceRuntime = {
  installed: boolean;
  running: boolean;
  pid?: number;
  startedAt?: Date;
  command?: GatewayServiceCommandConfig;
  env?: Record<string, string>;
};
```

## 重启结果

```typescript
type GatewayServiceRestartResult = {
  outcome: "restarted" | "scheduled";
};

// 描述重启结果
const description = describeGatewayServiceRestart("Gateway", result);
// { scheduled: false, message: "Gateway service restarted." }
```

## 运行时检测

```typescript
// runtime-binary.ts - 检测 Node.js 二进制
// runtime-hints.ts - 运行时提示
// runtime-paths.ts - 路径解析
// runtime-parse.ts - 解析运行时信息
// runtime-format.ts - 格式化输出
```

## 相关模块

- `src/gateway/` - Gateway 服务器
- `src/commands/` - CLI 命令（daemon install/uninstall）
- `src/cli/` - CLI 入口
