# Spec Kit 产品文档

## 产品愿景

Spec Kit 是一个开源的规范驱动开发 (SDD) 工具包，核心理念是**颠覆传统开发模式**：

- 传统模式：代码是核心，规范是辅助文档
- SDD 模式：**规范是可执行的主要产物，代码是规范的具体实现**

## 核心价值主张

1. **意图驱动开发** - 用自然语言表达开发意图，AI 将其转化为可执行规范
2. **规范即真相** - 规范成为单一数据源，代码从规范生成
3. **持续一致性** - 规范、计划、任务、代码保持同步
4. **多 Agent 支持** - 支持 30+ 种 AI 编码助手

## SDD 工作流

```
用户想法 → /speckit.constitution → 项目原则
         → /speckit.specify     → 功能规范 (spec.md)
         → /speckit.clarify     → 澄清模糊需求 (可选)
         → /speckit.plan        → 技术计划 (plan.md + 设计文档)
         → /speckit.tasks       → 任务列表 (tasks.md)
         → /speckit.implement   → 代码实现
```

## 核心命令详解

### `/speckit.constitution` - 项目宪法

创建项目的核心原则和开发准则：
- 定义不可变的架构原则
- 设置质量门禁和约束
- 建立团队开发规范

### `/speckit.specify` - 功能规范

从自然语言描述生成结构化规范：
- 自动创建功能分支
- 生成用户故事和验收标准
- 定义功能需求和成功指标
- 最多 3 个 `[NEEDS CLARIFICATION]` 标记

**关键原则**：
- 聚焦 **WHAT**（用户需要什么）和 **WHY**（为什么需要）
- 避免 **HOW**（如何实现）- 不涉及技术栈

### `/speckit.plan` - 技术计划

将规范转化为技术实现计划：
- 填充技术上下文（语言、框架、存储等）
- 执行宪法检查（门禁验证）
- 生成研究文档 (research.md)
- 生成数据模型 (data-model.md)
- 生成接口契约 (contracts/)
- 生成快速验证指南 (quickstart.md)

### `/speckit.tasks` - 任务分解

将计划分解为可执行任务：
- 按用户故事组织任务
- 标记并行任务 `[P]`
- 定义任务依赖关系
- 支持 MVP 优先策略

**任务格式**：
```
- [ ] T001 [P] [US1] 在 src/models/user.py 创建 User 模型
```

### `/speckit.implement` - 执行实现

按任务列表执行代码实现：
- 检查清单完成状态
- 按阶段执行任务
- 遵循 TDD 方法（如果请求）
- 自动标记完成的任务

## 扩展系统

扩展为 Spec Kit 添加新功能，不修改核心：

**社区扩展示例**：
- `jira` - Jira 集成，同步任务到工作项
- `azure-devops` - Azure DevOps 集成
- `review` - 代码审查扩展
- `verify` - 实现验证扩展
- `doctor` - 项目健康检查

**扩展命令格式**：`/speckit.<extension-id>.<command>`

## 预设系统

预设覆盖核心模板和命令，自定义工作流：

**解析优先级**（从高到低）：
1. `.specify/templates/overrides/` - 项目本地覆盖
2. `.specify/presets/<id>/templates/` - 已安装预设
3. `.specify/extensions/<id>/templates/` - 扩展模板
4. `.specify/templates/` - 核心模板

**预设用例**：
- 企业合规模板
- 行业特定工作流（医疗、金融）
- 团队定制规范格式

## 钩子系统

扩展可在工作流生命周期注入逻辑：

**支持的事件**：
- `before_specify` / `after_specify`
- `before_plan` / `after_plan`
- `before_tasks` / `after_tasks`
- `before_implement` / `after_implement`

**钩子类型**：
- `optional: true` - 提示用户是否执行
- `optional: false` - 自动执行

## 目标用户

1. **开发团队** - 需要结构化开发流程
2. **技术负责人** - 需要可追溯的技术决策
3. **产品经理** - 需要清晰的需求文档
4. **AI 辅助开发者** - 使用各种 AI 编码助手

## 成功指标

- 规范到实现的一致性
- 开发周期缩短
- 技术债务减少
- 团队协作效率提升
