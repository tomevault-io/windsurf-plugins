---
trigger: always_on
description: 本文件是本仓库后续所有开发者和 AI Agent 的强制执行规则。任何实现、测试、文档和重构都必须遵守。需求已对齐；未获得项目负责人明确确认，不得偏离本文件。
---

# AI 评测自迭代工作台：强制项目规则

本文件是本仓库后续所有开发者和 AI Agent 的强制执行规则。任何实现、测试、文档和重构都必须遵守。需求已对齐；未获得项目负责人明确确认，不得偏离本文件。

## 1. 技术栈锁定

| 层级 | 唯一技术 |
| --- | --- |
| 前端 | Next.js App Router + TypeScript + shadcn/ui + ECharts |
| 后端 | FastAPI + Python 业务服务 |
| 数据访问与迁移 | SQLAlchemy + Alembic |
| 数据库 | SQLite |
| 定时调度 | APScheduler |
| 模型调用 | LiteLLM，且全部调用只经 `backend/app/services/llm_client.py` |
| 后端测试 | pytest |
| 端到端测试 | Playwright |
| 正式质量告警 | 飞书 Webhook |

后端仅监听 `127.0.0.1:8000`，固定使用一个 Uvicorn worker。前端仅运行于 `127.0.0.1:3000`。模拟客服 API 仅运行于 `127.0.0.1:8001`。

## 2. 禁止擅自引入的同类技术

禁止引入 Django、Flask、Starlette 独立应用、Node.js 后端、NestJS、Express、Spring、Go 服务、Celery、Redis、RabbitMQ、Kafka、PostgreSQL、MySQL、MongoDB、ORM 替代品、Prisma、Drizzle、SQLModel、其他任务调度器、其他图表库、其他前端组件库、其他端到端测试框架、Docker 编排、多进程部署、Gunicorn、多 Uvicorn worker。

禁止在 `llm_client.py` 之外导入 LiteLLM 或任何模型供应商 SDK。禁止提供第二套模型调用路径、第二个数据库、第二种 JSON 字段命名或第二套状态命名。

## 3. 项目目录结构

```text
backend/
  app/
    api/                 # FastAPI 路由、请求和响应序列化
    db/                  # SQLAlchemy engine、session、模型
    domain/              # 枚举、Pydantic 契约、常量
    repositories/        # 数据读写和短事务
    services/            # 业务服务
    jobs/                # APScheduler 作业注册与调度入口
    main.py              # FastAPI 应用和启动恢复
  alembic/               # 迁移脚本
  tests/                 # pytest
frontend/
  app/                   # Next.js App Router 页面
  components/            # shadcn/ui 组件和 ECharts 图表
  lib/                   # API client 和前端工具
  tests/e2e/             # Playwright
simulator-api/
  app/                   # 模拟客服 API
  data/                  # 100 条固定演示会话
docs/
  requirements/          # 权威 PRD
  superpowers/           # 设计与实施计划
  acceptance/            # 三层验收证据
```

## 4. 模块与文件职责

| 模块 | 唯一文件或目录职责 |
| --- | --- |
| API | `backend/app/api/` 只处理 REST 通信、输入校验、响应序列化和可读错误；不得承载业务编排。 |
| Service | `backend/app/services/` 只承载领域业务；一个服务文件只负责一个领域。 |
| Client | `backend/app/services/source_client.py` 只调用客服数据 API；`backend/app/services/llm_client.py` 是全部模型能力的唯一网关。 |
| Job | `backend/app/jobs/collection_job.py` 只注册和执行定时采集；调度器初始化只位于 `backend/app/jobs/scheduler.py`。 |
| Repository | `backend/app/repositories/` 只执行查询、写入和短事务；不得调用 HTTP、模型或调度器。 |
| 数据库 | `backend/app/db/` 只定义 engine、session 和 SQLAlchemy 表模型；迁移只能位于 `backend/alembic/`。 |
| 前端 API | `frontend/lib/api.ts` 是前端访问 FastAPI 的唯一入口。 |
| 模拟 API | `simulator-api/app/main.py` 只模拟外部客服会话 API；不得访问工作台 SQLite。 |

## 5. 硬性规则（8 条）

1. 所有模型调用必须且只能由 `llm_client.py` 发起；评测、评分标准草稿、归因、QA 草稿和模拟模型均无例外。
2. 采集顺序固定为：过滤 → 按稳定会话 ID 比例抽样 → 按发生时间排序 → 截取单次上限；不得调整。
3. 单条模型调用最多尝试 10 次；单条失败、解析异常或结果不完整不得中断整批。
4. 仅评测结果 `status="success"` 且 `pass=false` 可成为 Badcase；链路异常只进入异常率。
5. 所有指标复用同一公式：通过率仅以成功评测为分母；平均分仅统计成功评测；异常率以三类链路异常除以评测总量。
6. QA 产物状态只能按 `draft -> approved -> exported` 流转；未批准不可导出，批准后不可编辑，导出是终态。
7. 后端、模拟 API、前端仅监听本机回环地址；禁止公网或局域网监听，禁止多 worker、多进程和横向扩容。
8. 密钥、Token、飞书 Webhook 和模型原始敏感配置不得写入 SQLite、日志、接口响应、前端代码或导出文件。

## 6. 不可变约定

### 6.1 数据库表名

以下表名固定，迁移、模型、查询和外键不得改名：

| 表名 | 主键 | 用途 |
| --- | --- | --- |
| `data_source_configs` | `id` | 数据源配置 |
| `collection_runs` | `id` | 每次采集运行 |
| `collection_record_errors` | `id` | 采集单条映射失败 |
| `conversations` | `id` | 当前原始会话；`source_conversation_id` 全局唯一 |
| `evaluation_tasks` | `id` | 评测任务与当前标准 |
| `evaluation_runs` | `id` | 批量评测运行 |
| `evaluation_results` | `id` | 单条评测结果及快照 |
| `alert_rules` | `id` | 质量告警规则 |
| `alert_records` | `id` | 飞书或本地模拟通知记录 |
| `badcase_analyses` | `id` | Badcase AI 归因和人工复核 |
| `improvement_artifacts` | `id` | QA 草稿、审核和导出状态 |
| `export_batches` | `id` | 每次 CSV/JSONL 导出 |
| `export_batch_items` | `id` | 导出批次与产物关联 |
| `pipeline_runs` | `id` | 一键流水线运行 |
| `pipeline_stage_runs` | `id` | 流水线阶段状态和摘要 |
| `app_settings` | `key` | 单用户操作人和模型模式 |

固定表名数量：16。

### 6.2 跨阶段主键和关联字段

| 上游 | 下游 | 固定关联字段 |
| --- | --- | --- |
| `data_source_configs` | `collection_runs` | `collection_runs.data_source_config_id` |
| `collection_runs` | `collection_record_errors` | `collection_record_errors.collection_run_id` |
| `data_source_configs` | `conversations` | `conversations.data_source_config_id` |
| `evaluation_tasks` | `evaluation_runs` | `evaluation_runs.evaluation_task_id` |
| `evaluation_runs` | `evaluation_results` | `evaluation_results.evaluation_run_id` |
| `conversations` | `evaluation_results` | `evaluation_results.conversation_id` |
| `evaluation_results` | `badcase_analyses` | `badcase_analyses.evaluation_result_id`，唯一 |
| `badcase_analyses` | `improvement_artifacts` | `improvement_artifacts.badcase_analysis_id`，唯一 |
| `export_batches` | `export_batch_items` | `export_batch_items.export_batch_id` |
| `improvement_artifacts` | `export_batch_items` | `export_batch_items.improvement_artifact_id` |
| `data_source_configs` | `pipeline_runs` | `pipeline_runs.data_source_config_id` |
| `evaluation_tasks` | `pipeline_runs` | `pipeline_runs.evaluation_task_id` |
| `pipeline_runs` | `pipeline_stage_runs` | `pipeline_stage_runs.pipeline_run_id` |

### 6.3 API 与前端 JSON 命名

后端 API、前端 `frontend/lib/api.ts`、Pydantic 模型和导出字段之外的所有 JSON 一律使用 `snake_case`。数据库列名与 JSON 字段名一致。禁止 camelCase、PascalCase、中文键名和同义字段。

五个固定采集映射键只能是：

```text
conversation_id
occurred_at
user_query
ai_response
context
```

`channel`、`transferred_to_human`、`order_info` 是固定可选扩展字段，不属于五个映射键，且不得替代任一映射键。

`context` 中每条消息的角色枚举固定为：

```text
user
assistant
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DDD123-LIU/ai-evaluation-iteration-workbench](https://github.com/DDD123-LIU/ai-evaluation-iteration-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
