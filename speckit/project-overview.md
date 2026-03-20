# Spec Kit 项目概览

## 项目简介

**GitHub Spec Kit** 是一个开源的规范驱动开发 (Spec-Driven Development, SDD) 工具包。它的核心理念是：**规范先行，代码服从规范**。

传统开发中，代码是核心，规范只是辅助文档。SDD 颠覆了这一模式——规范成为可执行的主要产物，代码是规范的具体实现。

## 核心组件

### 1. Specify CLI (`src/specify_cli/`)

命令行工具，用于初始化和管理 SDD 项目：

- `__init__.py` - 主入口，包含 `AGENT_CONFIG`（所有 AI Agent 配置的单一数据源）、CLI 命令实现
- `agents.py` - `CommandRegistrar` 类，负责向各 AI Agent 注册命令
- `presets.py` - 预设管理器，处理模板覆盖和自定义工作流
- `extensions.py` - 扩展管理器，添加新功能和命令

### 2. 模板系统 (`templates/`)

核心 SDD 工作流模板：

| 模板 | 用途 |
|------|------|
| `constitution-template.md` | 项目宪法/开发原则 |
| `spec-template.md` | 功能规范模板 |
| `plan-template.md` | 技术实现计划 |
| `tasks-template.md` | 可执行任务列表 |
| `checklist-template.md` | 质量检查清单 |

### 3. 命令文件 (`templates/commands/`)

AI Agent 可调用的斜杠命令：

- `/speckit.constitution` - 创建项目原则
- `/speckit.specify` - 定义功能需求
- `/speckit.plan` - 生成技术计划
- `/speckit.tasks` - 分解为任务列表
- `/speckit.implement` - 执行实现
- `/speckit.clarify` - 澄清模糊需求
- `/speckit.analyze` - 一致性分析

## 支持的 AI Agent

项目支持 30+ 种 AI 编码助手，分为两类：

### CLI 类型（需要安装命令行工具）
- Claude Code (`claude`)
- Gemini CLI (`gemini`)
- Codex CLI (`codex`)
- Kiro CLI (`kiro-cli`)
- Amp (`amp`)
- 等等...

### IDE 类型（内置于 IDE）
- GitHub Copilot (`copilot`)
- Cursor (`cursor-agent`)
- Windsurf (`windsurf`)
- Kilo Code (`kilocode`)
- 等等...

## 关键配置

### AGENT_CONFIG 结构

```python
AGENT_CONFIG = {
    "agent-key": {  # 使用实际 CLI 工具名作为 key
        "name": "显示名称",
        "folder": ".agent/",  # Agent 文件目录
        "commands_subdir": "commands",  # 命令子目录
        "install_url": "https://...",  # 安装文档 URL
        "requires_cli": True/False,  # 是否需要 CLI 工具
    }
}
```

### 命令文件格式

**Markdown 格式**（大多数 Agent）：
```markdown
---
description: "命令描述"
---

命令内容，使用 $ARGUMENTS 作为参数占位符
```

**TOML 格式**（Gemini、Tabnine）：
```toml
description = "命令描述"

prompt = """
命令内容，使用 {{args}} 作为参数占位符
"""
```

## 项目结构

```
spec-kit/
├── src/specify_cli/     # CLI 源代码
├── templates/           # 核心模板
│   └── commands/        # 斜杠命令
├── extensions/          # 扩展系统
├── presets/             # 预设系统
├── scripts/             # 辅助脚本 (bash/powershell)
├── tests/               # 测试文件
├── docs/                # 文档
└── media/               # 媒体资源
```

## 版本信息

- 当前版本: `0.3.2`
- Python 要求: `>=3.11`
- 主要依赖: typer, rich, httpx, pyyaml, packaging
