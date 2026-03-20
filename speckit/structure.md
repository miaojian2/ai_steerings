# Spec Kit 项目结构

## 仓库根目录

```
spec-kit/
├── src/                      # 源代码
├── templates/                # 核心模板和命令
├── extensions/               # 扩展系统
├── presets/                  # 预设系统
├── scripts/                  # 辅助脚本
├── tests/                    # 测试文件
├── docs/                     # 文档站点
├── media/                    # 媒体资源
├── newsletters/              # 新闻通讯
├── .devcontainer/            # 开发容器配置
├── .github/                  # GitHub 工作流
├── pyproject.toml            # Python 项目配置
├── README.md                 # 项目说明
├── AGENTS.md                 # Agent 集成指南
├── spec-driven.md            # SDD 方法论文档
└── spec-kit.code-workspace   # VS Code 工作区
```

## 源代码 (`src/`)

```
src/
└── specify_cli/
    ├── __init__.py      # 主入口 (4100+ 行)
    │                    # - AGENT_CONFIG 定义
    │                    # - CLI 命令实现
    │                    # - 模板下载和合并
    │                    # - Git 仓库初始化
    │
    ├── agents.py        # Agent 命令注册 (500+ 行)
    │                    # - CommandRegistrar 类
    │                    # - 多 Agent 格式支持
    │                    # - Frontmatter 解析
    │
    ├── extensions.py    # 扩展系统 (2000+ 行)
    │                    # - ExtensionManifest
    │                    # - ExtensionRegistry
    │                    # - ExtensionManager
    │                    # - ExtensionCatalog
    │                    # - HookExecutor
    │
    └── presets.py       # 预设系统 (1700+ 行)
                         # - PresetManifest
                         # - PresetRegistry
                         # - PresetManager
                         # - PresetCatalog
                         # - PresetResolver
```

## 模板系统 (`templates/`)

```
templates/
├── commands/                    # 斜杠命令定义
│   ├── specify.md               # /speckit.specify
│   ├── plan.md                  # /speckit.plan
│   ├── tasks.md                 # /speckit.tasks
│   ├── implement.md             # /speckit.implement
│   ├── constitution.md          # /speckit.constitution
│   ├── clarify.md               # /speckit.clarify
│   ├── analyze.md               # /speckit.analyze
│   ├── checklist.md             # /speckit.checklist
│   └── taskstoissues.md         # /speckit.taskstoissues
│
├── spec-template.md             # 功能规范模板
├── plan-template.md             # 实现计划模板
├── tasks-template.md            # 任务列表模板
├── constitution-template.md     # 项目宪法模板
├── checklist-template.md        # 检查清单模板
├── agent-file-template.md       # Agent 上下文模板
└── vscode-settings.json         # VS Code 设置模板
```

## 扩展系统 (`extensions/`)

```
extensions/
├── README.md                        # 扩展用户指南
├── EXTENSION-API-REFERENCE.md       # API 参考
├── EXTENSION-DEVELOPMENT-GUIDE.md   # 开发指南
├── EXTENSION-PUBLISHING-GUIDE.md    # 发布指南
├── EXTENSION-USER-GUIDE.md          # 使用指南
├── RFC-EXTENSION-SYSTEM.md          # 设计 RFC
├── catalog.json                     # 官方扩展目录
├── catalog.community.json           # 社区扩展目录
├── selftest/                        # 自测扩展
│   ├── extension.yml
│   └── commands/
└── template/                        # 扩展模板
    ├── extension.yml
    ├── commands/
    ├── README.md
    └── CHANGELOG.md
```

## 预设系统 (`presets/`)

```
presets/
├── README.md                    # 预设用户指南
├── ARCHITECTURE.md              # 架构文档
├── PUBLISHING.md                # 发布指南
├── catalog.json                 # 官方预设目录
├── catalog.community.json       # 社区预设目录
├── scaffold/                    # 预设脚手架
│   ├── preset.yml
│   ├── README.md
│   ├── commands/
│   │   ├── speckit.specify.md
│   │   └── speckit.myext.myextcmd.md
│   └── templates/
│       ├── spec-template.md
│       └── myext-template.md
└── self-test/                   # 自测预设
    ├── preset.yml
    ├── commands/
    └── templates/
```

## 脚本 (`scripts/`)

```
scripts/
├── bash/                            # Bash 脚本
│   ├── common.sh                    # 共享函数
│   ├── create-new-feature.sh        # 创建功能分支
│   ├── setup-plan.sh                # 设置计划文档
│   ├── check-prerequisites.sh       # 检查前置条件
│   └── update-agent-context.sh      # 更新 Agent 上下文
│
└── powershell/                      # PowerShell 脚本
    ├── common.ps1
    ├── create-new-feature.ps1
    ├── setup-plan.ps1
    ├── check-prerequisites.ps1
    └── update-agent-context.ps1
```

## 测试 (`tests/`)

```
tests/
├── __init__.py
├── test_extensions.py               # 扩展系统测试
├── test_presets.py                  # 预设系统测试
├── test_agent_config_consistency.py # Agent 配置一致性
├── test_ai_skills.py                # AI 技能测试
├── test_cursor_frontmatter.py       # Cursor frontmatter 测试
├── test_merge.py                    # JSON 合并测试
└── hooks/                           # 钩子测试
    ├── TESTING.md
    ├── spec.md
    ├── plan.md
    ├── tasks.md
    └── .specify/
```

## 文档 (`docs/`)

```
docs/
├── README.md
├── index.md                 # 首页
├── installation.md          # 安装指南
├── quickstart.md            # 快速开始
├── local-development.md     # 本地开发
├── upgrade.md               # 升级指南
├── docfx.json               # DocFX 配置
└── toc.yml                  # 目录结构
```

## 开发容器 (`.devcontainer/`)

```
.devcontainer/
├── devcontainer.json        # 容器配置
└── post-create.sh           # 创建后脚本
```

## 初始化后的项目结构

使用 `specify init` 后，目标项目会有以下结构：

```
my-project/
├── .specify/
│   ├── templates/
│   │   ├── overrides/           # 本地模板覆盖
│   │   ├── commands/            # 命令模板
│   │   ├── spec-template.md
│   │   ├── plan-template.md
│   │   ├── tasks-template.md
│   │   ├── constitution-template.md
│   │   └── checklist-template.md
│   ├── extensions/
│   │   └── .registry            # 扩展注册表
│   ├── presets/
│   │   └── .registry            # 预设注册表
│   ├── scripts/
│   │   ├── bash/
│   │   └── powershell/
│   ├── extensions.yml           # 钩子配置
│   └── init-options.json        # 初始化选项
│
├── .<agent>/                    # Agent 特定目录
│   └── commands/                # 或 workflows/, prompts/, agents/
│       ├── speckit.specify.md
│       ├── speckit.plan.md
│       ├── speckit.tasks.md
│       ├── speckit.implement.md
│       └── ...
│
├── memory/
│   └── constitution.md          # 项目宪法
│
├── specs/                       # 功能规范目录
│   └── ###-feature-name/
│       ├── spec.md
│       ├── plan.md
│       ├── tasks.md
│       ├── research.md
│       ├── data-model.md
│       ├── quickstart.md
│       ├── contracts/
│       └── checklists/
│
└── .gitignore
```

## 关键文件说明

| 文件 | 用途 |
|------|------|
| `AGENT_CONFIG` | 所有 AI Agent 配置的单一数据源 |
| `.registry` | JSON 格式的注册表，跟踪已安装的扩展/预设 |
| `extension.yml` | 扩展清单文件 |
| `preset.yml` | 预设清单文件 |
| `extensions.yml` | 项目级钩子配置 |
| `init-options.json` | 记录 `specify init` 时的选项 |
| `constitution.md` | 项目的核心原则和约束 |

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| Agent Key | 实际 CLI 工具名 | `claude`, `kiro-cli`, `cursor-agent` |
| 扩展 ID | 小写字母数字+连字符 | `jira`, `azure-devops` |
| 命令名 | `speckit.<ext>.<cmd>` | `speckit.jira.specstoissues` |
| 模板名 | 小写字母数字+连字符 | `spec-template`, `plan-template` |
| 功能分支 | `###-short-name` | `001-user-auth`, `002-payment` |
