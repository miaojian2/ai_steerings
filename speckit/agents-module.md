# Agent 命令注册模块深度解析 (`agents.py`)

## 模块概述

`src/specify_cli/agents.py` 包含约 500+ 行代码，负责：
- 多 Agent 命令格式支持
- 命令文件的解析和渲染
- 向各 AI Agent 注册/注销命令
- YAML Frontmatter 处理

## 核心类

### CommandRegistrar - 命令注册器

```python
class CommandRegistrar:
    """处理多 Agent 命令注册的核心类"""
```

#### AGENT_CONFIGS - Agent 格式配置

```python
AGENT_CONFIGS = {
    "claude": {
        "dir": ".claude/commands",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": ".md"
    },
    "gemini": {
        "dir": ".gemini/commands",
        "format": "toml",
        "args": "{{args}}",
        "extension": ".toml"
    },
    "copilot": {
        "dir": ".github/agents",
        "format": "markdown",
        "args": "$ARGUMENTS",
        "extension": ".agent.md"  # 特殊扩展名
    },
    "codex": {
        "dir": ".agents/skills",
        "format": "skill",  # SKILL.md 格式
        "args": "$ARGUMENTS",
        "extension": ".md"
    },
    "kimi": {
        "dir": ".kimi/skills",
        "format": "skill",  # SKILL.md 格式
        "args": "$ARGUMENTS",
        "extension": ".md"
    },
    # ... 17+ agents
}
```

**配置字段说明**：
| 字段 | 说明 |
|------|------|
| `dir` | 命令文件存放目录 |
| `format` | 文件格式：`markdown`、`toml`、`skill` |
| `args` | 参数占位符：`$ARGUMENTS` 或 `{{args}}` |
| `extension` | 文件扩展名 |

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

### SKILL.md 格式（Codex、Kimi）

```markdown
---
name: speckit-specify
description: "增强的技能描述"
compatibility: "Requires spec-kit project structure"
metadata:
  author: github-spec-kit
  source: templates/commands/specify.md
---

# Speckit Specify Skill

命令内容...
```

## 核心方法

### parse_frontmatter() - 解析 YAML Frontmatter

```python
@staticmethod
def parse_frontmatter(content: str) -> tuple[dict, str]:
    """解析 Markdown 文件的 YAML frontmatter
    
    Args:
        content: 文件内容
        
    Returns:
        (frontmatter_dict, body_content)
    """
```

**处理逻辑**：
1. 检查是否以 `---` 开头
2. 查找结束的 `---`
3. 使用 `yaml.safe_load()` 解析
4. 返回 frontmatter 字典和正文

### render_frontmatter() - 渲染 YAML Frontmatter

```python
@staticmethod
def render_frontmatter(fm: dict) -> str:
    """将字典渲染为 YAML frontmatter 格式"""
```

### render_markdown_command() - 渲染 Markdown 命令

```python
def render_markdown_command(
    self,
    frontmatter: dict,
    body: str,
    source_id: str,
    context_note: str = ""
) -> str:
    """渲染 Markdown 格式的命令文件"""
```

**输出格式**：
```markdown
---
description: "..."
---

<!-- Source: extension-id -->
<!-- Config: .specify/extensions/extension-id/ -->

命令正文...
```

### render_toml_command() - 渲染 TOML 命令

```python
def render_toml_command(
    self,
    frontmatter: dict,
    body: str,
    source_id: str
) -> str:
    """渲染 TOML 格式的命令文件"""
```

**输出格式**：
```toml
description = "..."

prompt = """
命令正文...
"""

# Source: extension-id
```

### render_skill_command() - 渲染 SKILL.md 命令

```python
def render_skill_command(
    self,
    frontmatter: dict,
    body: str,
    source_id: str,
    command_name: str
) -> str:
    """渲染 agentskills.io 规范的 SKILL.md 文件"""
```

## 命令注册流程

### register_commands() - 为单个 Agent 注册命令

```python
def register_commands(
    self,
    agent_name: str,
    commands: List[Dict],
    source_id: str,
    source_dir: Path,
    project_root: Path,
    context_note: str = ""
) -> List[str]:
    """为指定 Agent 注册命令
    
    Args:
        agent_name: Agent 名称（如 "claude"）
        commands: 命令列表 [{"name": "...", "file": "..."}]
        source_id: 来源标识（扩展/预设 ID）
        source_dir: 命令源文件目录
        project_root: 项目根目录
        context_note: 可选的上下文注释
        
    Returns:
        已注册的命令名列表
    """
```

**注册流程**：
1. 获取 Agent 配置
2. 创建目标目录
3. 读取源命令文件
4. 解析 frontmatter
5. 根据 Agent 格式渲染命令
6. 写入目标文件
7. 处理特殊情况（Copilot prompt 文件）

### register_commands_for_all_agents() - 为所有检测到的 Agent 注册

```python
def register_commands_for_all_agents(
    self,
    commands: List[Dict],
    source_id: str,
    source_dir: Path,
    project_root: Path,
    context_note: str = ""
) -> Dict[str, List[str]]:
    """为所有检测到的 Agent 注册命令
    
    Returns:
        {agent_name: [registered_command_names]}
    """
```

**检测逻辑**：
- 检查项目中是否存在 Agent 目录
- 只为已存在目录的 Agent 注册命令

### unregister_commands() - 注销命令

```python
def unregister_commands(
    self,
    registered_commands: Dict[str, List[str]],
    project_root: Path
) -> None:
    """移除之前注册的命令文件
    
    Args:
        registered_commands: {agent_name: [command_names]}
        project_root: 项目根目录
    """
```

## 特殊处理

### Copilot 双文件模式

GitHub Copilot 需要两个文件：
- `.agent.md` - 主命令文件
- `.prompt.md` - 提示文件

```python
@staticmethod
def write_copilot_prompt(project_root: Path, cmd_name: str) -> None:
    """为 Copilot 命令创建配套的 .prompt.md 文件"""
```

### Codex/Kimi Skills 目录结构

```
.agents/skills/           # Codex
  speckit-specify/
    SKILL.md
  speckit-plan/
    SKILL.md

.kimi/skills/             # Kimi
  speckit.specify/        # 使用点号而非连字符
    SKILL.md
```

### 参数占位符转换

不同 Agent 使用不同的参数占位符：
- Markdown Agent: `$ARGUMENTS`
- TOML Agent: `{{args}}`

注册时自动转换：
```python
body = body.replace("$ARGUMENTS", agent_config["args"])
```

## 与其他模块的集成

### 扩展系统 (`extensions.py`)

```python
# extensions.py 中的 CommandRegistrar 是包装器
class CommandRegistrar:
    def __init__(self):
        from .agents import CommandRegistrar as _Registrar
        self._registrar = _Registrar()
    
    def register_commands_for_all_agents(self, manifest, ...):
        # 添加扩展特定的上下文注释
        context_note = f"<!-- Extension: {manifest.id} -->"
        return self._registrar.register_commands_for_all_agents(
            manifest.commands, manifest.id, ..., context_note=context_note
        )
```

### 预设系统 (`presets.py`)

预设的命令覆盖也通过 `CommandRegistrar` 注册：
```python
def _register_commands(self, manifest, preset_dir):
    registrar = CommandRegistrar()
    return registrar.register_commands_for_all_agents(
        command_templates, manifest.id, preset_dir, self.project_root
    )
```

## 设计原则

1. **格式无关性**：统一的 API 处理多种命令格式
2. **增量注册**：只为已存在的 Agent 目录注册
3. **可追溯性**：命令文件包含来源注释
4. **安全注销**：记录已注册命令，支持完整清理

## 扩展点

添加新 Agent 格式支持：
1. 在 `AGENT_CONFIGS` 中添加配置
2. 如需新格式，添加 `render_xxx_command()` 方法
3. 在 `register_commands()` 中添加格式分支
