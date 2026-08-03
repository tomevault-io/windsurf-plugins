---
trigger: always_on
description: > **定位**: Why / What / Where 的入口。不负责 How。
---

# NL2DSL — 项目导航中心 (Knowledge Hub)

> **定位**: Why / What / Where 的入口。不负责 How。
> 详细设计请查阅 `docs/` 目录对应文档。

---

## 1. Project Philosophy

NL2DSL 不是 NL2SQL 项目。它是一个 **Governance-Aware Semantic Query Engine**。

- **SQL 是实现细节**，DSL 是语义契约
- **Governance 是数据源真理**，不是事后补丁
- LLM 只负责语义理解，系统负责执行治理

### 五条核心原则

| 原则 | 含义 |
|------|------|
| **Governance First** | 所有查询必须经过权限注入、敏感字段脱敏、审计日志记录。安全不是可选项 |
| **Semantic First** | 指标、维度、术语必须语义层注册。禁止自由文本字段名，禁止发明未定义的指标 |
| **DSL First** | LLM 只生成结构化 DSL，不生成 SQL。DSL 可校验、可修正、可做权限控制 |
| **Explainability First** | 每个查询必须可解释：用了什么指标、什么过滤条件、为什么这样聚合 |
| **Security First** | SQL 执行前必须经过扫描。禁止 DELETE/UPDATE/DROP/UNION/注释注入/多语句 |

---

## 2. Project Architecture Overview

高层架构：

```
Natural Language
       ↓
Intent Recognition（意图识别）
       ↓
Query Planner（查询规划）
       ↓
DSL（领域特定语言）← 语义契约
       ↓
Validation（校验与修正）
       ↓
SQL Generation（SQL 构建）
       ↓
Execution（执行与扫描）
       ↓
Explanation（结果解释）
```

双层解耦：
- **Agent 层**（宏观编排）：意图识别 → 任务分解 → 子查询调度 → 结果聚合 → 自然语言解释
- **Graph 层**（微观执行）：单查询 DSL → 校验 → 权限注入 → SQL 构建 → 扫描 → 执行

详细架构 → [`docs/architecture/02-system-architecture.md`](docs/architecture/02-system-architecture.md)
LangGraph 节点流程 → [`docs/agent/31-langgraph-workflow.md`](docs/agent/31-langgraph-workflow.md)

---

## 3. Documentation Navigation

> 以下按主题组织。每个文档附一句职责说明。
> 接到任务时，先定位主题，再读对应文档，最后读代码。

### Architecture（架构设计）

| 文档 | 职责说明 |
|------|------|
| [`docs/architecture/01-overview.md`](docs/architecture/01-overview.md) | 项目背景、设计目标（可校验/可优化/可治理/可扩展）、技术选型总览 |
| [`docs/architecture/02-system-architecture.md`](docs/architecture/02-system-architecture.md) | 整体架构图、数据流、模块边界 |
| [`docs/architecture/03-sql-engine.md`](docs/architecture/03-sql-engine.md) | SQLAlchemy Core 构建、sqlglot 方言转换、Query Planner |
| [`docs/architecture/04-deployment.md`](docs/architecture/04-deployment.md) | Docker Compose 部署、环境变量、性能调优 |
| [`docs/architecture/sql-execution-design.md`](docs/architecture/sql-execution-design.md) | SQL 执行层：连接池、异步执行、结果返回 |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | 面向开发者的完整架构文档，含与传统 NL2SQL 区别、API 参考 |

### DSL & API（语义契约与接口）

| 文档 | 职责说明 |
|------|------|
| [`docs/api/20-dsl-spec.md`](docs/api/20-dsl-spec.md) | DSL Schema 定义：metrics/dimensions/filters/order_by/limit 字段规范 |
| [`docs/api/21-api-contract.md`](docs/api/21-api-contract.md) | HTTP API 接口定义：/query /query/dsl /query/execute 请求/响应格式 |
| [`docs/api/22-error-handling.md`](docs/api/22-error-handling.md) | 错误分类、HTTP 状态码、歧义响应格式 |
| [`docs/business/11-dsl-validation.md`](docs/business/11-dsl-validation.md) | DSL 校验规则、风险控制层级、SQL 执行前正则扫描 |

### Business Layer（业务语义层）

| 文档 | 职责说明 |
|------|------|
| [`docs/business/10-semantic-layer.md`](docs/business/10-semantic-layer.md) | YAML 指标/维度注册、value_map 枚举映射、数据血缘与 Join 推导 |
| [`docs/business/12-permission.md`](docs/business/12-permission.md) | 行级权限注入规则、列级敏感字段黑名单、数据脱敏策略 |
| [`docs/business/13-business-rules.md`](docs/business/13-business-rules.md) | 术语表映射、Prompt 显式注入、歧义反问机制、时间语义处理 |

### Query Processing（查询处理）

| 文档 | 职责说明 |
|------|------|
| [`docs/query/query-clarification-design.md`](docs/query/query-clarification-design.md) | 歧义检测：时间缺失、指标歧义、维度歧义、比较基准歧义 |
| [`docs/query/query-sandbox-design.md`](docs/query/query-sandbox-design.md) | 查询沙箱：EXPLAIN 预估、LIMIT 预览、执行超时检测 |

### Agent & LangGraph（智能编排与执行管道）

| 文档 | 职责说明 |
|------|------|
| [`docs/agent/30-rag-design.md`](docs/agent/30-rag-design.md) | 向量检索：4 集合设计、混合检索策略、Milvus 集合结构 |
| [`docs/agent/31-langgraph-workflow.md`](docs/agent/31-langgraph-workflow.md) | StateGraph 完整节点流程图、条件分支、自检重试、链路追踪 |
| [`docs/agent/32-metadata-sync.md`](docs/agent/32-metadata-sync.md) | 元数据提取、向量库同步、增量更新策略 |
| [`docs/agent/33-testing.md`](docs/agent/33-testing.md) | 单元/集成/E2E 三层测试策略概览 |
| [`docs/agent/34-llm-risks.md`](docs/agent/34-llm-risks.md) | LLM 成本/延迟/幻觉/安全风险及缓解方案 |

### Planner（查询规划）

| 文档 | 职责说明 |
|------|------|
| [`docs/planner/query-optimization-design.md`](docs/planner/query-optimization-design.md) | 查询优化器：谓词下推、投影下推、Join 重排序（预留架构） |

### Optimizer（语义优化器）

| 文档 | 职责说明 |
|------|------|
| [`docs/specs/semantic-optimizer-architecture-v2.md`](docs/specs/semantic-optimizer-architecture-v2.md) | Optimizer 架构设计：Normalizer → Rule Engine → CanonicalResolver 三层管道 |
| [`docs/specs/semantic-optimizer-error-taxonomy-v2.md`](docs/specs/semantic-optimizer-error-taxonomy-v2.md) | 26 种错误类型分类体系（9 大类），含 Confidence 机制 |
| [`docs/specs/semantic-optimizer-implementation-plan-v2.md`](docs/specs/semantic-optimizer-implementation-plan-v2.md) | 5 Phase 实施计划（P0-P5）、里程碑、测试策略 |
| [`docs/evaluation/optimizer-guide.md`](docs/evaluation/optimizer-guide.md) | Optimizer 使用指南：CLI 参数、规则分类、报告解读 |

### Audit & Feedback（审计与反馈）

| 文档 | 职责说明 |
|------|------|
| [`docs/audit/audit-log-design.md`](docs/audit/audit-log-design.md) | 审计日志：数据模型、存储策略、查询接口、保留策略 |
| [`docs/feedback/feedback-loop-design.md`](docs/feedback/feedback-loop-design.md) | 反馈闭环：收集机制、存储格式、用于模型改进的流程 |

### Configuration（配置系统）

| 文档 | 职责说明 |
|------|------|
| [`docs/configuration/schema-reference.md`](docs/configuration/schema-reference.md) | 配置 Schema 参考：metrics/terms/intents/permissions/history 完整字段 |

### Evaluation（评测框架）

| 文档 | 职责说明 |
|------|------|
| [`docs/specs/evaluation-design.md`](docs/specs/evaluation-design.md) | 4 大类 12 维度量化评估框架设计 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kobegy248/nl2dsl-engine](https://github.com/kobegy248/nl2dsl-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
