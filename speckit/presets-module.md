# 预设系统模块深度解析 (`presets.py`)

## 模块概述

`src/specify_cli/presets.py` 包含约 1700+ 行代码，实现预设的完整生命周期管理：
- 预设清单验证
- 预设注册表管理
- 预设安装/卸载
- 预设目录管理
- 模板解析（优先级堆栈）
- Skills 覆盖与恢复

## 异常层次

```python
class PresetError(Exception):
    """预设相关错误的基类"""

class PresetValidationError(PresetError):
    """清单验证失败"""

class PresetCompatibilityError(PresetError):
    """版本不兼容"""
```

## 核心类

### PresetManifest - 预设清单

```python
class PresetManifest:
    SCHEMA_VERSION = "1.0"
    REQUIRED_FIELDS = ["schema_version", "preset", "requires", "provides"]
```

#### 清单结构

```yaml
schema_version: "1.0"

preset:
  id: "scaffold"              # 小写字母数字+连字符
  name: "Scaffold Preset"
  version: "1.0.0"            # 语义版本
  description: "..."
  author: "github"
  repository: "https://..."
  license: "MIT"

requires:
  speckit_version: ">=0.1.0,<2.0.0"

provides:
  templates:
    - type: "template"         # template | command | script
      name: "spec-template"
      file: "templates/spec-template.md"
    - type: "command"
      name: "speckit.specify"
      file: "commands/speckit.specify.md"

tags:
  - enterprise
  - compliance
```

#### 模板类型

```python
VALID_PRESET_TEMPLATE_TYPES = {"template", "command", "script"}
```

| 类型 | 说明 | 命名格式 |
|------|------|----------|
| `template` | 工件模板（spec、plan 等） | `^[a-z0-9-]+$` |
| `command` | 命令覆盖 | `^[a-z0-9.-]+$`（点号分隔） |
| `script` | 脚本模板 | `^[a-z0-9-]+$` |

#### 验证规则

- 必需字段：`schema_version`、`preset`、`requires`、`provides`
- 预设 ID 格式：`^[a-z0-9-]+$`
- 语义版本验证
- 至少一个模板
- 模板文件路径安全检查（禁止绝对路径和 `..` 遍历）
- 命令名格式验证

#### 属性访问

```python
@property
def id(self) -> str: ...
@property
def name(self) -> str: ...
@property
def version(self) -> str: ...
@property
def description(self) -> str: ...
@property
def author(self) -> str: ...
@property
def requires_speckit_version(self) -> str: ...
@property
def templates(self) -> List[Dict]: ...
@property
def tags(self) -> List[str]: ...

def get_hash(self) -> str:
    """计算清单文件的 SHA256 哈希"""
```

### PresetRegistry - 预设注册表

```python
class PresetRegistry:
    REGISTRY_FILE = ".registry"
    SCHEMA_VERSION = "1.0"
```

#### 注册表结构

```json
{
  "schema_version": "1.0",
  "presets": {
    "scaffold": {
      "version": "1.0.0",
      "source": "local",
      "manifest_hash": "sha256:...",
      "enabled": true,
      "priority": 10,
      "registered_commands": {"claude": ["speckit.specify"]},
      "registered_skills": ["speckit-specify"],
      "installed_at": "2024-01-01T00:00:00+00:00"
    }
  }
}
```

#### 核心方法

与 `ExtensionRegistry` 设计一致：

```python
def add(self, pack_id: str, metadata: dict): ...
def remove(self, pack_id: str): ...
def update(self, pack_id: str, updates: dict): ...
def restore(self, pack_id: str, metadata: dict): ...
def get(self, pack_id: str) -> Optional[dict]: ...
def list(self) -> Dict[str, dict]: ...
def keys(self) -> set: ...
def is_installed(self, pack_id: str) -> bool: ...
def list_by_priority(self, include_disabled: bool = False) -> List[tuple]: ...
```

**设计原则**：
- 深拷贝返回，防止意外修改
- 损坏条目过滤（优雅降级）
- `update()` 保留原始 `installed_at`
- `restore()` 用于回滚场景

### PresetManager - 预设管理器

```python
class PresetManager:
    """管理预设生命周期：安装、卸载、更新"""
```

#### 安装流程

```python
def install_from_directory(
    self,
    source_dir: Path,
    speckit_version: str,
    priority: int = 10,
) -> PresetManifest:
```

**安装步骤**：
1. 验证优先级（>= 1）
2. 加载并验证清单
3. 检查版本兼容性
4. 检查是否已安装
5. 复制预设文件
6. 注册命令覆盖（`_register_commands`）
7. 更新 Skills（`_register_skills`）
8. 更新注册表

#### 命令注册

```python
def _register_commands(
    self,
    manifest: PresetManifest,
    preset_dir: Path
) -> Dict[str, List[str]]:
    """注册预设命令覆盖到所有检测到的 AI Agent"""
```

**关键逻辑**：
- 只注册 `type: "command"` 的模板
- 过滤未安装扩展的命令覆盖
- 通过 `CommandRegistrar` 委托注册

#### Skills 注册与恢复

```python
def _register_skills(
    self,
    manifest: PresetManifest,
    preset_dir: Path,
) -> List[str]:
    """为预设命令覆盖生成 SKILL.md 文件"""
```

**工作原理**：
- 检查 `--ai-skills` 是否在初始化时启用
- 只覆盖已存在的 skill 目录
- 使用 `SKILL_DESCRIPTIONS` 提供增强描述

```python
def _unregister_skills(
    self,
    skill_names: List[str],
    preset_dir: Path
) -> None:
    """预设卸载后恢复原始 SKILL.md"""
```

**恢复逻辑**：
1. 查找核心命令模板
2. 如果存在，从核心模板重新生成 SKILL.md
3. 如果不存在，删除 skill 目录

#### 卸载流程

```python
def remove(self, pack_id: str) -> bool:
```

**卸载步骤**：
1. 注销 AI Agent 命令
2. 恢复原始 Skills
3. 删除预设目录
4. 更新注册表

### PresetCatalog - 预设目录

```python
class PresetCatalog:
    DEFAULT_CATALOG_URL = "https://raw.githubusercontent.com/.../presets/catalog.json"
    COMMUNITY_CATALOG_URL = "https://raw.githubusercontent.com/.../presets/catalog.community.json"
    CACHE_DURATION = 3600  # 1 小时
```

#### 多目录堆栈

与 `ExtensionCatalog` 设计一致：

**解析顺序**：
1. `SPECKIT_PRESET_CATALOG_URL` 环境变量
2. 项目级 `.specify/preset-catalogs.yml`
3. 用户级 `~/.specify/preset-catalogs.yml`
4. 内置默认堆栈

#### 核心方法

```python
def get_active_catalogs(self) -> List[PresetCatalogEntry]: ...
def fetch_catalog(self, force_refresh: bool = False) -> Dict: ...
def search(self, query=None, tag=None, author=None) -> List[Dict]: ...
def get_pack_info(self, pack_id: str) -> Optional[Dict]: ...
def download_pack(self, pack_id: str, target_dir=None) -> Path: ...
def clear_cache(self): ...
```

**安全措施**：
- HTTPS 强制（localhost 除外）
- 下载 URL 验证
- `install_allowed` 标志控制

### PresetResolver - 模板解析器

```python
class PresetResolver:
    """使用优先级堆栈解析模板名到文件路径"""
```

#### 解析优先级

```
1. .specify/templates/overrides/          ← 项目本地覆盖（最高）
2. .specify/presets/<preset-id>/          ← 已安装预设
3. .specify/extensions/<ext-id>/templates/ ← 扩展提供的模板
4. .specify/templates/                    ← 核心模板（最低）
```

#### 核心方法

```python
def resolve(
    self,
    template_name: str,
    template_type: str = "template",
) -> Optional[Path]:
    """解析模板名到文件路径
    
    Args:
        template_name: 模板名（如 "spec-template"）
        template_type: "template" | "command" | "script"
    """
```

**解析逻辑**：
1. 根据类型确定子目录和扩展名
2. 检查 overrides 目录
3. 遍历已安装预设（按优先级排序）
4. 遍历已安装扩展（按优先级排序）
5. 检查核心模板目录

```python
def resolve_with_source(
    self,
    template_name: str,
    template_type: str = "template",
) -> Optional[Dict]:
    """解析模板并返回来源信息
    
    Returns:
        {"path": Path, "source": "overrides|preset:id|extension:id|core"}
    """
```

#### 扩展优先级处理

```python
def _get_all_extensions_by_priority(self) -> list[tuple]:
    """构建统一的扩展列表（注册+未注册），按优先级排序
    
    Returns:
        [(priority, ext_id, metadata_or_none), ...]
    """
```

- 注册的扩展使用存储的优先级
- 未注册的目录使用隐式优先级 10
- 禁用的扩展被排除
- 按 `(priority, ext_id)` 排序确保确定性

## 与其他模块的关系

### 与 `__init__.py` 的集成

- `load_init_options()` 判断是否启用了 `--ai-skills`
- `_get_skills_dir()` 解析 skills 目录
- `SKILL_DESCRIPTIONS` 提供增强描述

### 与 `agents.py` 的集成

- `CommandRegistrar` 处理实际的命令注册/注销
- 预设命令覆盖通过 `register_commands_for_all_agents()` 注册

### 与 `extensions.py` 的集成

- 导入 `ExtensionRegistry` 用于模板解析时查询扩展
- 导入 `normalize_priority` 用于优先级规范化
- 命令覆盖过滤依赖扩展安装状态

## 设计原则

1. **优先级堆栈** - 清晰的模板解析顺序，支持多层覆盖
2. **增量覆盖** - 预设只覆盖它声明的模板，其余保持不变
3. **可恢复性** - 卸载预设后恢复原始 Skills
4. **扩展感知** - 命令覆盖自动过滤未安装扩展的命令
5. **与扩展系统对称** - Registry/Manager/Catalog 设计模式一致
