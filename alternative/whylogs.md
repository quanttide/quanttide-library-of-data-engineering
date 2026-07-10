# whylogs — 开源数据日志标准

## 概述

[whylogs](https://github.com/whylabs/whylogs) 是 WhyLabs 开源的数据日志库，号称"数据日志的开放标准"。它能够为任何数据集生成轻量级的统计摘要（称为 **whylogs profile**），用于数据质量监控、数据漂移检测、模型监控等场景。

| 项目 | 信息 |
|---|---|
| 仓库 | https://github.com/whylabs/whylogs |
| 文档 | https://whylogs.readthedocs.io/ |
| 许可 | Apache-2.0 |
| Stars | 2.8k |
| 语言 | Python（为主），Java |

## 核心能力

### whylogs Profile（数据摘要）

Profile 是 whylogs 的核心概念，它捕获数据的关键统计属性：

- 分布统计（均值、分位数、标准差等，远超简单的 mean/median）
- 缺失值计数
- 数据类型和 schema 信息
- 自定义指标（可扩展）

Profile 的三个关键特性：

1. **高效（Efficient）**：用很小的体积高保真地描述整个数据集，比采样更准确
2. **可定制（Customizable）**：可按数据类型和场景配置不同的统计追踪器
3. **可合并（Mergeable）**：多个 profile 可以合并为一个聚合 profile，支持分布式和流式系统的日志聚合

### 数据约束（Data Constraints）

基于 profile 可以设置数据约束，用于数据单元测试和 CI/CD 验证：

```python
import whylogs as why
from whylogs.core.constraints import Constraints, ConstraintsBuilder
from whylogs.core.constraints.factories import greater_than_number

profile_view = why.log(df).view()
builder = ConstraintsBuilder(profile_view)
builder.add_constraint(greater_than_number(column_name="col_name", number=0.15))
constraints = builder.build()
constraints.report()
```

### 可视化分析

通过 `whylogs[viz]` 模块可以在 Jupyter 中生成交互式报表，包括漂移报告、分布对比等。

### WhyLabs 平台集成

Profile 可以上传至 WhyLabs SaaS 平台进行自动化监控，免费 Starter 版即可体验全部功能。

## 支持的数据类型

| 数据类型 | 支持 |
|---|---|
| 表格数据（Tabular） | ✅ |
| 图片数据（Image） | ✅ |
| 文本数据（Text） | ✅ |
| 嵌入向量（Embeddings） | ✅ |

## 集成生态

whylogs 可以嵌入到主流数据管线和 ML 管线中：

- Apache Spark
- Apache Airflow
- AWS S3 / GCS
- Mlflow
- Ray

## 性能

whylogs 设计为在数据管线中旁路运行，计算开销极低（多数场景 < 1%）。大数据量下支持分布式并行计算——profile 的 map-reducible 特性使其能随特征数量而非行数扩展。

| 数据量 | 成本 | 实例 | 处理时间 |
|---|---|---|---|
| 10GB, 34M×43 列 | ~$0.026 | c5a.2xlarge × 2 | 2.6 分钟 |
| 10GB, 34M×43 列 | ~$0.016 | c6g.2xlarge × 2 | 1.7 分钟 |
| 80GB, 83M×119 列 | ~$0.139 | c5a.2xlarge × 16 | 1.7 分钟 |

## 适用场景

- **数据漂移检测**：监控模型输入特征的分布变化
- **训练-服务偏差检测**：发现训练环境和生产环境的数据差异
- **数据质量验证**：在数据管线中对数据集进行质量把关
- **探索性数据分析（EDA）**：对大规模数据集快速生成统计摘要
- **ML 实验追踪**：记录每次实验的数据分布和质量
- **数据审计与治理**：标准化数据文档和实践

## 快速开始

```bash
pip install whylogs
```

```python
import whylogs as why
import pandas as pd

df = pd.read_csv("path/to/file.csv")
results = why.log(df)

# 写入 WhyLabs
results.writer("whylabs").write()
```

## 总结

whylogs 是一个轻量、高效、可扩展的数据日志工具，填补了数据管线中"数据可观测性"的空白。它的 profile 设计兼顾了统计精度和计算效率，可合并特性使其天然适配分布式系统。对于需要数据质量监控和模型监控的团队，whylogs 是一个值得纳入技术栈的开源选择。
