# CLI 主模块深度解析 (`__init__.py`)

## 模块概述

`src/specify_cli/__init__.py` 是 Spec Kit CLI 的主入口模块，包含约 4100+ 行代码，负责：
- AI Agent 配置管理
- CLI 命令实现
- 模板下载与提取
- Git 仓库初始化
- Agent Skills 安装

## 核心数据结构

### AGENT_CONFIG - AI Agent 配置单一数据源

```python
AGENT_CONFIG = {
    "agent-key": {  # 使用实际 CLI 工具名作为 key
        "name": "显示名称",
        "folder": ".agent/",           # Agent 文件目录
        "commands_subdir": "commands", # 命令子目录
        "install_url": "https://...",  # 安装文档 URL
        "requires_cli": True/False,    # 是否需要 CLI 工具
    }
}
```

**设计原则**：
- Key 必须是实际可执行文件名（如 `cursor-agent` 而非 `cursor`）
- 避免特殊映射，`check_tool()` 直接使用 key 查找
- `requires_cli=False` 表示 IDE 内置 Agent，跳过 CLI 检查

**当前支持的 Agent（27+）**：

| Agent Key | 名称 | 目录 | 命令子目录 | 类型 |
|-----------|------|------|------------|------|
| `claude` | Claude Code | `.claude/` | `commands` | CLI |
| `gemini` | Gemini CLI | `.gemini/` | `commands` | CLI |
| `copilot` | GitHub Copilot | `.github/` | `agents` | IDE |
| `cursor-agent` | Cursor | `.cursor/` | `commands` | IDE |
| `windsurf` | Windsurf | `.windsurf/` | `workflows` | IDE |
| `codex` | Codex CLI | `.agents/` | `skills` | CLI |
| `kiro-cli` | Kiro CLI | `.kiro/` | `prompts` | CLI |
| `kimi` | Kimi Code | `.kimi/` | `skills` | CLI |

### AI_ASSISTANT_ALIASES - Agent 别名映射

```python
AI_ASSISTANT_ALIASES = {
    "kiro": "kiro-cli",  # 允许用户使用简短别名
}
```

## 关键函数

### check_tool() - CLI 工具检查

```python
def check_tool(tool: str, tracker: StepTracker = None) -> bool:
    """检查工具是否已安装"""
    # 特殊处理：Claude CLI 迁移后的路径
    if tool == "claude":
        if CLAUDE_LOCAL_PATH.exists():  # ~/.claude/local/claude
            return True
    
    # Kiro 支持两个可执行名
    if tool == "kiro-cli":
        found = shutil.which("kiro-cli") or shutil.which("kiro")
    else:
        found = shutil.which(tool) is not None
    
    return found
```

### download_template_from_github() - 模板下载

```python
def download_template_from_github(
    ai_assistant: str,
    download_dir: Path,
    *,
    script_type: str = "sh",
    github_token: str = None
) -> Tuple[Path, dict]:
    """从 GitHub Releases 下载模板"""
```

**功能**：
- 获取最新 Release 信息
- 匹配 `spec-kit-template-{agent}-{script_type}` 资产
- 支持 GitHub Token 认证（提高 API 限额）
- 返回 ZIP 路径和元数据

**Rate Limit 处理**：
- 解析 `X-RateLimit-*` 响应头
- 提供详细的错误信息和重试建议
- 认证请求 5000/小时 vs 未认证 60/小时

### merge_json_files() - JSON 配置合并

```python
def merge_json_files(
    existing_path: Path,
    new_content: Any,
    verbose: bool = False
) -> Optional[dict]:
    """礼貌式深度合并 JSON 配置"""
```

**合并策略**：
- 新 key 被添加
- 现有 key 被保留（不覆盖）
- 嵌套字典递归合并
- 支持 JSONC（带注释的 JSON）

### install_ai_skills() - Agent Skills 安装

```python
def install_ai_skills(
    project_path: Path,
    selected_ai: str,
    tracker: StepTracker = None
) -> bool:
    """安装 agentskills.io 规范的 SKILL.md 文件"""
```

**实现细节**：
- 遵循 [agentskills.io](https://agentskills.io/specification) 规范
- 从命令模板生成 SKILL.md
- 支持增量安装（不覆盖已存在的 skills）
- 使用 `SKILL_DESCRIPTIONS` 提供增强描述

**Skills 目录解析**：
```python
AGENT_SKILLS_DIR_OVERRIDES = {
    "codex": ".agents/skills",  # Codex 特殊布局
}

def _get_skills_dir(project_path: Path, selected_ai: str) -> Path:
    # 1. 检查 AGENT_SKILLS_DIR_OVERRIDES
    # 2. 使用 AGENT_CONFIG[agent]["folder"] + "skills"
    # 3. 回退到 DEFAULT_SKILLS_DIR (.agents/skills)
```

### save_init_options() / load_init_options() - 初始化选项持久化

```python
INIT_OPTIONS_FILE = ".specify/init-options.json"

def save_init_options(project_path: Path, options: dict) -> None:
    """保存 specify init 使用的 CLI 选项"""

def load_init_options(project_path: Path) -> dict:
    """加载之前保存的初始化选项"""
```

**用途**：
- 预设安装时判断是否启用了 `--ai-skills`
- 后续操作可适配初始化时的配置

## CLI 命令

### init 命令

```python
@app.command()
def init(
    project_name: str,
    ai_assistant: str = typer.Option(None, "--ai"),
    ai_commands_dir: str = typer.Option(None, "--ai-commands-dir"),
    script_type: str = typer.Option(None, "--script"),
    ai_skills: bool = typer.Option(False, "--ai-skills"),
    preset: List[str] = typer.Option(None, "--preset"),
    # ... 更多选项
):
```

**初始化流程**：
1. 验证参数和 Agent 工具
2. 下载并提取模板
3. 设置脚本执行权限
4. 初始化 Git 仓库
5. 安装 Agent Skills（如果启用）
6. 安装预设（如果指定）
7. 保存初始化选项

**Agent 迁移处理**：
```python
AGENT_SKILLS_MIGRATIONS = {
    "agy": {"error": "...", "usage": "..."},
    "codex": {"error": "...", "usage": "..."},
}
```

某些 Agent 已弃用显式命令支持，必须使用 `--ai-skills`。

### check 命令

```python
@app.command()
def check():
    """检查所有必需工具是否已安装"""
```

遍历 `AGENT_CONFIG`，检查每个 Agent 的 CLI 工具可用性。

### version 命令

```python
@app.command()
def version():
    """显示版本和系统信息"""
```

显示 CLI 版本、模板版本、Python 版本、平台信息等。

## 扩展和预设命令

模块还包含完整的扩展和预设管理命令：

```python
extension_app = typer.Typer(name="extension")
preset_app = typer.Typer(name="preset")

# 扩展命令
# specify extension list/add/remove/search/info/enable/disable/set-priority

# 预设命令
# specify preset list/add/remove/search/info/resolve/enable/disable/set-priority
```

## 辅助类

### StepTracker - 进度跟踪器

```python
class StepTracker:
    """跟踪和渲染层级步骤，类似 Claude Code 树形输出"""
    
    def add(self, key: str, label: str): ...
    def start(self, key: str, detail: str = ""): ...
    def complete(self, key: str, detail: str = ""): ...
    def error(self, key: str, detail: str = ""): ...
    def skip(self, key: str, detail: str = ""): ...
    def render(self) -> Tree: ...
```

### BannerGroup - 自定义帮助组

```python
class BannerGroup(TyperGroup):
    """在帮助信息前显示 ASCII Banner"""
```

## 安全考虑

1. **GitHub Token 处理**：
   - 支持 `--github-token`、`GH_TOKEN`、`GITHUB_TOKEN`
   - Token 被清理（strip）后使用

2. **TLS 验证**：
   - 默认使用 `truststore` 进行 SSL 验证
   - `--skip-tls` 选项仅用于调试

3. **Agent 文件夹安全提示**：
   - 初始化后提醒用户考虑将 Agent 文件夹添加到 `.gitignore`
   - 防止凭证泄露

## 扩展点

添加新 Agent 时需要：
1. 在 `AGENT_CONFIG` 中添加配置
2. 更新 `--ai` 帮助文本
3. 如有特殊处理，添加到相应函数中
