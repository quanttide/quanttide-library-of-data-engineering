# Apache Atlas

> 企业级数据治理平台，重在血缘和分类

## 概览

| 项目 | 说明 |
|------|------|
| 官网 | https://atlas.apache.org/ |
| GitHub | https://github.com/apache/atlas |
| 协议 | Apache 2.0 |
| 开发语言 | Java |
| 存储 | HBase + Solr / JanusGraph |

## 核心能力

- **数据分类**：基于标签的分类系统，支持自动分类和手动分类
- **数据血缘**：字段级血缘追踪，支持跨系统的全链路血缘
- **数据发现**：按类型、分类、标签等维度搜索和浏览
- **治理集成**：与 Hadoop 生态深度集成（Hive、HBase、Sqoop、Flink、Kafka 等）
- **安全与合规**：结合 Apache Ranger 实现基于元数据的访问控制

## 架构模式

```
钩子 (Hooks) → Atlas Server → JanusGraph (存储) / Solr (索引) → UI / REST API
```

- **强 schema + 钩子**：通过嵌入在数据系统中的钩子自动捕获元数据变更
- 预定义丰富的数据类型和分类体系，类 SQL 的搜索语法
- 事件通知通过 Kafka 分发

## 适用场景

已有 Hadoop / 大数据生态、需要严格分类体系和合规治理的企业。
