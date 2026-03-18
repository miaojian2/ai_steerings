---
inclusion: manual
---

# Commands CLI 命令

## 概述

Commands 模块包含所有 CLI 命令的实现，是用户与 OpenClaw 交互的主要入口。

## 目录结构

```
src/commands/
├── agent*.ts               # agent 命令
├── agents*.ts              # agents 管理命令
├── auth-choice*.ts         # 认证选择
├── backup*.ts              # 备份命令
├── channels*.ts            # 渠道命令
├── configure*.ts           # 配置命令
├── daemon*.ts              # 守护进程命令
├── dashboard*.ts           # 仪表板
├── doctor*.ts              # 诊断命令
├── docs.ts                 # 文档命令
├── gateway*.ts             # Gateway 命令
├── health*.ts              # 健康检查
├── message*.ts             # 消息命令
├── model*.ts               # 模型命令
├── models*.ts              # 模型管理
├── oauth*.ts               # OAuth 流程
├── ollama-setup.ts         # Ollama 设置
├── onboard*.ts             # 引导流程
├── reset.ts                # 重置命令
├── sandbox*.ts             # 沙箱命令
├── sessions*.ts            # 会话命令
├── setup*.ts               # 设置命令
├── signal-install.ts       # Signal 安装
├── status*.ts              # 状态命令
├── uninstall.ts            # 卸载命令
├── agent/                  # agent 子模块
├── channel-setup/          # 渠道设置
├── channels/               # 渠道子命令
├── gateway-status/         # Gateway 状态
├── models/                 # 模型子命令
├── onboard-non-interactive/# 非交互引导
├── setup/                  # 设置子模块
├── status-all/             # 全状态
```

## 主要命令

### agent

运行 Agent 会话：

```bash
openclaw agent --message "Hello"
openclaw agent --local  # 本地模式
openclaw agent --session my-session
```

### agents

管理 Agent 配置：

```bash
openclaw agents list
openclaw agents add my-agent
openclaw agents delete my-agent
openclaw agents bind my-agent --channel telegram
```

### channels

管理消息渠道：

```bash
openclaw channels status
openclaw channels add telegram
openclaw channels remove telegram
openclaw channels logs telegram
```

### config / configure

配置管理：

```bash
openclaw config get gateway.port
openclaw config set gateway.port 18789
openclaw configure gateway
openclaw configure channels
```

### doctor

诊断和修复：

```bash
openclaw doctor
openclaw doctor --fix
openclaw doctor --check auth
```

### gateway

Gateway 管理：

```bash
openclaw gateway run
openclaw gateway status
openclaw gateway status --deep
```

### models

模型管理：

```bash
openclaw models list
openclaw models set openai/gpt-4o
openclaw models auth
```

### onboard

引导设置：

```bash
openclaw onboard
openclaw onboard --non-interactive
```

### sessions

会话管理：

```bash
openclaw sessions list
openclaw sessions show session-id
openclaw sessions cleanup
```

### status

系统状态：

```bash
openclaw status
openclaw status --all
openclaw status --json
```

## 命令结构

每个命令通常包含：

1. 主文件（如 `agent.ts`）- 命令定义和入口
2. 测试文件（如 `agent.test.ts`）- 单元测试
3. 子模块目录（如 `agent/`）- 复杂逻辑拆分

## 命令模式

### 依赖注入

```typescript
type AgentCommandDeps = {
  config: OpenClawConfig;
  runtime: RuntimeEnv;
  // ...
};

export async function runAgentCommand(deps: AgentCommandDeps) {
  // 实现
}
```

### 进度显示

```typescript
import { createProgress } from "../cli/progress.js";

const progress = createProgress();
progress.start("Loading...");
// 工作
progress.stop("Done!");
```

### 表格输出

```typescript
import { renderTable } from "../terminal/table.js";

const output = renderTable({
  headers: ["Name", "Status"],
  rows: data.map(item => [item.name, item.status]),
});
```

## 相关模块

- `src/cli/` - CLI 框架和参数解析
- `src/terminal/` - 终端输出工具
- `src/config/` - 配置读写
- `src/gateway/` - Gateway 交互
