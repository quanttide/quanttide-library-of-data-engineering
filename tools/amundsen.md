# Amundsen

> 数据搜索与发现引擎，"找数据" 优先

## 概览

| 项目 | 说明 |
|------|------|
| 官网 | https://www.amundsen.io/ |
| GitHub | https://github.com/amundsen-io/amundsen |
| 协议 | Apache 2.0 |
| 开发语言 | Python (后端) / TypeScript (前端) |
| 存储 | Neo4j + Elasticsearch + AtlasDB |

## 核心能力

- **数据发现**：搜索优先的交互设计，Google-like 搜索体验
- **数据血缘**：表级血缘（非字段级），展示上下游依赖
- **数据目录**：表、列、分区、Owner、描述等元数据展示
- **使用统计**：数据热度、查询频次等（需集成）
- **Badge/标签**：标记数据质量等级、PII 状态等

## 架构模式

```
Databuilder (ETL ingestion) → Neo4j (图存储) + ES (全文索引) → Frontend Service / Search Service → UI
```

- **基于索引**：元数据通过 Databuilder 周期性地抽取并写入 Neo4j 和 Elasticsearch
- 搜索全部走 ES，图查询走 Neo4j
- 不强调实时性，适合"先找到数据"的使用场景

## 适用场景

团队规模较大、数据源多、成员主要靠搜索定位数据的组织。
