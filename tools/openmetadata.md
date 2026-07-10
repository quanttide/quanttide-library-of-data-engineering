# OpenMetadata

> 全栈元数据管理平台：数据资产 + 血缘 + 治理 + 发现

## 概览

| 项目 | 说明 |
|------|------|
| 官网 | https://open-metadata.org/ |
| GitHub | https://github.com/open-metadata/OpenMetadata |
| 协议 | Apache 2.0 |
| 开发语言 | Java (后端) / TypeScript (前端) |
| 存储 | MySQL / PostgreSQL + Elasticsearch |

## 核心能力

- **数据发现**：统一搜索，按表/视图/管道/仪表板等类型过滤
- **数据血缘**：自动解析 SQL 查询构建字段级血缘
- **数据治理**：分类/标签、数据域、Owner 管理、PII 标记
- **质量与 profiler**：内置数据 profiler 和测试框架
- **协作**：文档、注解、团队/角色权限
- **Ingestion**：100+ 连接器（数据库、数仓、BI、数据管道等）

## 架构模式

```
Connectors (Airflow / 内置调度) → Ingestion Framework → OpenMetadata Server → UI / API
```

- **自动 ingestion + 手动补充**：连接器自动抽取元数据，用户可手工补充描述、标签、Owner
- **支持轮询和事件驱动两种 ingestion 模式**
- 所有元数据变更通过 Webhook 或事件流向外推送

## 适用场景

需要统一平台管理数据资产全生命周期、兼顾自动发现和人工治理的团队。
