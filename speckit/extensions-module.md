# 扩展系统模块深度解析 (`extensions.py`)

## 模块概述

`src/specify_cli/extensions.py` 包含约 2000+ 行代码，实现完整的扩展生命周期管理：
- 扩展清单验证
- 扩展注册表管理
- 扩展安装/卸载
- 扩展目录管理
- 钩子系统
- 配置管理

## 异常层次

```python
class ExtensionError(Exception):
    """扩展相关错误的基类"""

class ValidationError(ExtensionError):
    """清单验证失败"""

class CompatibilityError(ExtensionError):
    """版本不兼容"""
```

## 核心类

### ExtensionManifest - 扩展清单

```python
class ExtensionManifest:
    """表示和验证扩展清单 (extension.yml)"""
    
    SCHEMA_VERSION = "1.0"
    REQUIRED_FIELDS = ["schema_version", "extension", "requires", "provides"]
```

#### 清单结构

```yaml
schema_version: "1.0"

extension:
  id: "jira"                    # 小写字母数字+连字符
  name: "Jira Integration"
  version: "1.0.0"              # 语义版本
  description: "..."

requires:
  speckit_version: ">=0.1.0,<2.0.0"

provides:
  commands:
    - name: "speckit.jira.sync"  # 格式: speckit.<ext>.<cmd>
      file: "commands/sync.md"
      description: "..."

hooks:
  after_tasks:
    command: "speckit.jira.sync"
    optional: true
    prompt: "Sync tasks to Jira?"
```

#### 验证规则

```python
def _validate(self):
    # 1. 检查必需字段
    # 2. 验证 schema_version == "1.0"
    # 3. 验证 extension.id 格式: ^[a-z0-9-]+$
    # 4. 验证语义版本
    # 5. 验证 requires.speckit_version
    # 6. 验证至少一个命令
    # 7. 验证命令名格式: ^speckit\.[a-z0-9-]+\.[a-z0-9-]+$
```

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
def requires_speckit_version(self) -> str: ...
@property
def commands(self) -> List[Dict]: ...
@property
def hooks(self) -> Dict[str, Any]: ...

def get_hash(self) -> str:
    """计算清单文件的 SHA256 哈希"""
```

### ExtensionRegistry - 扩展注册表

```python
class ExtensionRegistry:
    """管理已安装扩展的注册表"""
    
    REGISTRY_FILE = ".registry"
    SCHEMA_VERSION = "1.0"
```

#### 注册表结构

```json
{
  "schema_version": "1.0",
  "extensions": {
    "jira": {
      "version": "1.0.0",
      "source": "local",
      "manifest_hash": "sha256:...",
      "enabled": true,
      "priority": 10,
      "registered_commands": {"claude": ["speckit.jira.sync"]},
      "installed_at": "2024-01-01T00:00:00+00:00"
    }
  }
}
```

#### 核心方法

```python
def add(self, extension_id: str, metadata: dict):
    """添加扩展到注册表（自动添加 installed_at）"""

def update(self, extension_id: str, metadata: dict):
    """更新扩展元数据（保留 installed_at）"""

def restore(self, extension_id: str, metadata: dict):
    """恢复扩展元数据（用于回滚，保留原始时间戳）"""

def remove(self, extension_id: str):
    """从注册表移除扩展"""

def get(self, extension_id: str) -> Optional[dict]:
    """获取扩展元数据（返回深拷贝）"""

def list(self) -> Dict[str, dict]:
    """获取所有已安装扩展（返回深拷贝，过滤损坏条目）"""

def keys(self) -> set:
    """获取所有扩展 ID（轻量级，不深拷贝）"""

def is_installed(self, extension_id: str) -> bool:
    """检查扩展是否已安装"""

def list_by_priority(self, include_disabled: bool = False) -> List[tuple]:
    """按优先级排序获取扩展列表"""
```

**设计原则**：
- `get()` 和 `list()` 返回深拷贝，防止意外修改内部状态
- 损坏的注册表条目被过滤而非报错（优雅降级）
- 优先级数字越小，优先级越高

### ExtensionManager - 扩展管理器

```python
class ExtensionManager:
    """管理扩展生命周期：安装、卸载、更新"""
```

#### 安装流程

```python
def install_from_directory(
    self,
    source_dir: Path,
    speckit_version: str,
    register_commands: bool = True,
    priority: int = 10,
) -> ExtensionManifest:
    """从本地目录安装扩展"""
```

**安装步骤**：
1. 验证优先级（必须 >= 1）
2. 加载并验证清单
3. 检查版本兼容性
4. 检查是否已安装
5. 复制扩展文件（支持 `.extensionignore`）
6. 注册命令到 AI Agent
7. 注册钩子
8. 更新注册表

```python
def install_from_zip(
    self,
    zip_path: Path,
    speckit_version: str,
    priority: int = 10,
) -> ExtensionManifest:
    """从 ZIP 文件安装扩展"""
```

**安全措施**：
- Zip Slip 攻击防护：验证所有路径在目标目录内
- 支持嵌套目录结构

#### 卸载流程

```python
def remove(self, extension_id: str, keep_config: bool = False) -> bool:
    """卸载扩展
    
    Args:
        keep_config: 如果为 True，保留配置文件
    """
```

**卸载步骤**：
1. 获取已注册命令
2. 注销 AI Agent 命令
3. 备份配置文件（如果不保留）
4. 删除扩展目录
5. 注销钩子
6. 更新注册表

#### .extensionignore 支持

```python
@staticmethod
def _load_extensionignore(source_dir: Path):
    """加载 .extensionignore 文件，返回 shutil.copytree 兼容的忽略函数"""
```

使用 `pathspec.GitIgnoreSpec` 解析，支持 `.gitignore` 兼容的模式。

### ExtensionCatalog - 扩展目录

```python
class ExtensionCatalog:
    """管理扩展目录的获取、缓存和搜索"""
    
    DEFAULT_CATALOG_URL = "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.json"
    COMMUNITY_CATALOG_URL = "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.community.json"
    CACHE_DURATION = 3600  # 1 小时
```

#### 多目录堆栈

```python
@dataclass
class CatalogEntry:
    url: str
    name: str
    priority: int
    install_allowed: bool
    description: str = ""
```

**解析顺序**：
1. `SPECKIT_CATALOG_URL` 环境变量
2. 项目级 `.specify/extension-catalogs.yml`
3. 用户级 `~/.specify/extension-catalogs.yml`
4. 内置默认堆栈（default + community）

#### 目录配置格式

```yaml
# .specify/extension-catalogs.yml
catalogs:
  - name: "enterprise"
    url: "https://internal.company.com/catalog.json"
    priority: 1
    install_allowed: true
  - name: "community"
    url: "https://..."
    priority: 2
    install_allowed: false  # 仅发现，不允许安装
```

#### 搜索和下载

```python
def search(
    self,
    query: Optional[str] = None,
    tag: Optional[str] = None,
    author: Optional[str] = None,
    verified_only: bool = False,
) -> List[Dict[str, Any]]:
    """搜索扩展目录"""

def get_extension_info(self, extension_id: str) -> Optional[Dict]:
    """获取扩展详细信息"""

def download_extension(self, extension_id: str, target_dir: Path = None) -> Path:
    """下载扩展 ZIP 文件"""
```

**安全措施**：
- 目录 URL 必须使用 HTTPS（localhost 除外）
- 下载 URL 必须使用 HTTPS

### ConfigManager - 配置管理器

```python
class ConfigManager:
    """管理扩展的分层配置"""
```

#### 配置层次（优先级从低到高）

1. **默认值** - `extension.yml` 中的 `config.defaults`
2. **项目配置** - `.specify/extensions/{ext-id}/{ext-id}-config.yml`
3. **本地配置** - `.specify/extensions/{ext-id}/local-config.yml`（gitignored）
4. **环境变量** - `SPECKIT_{EXT_ID}_{SECTION}_{KEY}`

#### 使用示例

```python
config = ConfigManager(project_root, "jira")

# 获取完整配置
full_config = config.get_config()

# 获取特定值
url = config.get_value("connection.url")
timeout = config.get_value("connection.timeout", default=30)

# 检查值是否存在
if config.has_value("connection.api_key"):
    ...
```

### HookExecutor - 钩子执行器

```python
class HookExecutor:
    """管理扩展钩子的执行"""
```

#### 钩子配置文件

```yaml
# .specify/extensions.yml
installed: []
settings:
  auto_execute_hooks: true
hooks:
  after_tasks:
    - extension: "jira"
      command: "speckit.jira.sync"
      enabled: true
      optional: true
      prompt: "Sync tasks to Jira?"
      description: "..."
      condition: "config.connection.url is set"
```

#### 条件表达式

支持的条件模式：
- `config.key.path is set` - 检查配置值是否存在
- `config.key.path == 'value'` - 检查配置值相等
- `config.key.path != 'value'` - 检查配置值不等
- `env.VAR_NAME is set` - 检查环境变量是否存在
- `env.VAR_NAME == 'value'` - 检查环境变量值

#### 核心方法

```python
def register_hooks(self, manifest: ExtensionManifest):
    """注册扩展钩子到项目配置"""

def unregister_hooks(self, extension_id: str):
    """从项目配置移除扩展钩子"""

def get_hooks_for_event(self, event_name: str) -> List[Dict]:
    """获取特定事件的所有已注册钩子"""

def check_hooks_for_event(self, event_name: str) -> Dict[str, Any]:
    """检查事件的钩子（包含条件评估）
    
    Returns:
        {
            "has_hooks": bool,
            "hooks": List[Dict],
            "message": str  # 格式化的显示消息
        }
    """

def enable_hooks(self, extension_id: str):
    """启用扩展的所有钩子"""

def disable_hooks(self, extension_id: str):
    """禁用扩展的所有钩子"""
```

## 辅助函数

### normalize_priority()

```python
def normalize_priority(value: Any, default: int = 10) -> int:
    """规范化优先级值
    
    处理损坏的注册表数据（缺失、非数字、非正数）
    """
```

### version_satisfies()

```python
def version_satisfies(current: str, required: str) -> bool:
    """检查当前版本是否满足要求的版本规范"""
```

## 设计原则

1. **深拷贝返回** - Registry 的读取方法返回深拷贝，防止意外修改
2. **优雅降级** - 损坏的注册表条目被过滤而非报错
3. **安全优先** - HTTPS 强制、Zip Slip 防护、路径验证
4. **分层配置** - 支持默认值、项目、本地、环境变量多层覆盖
5. **可追溯性** - 命令文件包含来源注释

## 扩展点

1. 添加新的条件表达式模式到 `_evaluate_condition()`
2. 添加新的配置层到 `ConfigManager`
3. 扩展目录格式支持
