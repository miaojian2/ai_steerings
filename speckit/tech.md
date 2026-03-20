# Spec Kit 技术文档

## 技术栈

| 组件 | 技术 | 版本要求 |
|------|------|----------|
| 语言 | Python | >=3.11 |
| CLI 框架 | Typer | - |
| 终端 UI | Rich | - |
| HTTP 客户端 | httpx | - |
| YAML 解析 | PyYAML | >=6.0 |
| JSON5 解析 | json5 | >=0.13.0 |
| 版本管理 | packaging | >=23.0 |
| 路径匹配 | pathspec | >=0.12.0 |
| TLS | truststore | >=0.10.4 |
| 键盘输入 | readchar | - |

## 核心模块架构

### `src/specify_cli/__init__.py`

主入口模块，包含：

**AGENT_CONFIG** - AI Agent 配置的单一数据源：
```python
AGENT_CONFIG = {
    "agent-key": {
        "name": "显示名称",
        "folder": ".agent/",
        "commands_subdir": "commands",
        "install_url": "https://...",
        "requires_cli": True/False,
    }
}
```

**CLI 命令**：
- `specify init` - 初始化项目
- `specify check` - 检查工具安装
- `specify extension *` - 扩展管理
- `specify preset *` - 预设管理

**关键函数**：
- `check_tool()` - 检查 CLI 工具是否安装
- `download_template_from_github()` - 从 GitHub 下载模板
- `merge_json_files()` - 合并 JSON 配置
- `init_git_repo()` - 初始化 Git 仓库

### `src/specify_cli/agents.py`

**CommandRegistrar** 类 - 向 AI Agent 注册命令：

```python
AGENT_CONFIGS = {
    "claude": {"dir": ".claude/commands", "format": "markdown", "args": "$ARGUMENTS", "extension": ".md"},
    "gemini": {"dir": ".gemini/commands", "format": "toml", "args": "{{args}}", "extension": ".toml"},
    "copilot": {"dir": ".github/agents", "format": "markdown", "args": "$ARGUMENTS", "extension": ".agent.md"},
    # ... 17+ agents
}
```

**核心方法**：
- `register_commands()` - 为特定 Agent 注册命令
- `register_commands_for_all_agents()` - 为所有检测到的 Agent 注册
- `unregister_commands()` - 移除已注册命令
- `parse_frontmatter()` / `render_frontmatter()` - YAML frontmatter 处理

### `src/specify_cli/extensions.py`

**ExtensionManifest** - 扩展清单验证：
- Schema 版本: 1.0
- 必需字段: `schema_version`, `extension`, `requires`, `provides`
- 命令名格式: `speckit.<extension-id>.<command>`

**ExtensionRegistry** - 扩展注册表：
- 存储位置: `.specify/extensions/.registry`
- 方法: `add()`, `remove()`, `get()`, `list()`, `is_installed()`

**ExtensionManager** - 扩展生命周期管理：
- `install_from_directory()` / `install_from_zip()`
- `remove()` - 支持 `keep_config` 选项
- `check_compatibility()` - 版本兼容性检查

**ExtensionCatalog** - 扩展目录管理：
- 支持多目录堆栈
- 1 小时缓存
- `search()`, `get_extension_info()`

### `src/specify_cli/presets.py`

**PresetManifest** - 预设清单：
- 模板类型: `template`, `command`, `script`
- 命令名格式: `speckit.<command>` 或 `speckit.<ext>.<cmd>`

**PresetRegistry** - 预设注册表：
- 存储位置: `.specify/presets/.registry`
- 支持优先级排序

**PresetManager** - 预设管理：
- `install_from_directory()` / `install_from_zip()`
- `_register_commands()` - 命令覆盖注册
- `_register_skills()` - 技能注册（用于 `--ai-skills`）

**PresetResolver** - 模板解析：
```
解析顺序:
1. .specify/templates/overrides/
2. .specify/presets/<id>/templates/
3. .specify/extensions/<id>/templates/
4. .specify/templates/
```

## 命令文件格式

### Markdown 格式（大多数 Agent）

```markdown
---
description: "命令描述"
scripts:
  sh: scripts/bash/script.sh "{ARGS}"
  ps: scripts/powershell/script.ps1 "{ARGS}"
---

## User Input
$ARGUMENTS

## Outline
1. 步骤一
2. 步骤二
```

### TOML 格式（Gemini、Tabnine）

```toml
description = "命令描述"

prompt = """
命令内容，使用 {{args}} 作为参数占位符
"""
```

## 脚本系统

### Bash 脚本 (`scripts/bash/`)

- `common.sh` - 共享函数（`resolve_template()`）
- `create-new-feature.sh` - 创建功能分支和规范文件
- `setup-plan.sh` - 设置计划文档
- `check-prerequisites.sh` - 检查前置条件
- `update-agent-context.sh` - 更新 Agent 上下文

### PowerShell 脚本 (`scripts/powershell/`)

对应的 PowerShell 实现，函数命名使用 PascalCase：
- `Resolve-Template`
- `Create-NewFeature`
- 等等

## 文件系统布局

### 项目初始化后

```
project/
├── .specify/
│   ├── templates/           # 核心模板
│   │   ├── overrides/       # 本地覆盖
│   │   └── commands/        # 命令模板
│   ├── extensions/          # 已安装扩展
│   │   ├── .registry        # 扩展注册表
│   │   └── <ext-id>/        # 扩展目录
│   ├── presets/             # 已安装预设
│   │   ├── .registry        # 预设注册表
│   │   └── <preset-id>/     # 预设目录
│   ├── scripts/             # 辅助脚本
│   └── extensions.yml       # 钩子配置
├── .<agent>/                # Agent 命令目录
│   └── commands/            # 注册的命令
├── memory/
│   └── constitution.md      # 项目宪法
└── specs/
    └── <feature>/           # 功能规范目录
        ├── spec.md
        ├── plan.md
        ├── tasks.md
        ├── research.md
        ├── data-model.md
        ├── quickstart.md
        ├── contracts/
        └── checklists/
```

## API 设计原则

1. **单一数据源** - `AGENT_CONFIG` 是所有 Agent 配置的唯一来源
2. **CLI 工具名作为 Key** - 使用实际可执行文件名，避免映射
3. **深拷贝返回** - Registry 的 `get()` 和 `list()` 返回深拷贝
4. **优雅降级** - 损坏的注册表条目被过滤而非报错
5. **版本兼容性** - 使用 `packaging` 库进行语义版本检查

## 错误处理

**异常层次**：
```
ExtensionError (基类)
├── ValidationError      # 清单验证失败
└── CompatibilityError   # 版本不兼容

PresetError (基类)
├── PresetValidationError
└── PresetCompatibilityError
```

## 测试

测试位于 `tests/` 目录：
- `test_extensions.py` - 扩展系统测试
- `test_presets.py` - 预设系统测试
- `test_agent_config_consistency.py` - Agent 配置一致性
- `test_merge.py` - JSON 合并测试

运行测试：
```bash
pytest tests/ -v
```
