---
inclusion: manual
---

# Web UI (ui/)

## 概述

Web UI 是 OpenClaw 的前端界面，提供 Control UI 和 WebChat 功能。

## 目录结构

```
ui/
├── src/
│   ├── main.ts            # 入口文件
│   ├── ui/                # UI 组件
│   ├── i18n/              # 国际化
│   └── styles/            # 样式
├── public/                # 静态资源
├── index.html             # HTML 入口
├── package.json           # 依赖配置
├── vite.config.ts         # Vite 配置
└── vitest.config.ts       # 测试配置
```

## 技术栈

- 构建工具: Vite
- 测试: Vitest
- 样式: CSS

## 开发命令

```bash
# 安装依赖
cd ui && pnpm install

# 开发模式
pnpm dev

# 构建
pnpm build

# 测试
pnpm test
```

## 功能模块

### Control UI

- Gateway 状态监控
- 配置管理
- 渠道状态
- Agent 管理
- 安全审计

### WebChat

- 实时对话
- 消息历史
- 媒体支持
- 工具调用展示

## 与 Gateway 集成

UI 通过 WebSocket 连接到 Gateway：

```typescript
// 连接到 Gateway
const ws = new WebSocket(`ws://localhost:8080/ws`);

// 认证
ws.send(JSON.stringify({
  type: "auth",
  token: "gateway-token"
}));
```

## 国际化 (i18n/)

支持多语言：

```typescript
import { t } from "./i18n";

const label = t("settings.title");
```

## 样式 (styles/)

使用 CSS 变量实现主题：

```css
:root {
  --color-primary: #007bff;
  --color-background: #ffffff;
}

[data-theme="dark"] {
  --color-primary: #4dabf7;
  --color-background: #1a1a1a;
}
```

## 构建输出

构建产物位于 `ui/dist/`，由 Gateway 静态服务。

## 阅读建议

1. 从 `main.ts` 了解入口
2. 查看 `ui/` 目录了解组件结构
3. 阅读 `vite.config.ts` 了解构建配置
4. 查看 Gateway 的 `control-ui-assets.ts` 了解集成方式
