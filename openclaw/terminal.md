---
inclusion: manual
---

# Terminal 终端工具

## 概述

Terminal 模块提供 CLI 输出格式化、ANSI 颜色处理、表格渲染等终端相关工具。

## 目录结构

```
src/terminal/
├── palette.ts              # Lobster 调色板
├── theme.ts                # 主题定义
├── ansi.ts                 # ANSI 转义码处理
├── table.ts                # 表格渲染
├── note.ts                 # 提示信息
├── links.ts                # 终端链接
├── safe-text.ts            # 安全文本处理
├── stream-writer.ts        # 流写入器
├── progress-line.ts        # 进度行
├── health-style.ts         # 健康状态样式
├── restore.ts              # 终端状态恢复
├── prompt-style.ts         # 提示样式
├── prompt-select-styled.ts # 样式化选择提示
```

## Lobster 调色板

```typescript
// palette.ts
export const LOBSTER_PALETTE = {
  accent: "#FF5A2D",       // 主强调色
  accentBright: "#FF7A3D", // 亮强调色
  accentDim: "#D14A22",    // 暗强调色
  info: "#FF8A5B",         // 信息色
  success: "#2FBF71",      // 成功色
  warn: "#FFB020",         // 警告色
  error: "#E23D2D",        // 错误色
  muted: "#8B7F77",        // 静音色
} as const;
```

使用调色板（"lobster seam"）：

```typescript
import { LOBSTER_PALETTE } from "./palette.js";
import chalk from "chalk";

console.log(chalk.hex(LOBSTER_PALETTE.success)("✓ Success"));
console.log(chalk.hex(LOBSTER_PALETTE.error)("✗ Error"));
```

## ANSI 处理

```typescript
import { stripAnsi, hasAnsi, wrapAnsi } from "./ansi.js";

// 移除 ANSI 转义码
const plain = stripAnsi("\x1b[32mGreen\x1b[0m");  // "Green"

// 检测是否包含 ANSI
const has = hasAnsi("\x1b[32mGreen\x1b[0m");  // true

// 自动换行（保留 ANSI）
const wrapped = wrapAnsi(coloredText, 80);
```

## 表格渲染

```typescript
import { renderTable } from "./table.js";

const table = renderTable({
  headers: ["Name", "Status", "Port"],
  rows: [
    ["Gateway", "Running", "18789"],
    ["Canvas", "Stopped", "-"],
  ],
  columnWidths: [20, 10, 8],
});
console.log(table);
```

## 安全文本

```typescript
import { safeText, truncateText } from "./safe-text.js";

// 移除不安全字符
const safe = safeText(userInput);

// 截断长文本
const truncated = truncateText(longText, 100);
```

## 流写入器

```typescript
import { createStreamWriter } from "./stream-writer.js";

const writer = createStreamWriter(process.stdout);
writer.write("Processing...");
writer.clearLine();
writer.write("Done!");
```

## 进度行

```typescript
import { ProgressLine } from "./progress-line.js";

const progress = new ProgressLine();
progress.update("Loading... 50%");
progress.clear();
progress.finish("Complete!");
```

## 健康状态样式

```typescript
import { formatHealthStatus } from "./health-style.js";

const styled = formatHealthStatus("healthy");  // 绿色 ✓
const styled2 = formatHealthStatus("degraded"); // 黄色 ⚠
const styled3 = formatHealthStatus("unhealthy"); // 红色 ✗
```

## 终端链接

```typescript
import { terminalLink } from "./links.js";

// 创建可点击的终端链接
const link = terminalLink("OpenClaw Docs", "https://docs.openclaw.ai");
```

## 提示样式

```typescript
import { styledSelect } from "./prompt-select-styled.js";

const choice = await styledSelect({
  message: "Select provider:",
  options: [
    { value: "openai", label: "OpenAI" },
    { value: "anthropic", label: "Anthropic" },
  ],
});
```

## 相关模块

- `src/cli/` - CLI 入口使用终端工具
- `src/commands/` - 命令输出格式化
- `src/tui/` - 终端 UI 组件
