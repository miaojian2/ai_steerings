---
inclusion: always
---

# OpenClaw 项目指南

## 项目概述

OpenClaw 是一个本地优先的个人 AI 助手平台，通过 WebSocket Gateway 连接 20+ 个消息渠道（WhatsApp、Telegram、Slack、Discord 等），支持语音、画布、浏览器控制等能力。

## 技术栈

- **语言**: TypeScript (ESM)，Swift (macOS/iOS)，Kotlin (Android)
- **运行时**: Node.js 22+
- **包管理**: pnpm (monorepo workspace)
- **构建**: tsdown
- **测试**: Vitest
- **Lint/Format**: Oxlint + Oxfmt

## 项目结构

| 目录 | 说明 |
|------|------|
| `src/` | 核心源码 |
| `src/gateway/` | WebSocket 网关服务器（系统核心） |
| `src/cli/` | CLI 入口和参数解析 |
| `src/commands/` | CLI 命令实现 |
| `src/agents/` | AI 代理运行时 |
| `src/channels/` | 消息渠道抽象层 |
| `src/providers/` | LLM 模型提供商抽象 |
| `src/plugins/` | 插件系统核心 |
| `src/plugin-sdk/` | 插件开发 SDK |
| `extensions/` | 扩展插件（渠道、模型提供商等） |
| `skills/` | 内置技能（工具集合） |
| `apps/macos/` | macOS 菜单栏应用 (Swift) |
| `apps/ios/` | iOS 应用 (Swift) |
| `apps/android/` | Android 应用 (Kotlin) |
| `ui/` | Web UI 前端 |
| `docs/` | 文档 (Mintlify) |

## 常用命令

```bash
# 安装依赖
pnpm install

# 构建
pnpm build

# 类型检查
pnpm tsgo

# Lint + Format
pnpm check
pnpm format:fix

# 测试
pnpm test
pnpm test -- <path-or-filter>  # 指定测试

# 开发运行
pnpm openclaw ...              # 运行 CLI
pnpm gateway:watch             # 开发模式启动网关
```

## 编码规范

- 使用严格 TypeScript，避免 `any`
- 不要添加 `@ts-nocheck`
- 文件保持在 ~500-700 LOC 以内
- 测试文件命名：`*.test.ts`，e2e 测试：`*.e2e.test.ts`
- 提交前运行 `pnpm check`
- 使用 `scripts/committer "<msg>" <file...>` 提交代码

## 核心概念

- **Gateway**: WebSocket 控制平面，管理会话、渠道、工具和事件
- **Channels**: 消息平台适配器
- **Providers**: LLM 模型提供商
- **Skills**: 可扩展的工具集合
- **Nodes**: 设备端能力（摄像头、屏幕录制等）
- **Sessions**: 对话上下文管理

## 重要约定

- 产品/文档标题用 **OpenClaw**，CLI/包名/路径用 `openclaw`
- 使用美式英语拼写
- 插件依赖放在扩展自己的 `package.json`，不要加到根目录
- 新渠道/扩展需要更新 `.github/labeler.yml` 和创建对应 GitHub label
- 文档链接使用根相对路径，不带 `.md` 后缀（Mintlify 规范）

## 安全注意

- 入站 DM 视为不可信输入
- 默认 DM 策略是 `pairing`（需要配对码）
- 不要提交真实电话号码、视频或配置值
- 敏感路径修改需要 CODEOWNERS 审批
