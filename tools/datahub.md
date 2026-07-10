# DataHub

> 实时元数据平台，事件驱动

## 概览

| 项目 | 说明 |
|------|------|
| 官网 | https://datahubproject.io/ |
| GitHub | https://github.com/datahub-project/datahub |
| 协议 | Apache 2.0 |
| 开发语言 | Java + Python (元数据服务) / TypeScript (前端) |
| 存储 | MySQL / PostgreSQL + Elasticsearch + Kafka |

## 核心能力

- **实时元数据**：基于 Kafka 事件流的元数据变更通知
- **数据血缘**：字段级血缘，自动解析 SQL + 手动补充
- **数据发现**：全文搜索 + 过滤（类型、平台、标签、域）
- **数据治理**：数据域、分类、Owner、文档、Glossary
- **质量集成**：与 Great Expectations / dbt 测试结果联动
- **自动化**：Actions 框架 + Webhook，元数据变更触发下游流程

## 架构模式

```
Streaming Ingestion (Kafka) → Metadata Service (GMS) → Elasticsearch (搜索) + MySQL (存储) → UI / GraphQL API
```

- **流式 ingestion**：通过 Kafka 事件总线实时推送元数据变更
- 支持 Rest.li API 和 GraphQL 两种查询接口
- Metadata Change Log (MCL) 可供下游消费
- Actions 框架支持元数据变更触发自动化流程

## 适用场景

数据基础设施完善、要求元数据实时更新、需要事件驱动自动化的团队。
