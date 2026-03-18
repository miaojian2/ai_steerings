---
inclusion: manual
---

# 记忆系统 (src/memory/)

## 概述

记忆系统提供向量搜索和全文搜索能力，支持 Agent 检索历史对话、文档和工作区文件。

## 目录结构

```
src/memory/
├── manager.ts             # 主管理器
├── manager-*.ts           # 管理器子模块
├── embeddings.ts          # 嵌入向量生成
├── embeddings-*.ts        # 各提供商实现
├── hybrid.ts              # 混合搜索
├── sqlite.ts              # SQLite 存储
├── sqlite-vec.ts          # 向量扩展
├── types.ts               # 类型定义
├── batch-*.ts             # 批量处理
├── qmd-*.ts               # QMD 查询
└── *.test.ts              # 测试文件
```

## 核心组件

### 记忆管理器 (manager.ts)

```typescript
import { MemoryIndexManager } from "./manager.js";

// 获取管理器实例（单例缓存）
const manager = await MemoryIndexManager.get({
  cfg: config,
  agentId: "default"
});

// 搜索
const results = await manager.search("API authentication", {
  maxResults: 10,
  minScore: 0.3
});

// 同步索引
await manager.sync({ reason: "manual" });

// 获取状态
const status = manager.status();

// 关闭
await manager.close();
```

### 搜索结果

```typescript
type MemorySearchResult = {
  path: string;        // 文件路径
  source: MemorySource; // 来源类型
  snippet: string;     // 匹配片段
  score: number;       // 相关性分数
  startLine: number;
  endLine: number;
};

type MemorySource = "memory" | "session" | "workspace";
```

## 嵌入向量 (embeddings.ts)

### 提供商

```typescript
// 支持的提供商
type EmbeddingProvider = "openai" | "gemini" | "voyage" | "mistral" | "ollama" | "local";

// 创建提供商
const provider = await createEmbeddingProvider({
  config: cfg,
  provider: "openai",
  model: "text-embedding-3-small"
});

// 生成嵌入
const vectors = await provider.embed(["Hello world"]);
```

### 各提供商实现

- `embeddings-openai.ts` - OpenAI
- `embeddings-gemini.ts` - Google Gemini
- `embeddings-voyage.ts` - Voyage AI
- `embeddings-mistral.ts` - Mistral
- `embeddings-ollama.ts` - Ollama (本地)

## 混合搜索 (hybrid.ts)

结合向量搜索和全文搜索：

```typescript
import { mergeHybridResults, buildFtsQuery } from "./hybrid.js";

// 合并结果
const merged = await mergeHybridResults({
  vector: vectorResults,
  keyword: keywordResults,
  vectorWeight: 0.7,
  textWeight: 0.3,
  mmr: { enabled: true, lambda: 0.5 },
  temporalDecay: { enabled: true, halfLifeDays: 30 }
});
```

### MMR (Maximal Marginal Relevance)

减少结果冗余，增加多样性。

### 时间衰减

较新的内容获得更高权重。

## 存储层

### SQLite (sqlite.ts)

```typescript
// 表结构
// - files: 索引的文件
// - chunks: 文本块
// - chunks_vec: 向量索引
// - chunks_fts: 全文索引
// - embedding_cache: 嵌入缓存
```

### 向量扩展 (sqlite-vec.ts)

使用 `sqlite-vec` 扩展进行向量相似度搜索。

## 批量处理 (batch-*.ts)

支持批量嵌入生成：

```typescript
// batch-runner.ts - 批量任务运行
// batch-openai.ts - OpenAI 批量 API
// batch-gemini.ts - Gemini 批量 API
// batch-voyage.ts - Voyage 批量 API
```

## 配置

```json
{
  "agents": {
    "defaults": {
      "memory": {
        "enabled": true,
        "provider": "openai",
        "model": "text-embedding-3-small",
        "sources": ["memory", "session"],
        "query": {
          "maxResults": 10,
          "minScore": 0.3,
          "hybrid": {
            "enabled": true,
            "vectorWeight": 0.7,
            "textWeight": 0.3
          }
        }
      }
    }
  }
}
```

## 数据源

- `memory`: 工作区 `.memory/` 目录下的 Markdown 文件
- `session`: 会话历史记录
- `workspace`: 工作区其他文件

## 阅读建议

1. 从 `manager.ts` 了解主要 API
2. 阅读 `embeddings.ts` 理解嵌入生成
3. 查看 `hybrid.ts` 了解混合搜索
4. 研究 `sqlite.ts` 了解存储结构
5. 查看 `types.ts` 了解数据类型
