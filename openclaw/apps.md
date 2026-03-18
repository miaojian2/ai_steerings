---
inclusion: manual
---

# OpenClaw Apps 模块指南

## 概述

Apps 目录包含 OpenClaw 的原生客户端应用，包括 macOS 菜单栏应用、iOS 应用和 Android 应用。这些应用作为 Gateway 的客户端，提供原生 UI 和设备特定功能。

## 目录结构

```
apps/
├── macos/           # macOS 菜单栏应用 (Swift)
├── ios/             # iOS 应用 (Swift)
├── android/         # Android 应用 (Kotlin)
└── shared/          # 共享代码 (OpenClawKit)
```

## macOS 应用

### 结构

```
apps/macos/
├── Package.swift           # Swift Package 配置
├── Package.resolved        # 依赖锁定
├── README.md
├── Icon.icon/              # 应用图标
├── Sources/
│   └── OpenClaw/
│       ├── App/            # 应用入口
│       ├── Views/          # SwiftUI 视图
│       ├── ViewModels/     # 视图模型
│       ├── Services/       # 服务层
│       ├── Models/         # 数据模型
│       └── Resources/      # 资源文件
└── Tests/                  # 测试
```

### 功能

- 菜单栏控制面板
- Gateway 状态监控
- Voice Wake（语音唤醒）
- Push-to-Talk（按键说话）
- Talk Mode 覆盖层
- WebChat 集成
- 调试工具
- 远程 Gateway 控制

### 构建

```bash
# 打开 Xcode 项目
cd apps/macos
swift package resolve
open Package.swift

# 或使用脚本
pnpm mac:package
```

### 技术栈

- Swift 5.9+
- SwiftUI
- Observation framework（状态管理）
- WebSocket（Gateway 通信）
- Speech framework（语音）

## iOS 应用

### 结构

```
apps/ios/
├── project.yml             # XcodeGen 配置
├── Sources/
│   ├── App/                # 应用入口
│   ├── Views/              # SwiftUI 视图
│   ├── Services/           # 服务层
│   └── Resources/          # 资源文件
└── Tests/
```

### 功能

- Gateway 节点模式
- Canvas 画布
- Voice Wake 语音唤醒
- Talk Mode 语音对话
- 摄像头快照/录制
- 屏幕录制
- Bonjour 设备发现
- 设备配对

### 构建

```bash
# 生成 Xcode 项目
pnpm ios:gen

# 打开项目
pnpm ios:open

# 构建
pnpm ios:build

# 运行（模拟器）
pnpm ios:run
```

### 技术栈

- Swift 5.9+
- SwiftUI
- XcodeGen（项目生成）
- AVFoundation（媒体）
- Network framework（Bonjour）

## Android 应用

### 结构

```
apps/android/
├── app/
│   ├── build.gradle.kts    # 应用构建配置
│   └── src/
│       └── main/
│           ├── kotlin/     # Kotlin 源码
│           ├── res/        # 资源文件
│           └── AndroidManifest.xml
├── build.gradle.kts        # 项目构建配置
├── settings.gradle.kts
└── scripts/                # 构建脚本
```

### 功能

- Connect 标签页（设置码/手动连接）
- Chat 会话
- Voice 语音标签页
- Canvas 画布
- 摄像头/屏幕录制
- Android 设备命令
  - 通知
  - 位置
  - SMS
  - 照片
  - 联系人
  - 日历
  - 运动数据
  - 应用更新

### 构建

```bash
# 组装 Debug APK
pnpm android:assemble

# 安装到设备
pnpm android:install

# 运行
pnpm android:run

# 测试
pnpm android:test

# Lint
pnpm android:lint
```

### 技术栈

- Kotlin
- Jetpack Compose
- Gradle (Kotlin DSL)
- OkHttp（网络）
- CameraX（摄像头）

## 共享代码 (OpenClawKit)

### 结构

```
apps/shared/
└── OpenClawKit/
    └── Sources/
        └── OpenClawProtocol/
            └── GatewayModels.swift  # 协议模型
```

### 用途

- Gateway WebSocket 协议定义
- 共享数据模型
- 跨平台工具函数

### 协议生成

```bash
# 生成 Swift 协议模型
pnpm protocol:gen:swift

# 检查协议一致性
pnpm protocol:check
```

## 版本管理

版本位置：
- `apps/android/app/build.gradle.kts` - versionName/versionCode
- `apps/ios/Sources/Info.plist` - CFBundleShortVersionString/CFBundleVersion
- `apps/macos/Sources/OpenClaw/Resources/Info.plist` - 同上

## 开发注意事项

### macOS

- 签名构建才能保持权限
- 使用 Observation framework，不要用 ObservableObject
- 通过应用启动/停止 Gateway，不要用 tmux

### iOS

- 优先使用真机测试
- 需要配置签名
- XcodeGen 生成项目

### Android

- 优先使用真机测试
- 使用 ktlint 格式化
- Compose 优先

## 常用命令汇总

```bash
# macOS
pnpm mac:package          # 打包
pnpm mac:open             # 打开
pnpm mac:restart          # 重启

# iOS
pnpm ios:gen              # 生成项目
pnpm ios:open             # 打开
pnpm ios:build            # 构建
pnpm ios:run              # 运行

# Android
pnpm android:assemble     # 组装
pnpm android:install      # 安装
pnpm android:run          # 运行
pnpm android:test         # 测试
pnpm android:lint         # Lint
pnpm android:format       # 格式化
```

## 测试

```bash
# Android 单元测试
pnpm android:test

# Android 集成测试
pnpm android:test:integration

# iOS 测试（通过 Xcode）
```
