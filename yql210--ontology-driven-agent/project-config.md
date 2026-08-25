---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

> **For Hermes:** 本项目仅使用 Codex 作为本地代码执行器。运行 `codex exec --json`，解析 `thread.started`、`agent_message`、`turn.completed` 和 `turn.failed`。运行时将 `/opt/data/.env` 的实际 `CUSTOM_API_KEY` 注入 `OPENAI_API_KEY`；不得写入 key 或使用 `${CUSTOM_API_KEY}` 字面量。Codex 无额外 turn 参数；Hermes 管控范围、超时、重试及质量门禁。每个任务 2-5 分钟粒度。强制 TDD。详细规范见 `.codex/rules/` 目录。

## 项目简介

OntoAgent — 基于本体驱动的可更新知识图谱引擎。从源代码 + 文档自动构建知识图谱（Neo4j + ChromaDB），支持自然语言查询、变更影响分析、增量更新。上层 LangGraph ReAct Agent 编排查询与操作。

技术栈：Python 3.13+ · uv · tree-sitter（Python+Java）· Neo4j 5.x · ChromaDB · LangGraph · Click · ruff · pyright。

## 精确命令

```bash
# 包管理
uv add <package>              # 添加依赖（运行时）
uv add --dev <package>        # 添加开发依赖
uv sync                       # 同步环境

# 测试（markers: unit / integration / slow）
uv run pytest tests/ -v                              # 全量
uv run pytest tests/unit/ -v                         # 仅单元测试
uv run pytest -m "not integration" -v                # 跳过集成测试（无需 Neo4j）
uv run pytest tests/unit/test_schema.py::test_x -v   # 单个测试
uv run pytest tests/ --cov=ontoagent --cov-report=term-missing  # 覆盖率

# 静态检查 / 格式化 / 类型检查（提交前必须通过）
uv run ruff check src/ tests/        # 静态检查
uv run ruff check --fix src/ tests/  # 自动修复
uv run ruff format src/ tests/       # 自动格式化
uv run pyright src/                  # 类型检查

# CLI（入口 ontoagent = ontoagent.api.cli:main；配置从 .env / 环境变量读取）
uv run ontoagent build ./repo [--skip-semantic] [--skip-clustering] [--clear] [--verbose-build]
uv run ontoagent query "text" [-t function|class|...] [-n 10]
uv run ontoagent update ./repo [--since HEAD~1] [--dry-run] [--full-scan]   # 增量更新
uv run ontoagent migrate [--target <ver>]                                    # schema 迁移 / 回滚
uv run ontoagent ask "merge_node 被谁调用" | ask -i                          # LangGraph Agent 问答
uv run ontoagent serve [--transport stdio|http] [--port 8000]                # MCP Server
uv run ontoagent web [--host 0.0.0.0] [--port 8000] [--reload]               # FastAPI Web API
uv run ontoagent info | version
uv run ontoagent butler {serve|update|build|status}                          # 事件驱动知识管理引擎

# Git
git add -A && git commit -m "type: description"
```

## 架构（V3.4 四层架构 — 需要跨多个文件理解的全局图景）

```
Intent（意图层）    Agent 识别意图 → express_intent 工具路由到 Action
Control（控制层）   ActionExecutor · Shape 约束系统（ShapeEvaluator + DecisionFuser + ShapeRegistry）· 审批（ApprovalGate）
Capability（能力层）Function 注册表（通用+领域）· FunctionRunner（重试/熔断）
Semantic（语义层）  Schema（9 实体 13 关系）· GraphStore（Neo4j + ChromaDB）· Shape 规则（shapes.yaml）
```

**关键约束：每层只依赖下一层，不跨层不反向。** Action 只引用 Function 名；Function 通过 `graph_store` 操作语义层、通过 `Connector` 访问外部系统；Connector 只搬运数据不含业务逻辑。

### 意图 → 执行 的完整链路
1. Agent 收到自然语言 → prompt 中的 `trigger_hint` 列表匹配 `intent_type`（prompt 由 `intent_router.build_intent_prompt()` 从 `pipeline/ontology_actions.yaml` 自动生成）。
2. Agent 调用工具 `express_intent(intent_type, target, params)`（`agent/tools.py`）。
3. `ActionExecutor.execute()`（`execution/action_executor.py`）：查 `intent_map` → `_resolve_entity` → `_check_criteria`（Submission Criteria）→ 通过 `FunctionRunner` 同步执行对应 Function。
4. Function 经 `ActionContext` 注入 `graph_store` + `function_runner`，可链式调用其他 Function；写操作线性执行（分布式事务 SAGA/DAG 已在 v0.2 删除，单 Neo4j 场景下线性执行足够）。

### 各层落地位置（重构后目录结构）
| 层 | 目录 | 关键文件 |
|----|------|---------|
| Domain | `domain/` | `schema.py`、`exceptions.py`、`provenance.py` |
| Store | `store/` | `graph_store.py`（抽象）、`neo4j_store.py`、`chroma_store.py`、`schema_version.py`、`migrations/` |
| Parsing | `parsing/` | `parser/`（python/java/doc）、`extractor/`（relation、semantic） |
| Pipeline | `pipeline/` | `builder.py`、`builder_utils.py`、`semantic_linker.py`、`incremental_updater.py`、`change_detector.py`、`impact_propagator.py`、`aligner.py`、`module_clustering.py`、`ontology_actions.yaml` |
| Execution | `execution/` | `action_executor.py`、`action_types.py`、`intent_router.py`、`function_runner.py`、`circuit_breaker.py`、`execution_policy.py`、`shape_evaluator.py`、`shape_registry.py`、`decision_fuser.py`、`path_compiler.py`、`constraints/`（approval_gate, policies）、`functions/` |
| Agent | `agent/` | `graph.py`、`tools.py`、`prompt.py`、`_helpers.py` |
| Butler | `butler/` | `engine.py`、`event_bus.py`、`scheduler.py`、`handlers/` |
| API | `api/` | `cli.py`、`mcp_server.py`、`web/` |

> 架构约束详见 `.codex/rules/architecture.md`（根目录文件上限 5 个、单文件行数上限 800、分层单向依赖）。

## 知识图谱构建流水线（`pipeline/builder.py::OntoAgentBuilder.build`）

多阶段管线，**只有前两阶段是关键路径**（失败立即 `aborted=True`），后续阶段（语义/聚类/向量）可降级跳过：

| 阶段 | 作用 | 失败行为 |
|------|------|---------|
| Stage 1 Parse | tree-sitter 扫描解析（`parsing/parser/`：Python+Java+doc）+ 结构关系提取 | 关键 |
| Stage 2 Structural Write | 结构实体/关系 MERGE 写入 Neo4j | 关键（`RuntimeError`） |
| Stage 2.5 Doc-Code Link | 文档→代码 `DESCRIBES` 关联 | 关键 |
| Stage 3 Semantic | LLM 语义提取（`parsing/extractor/semantic.py`）+ 概念对齐（`pipeline/aligner.py` 四步：精确→别名→向量→图结构）→ 写 ConceptEntity | 可降级 |
| Stage 4 Clustering | 模块聚类（`pipeline/module_clustering.py`）→ ModuleEntity | 可降级 |
| Stage 5 Vector Index | 实体向量写入 ChromaDB（`store/chroma_store.py`，Ollama embedding） | 可降级 |

增量更新走 `pipeline/incremental_updater.py`（基于 `pipeline/change_detector.py` git diff + `pipeline/impact_propagator.py` 双向 BFS 影响传播）。Butler 引擎（`butler/`：EventBus + Handler + GitWatcher）把上述能力包装成事件驱动的常驻服务。

## 测试结构

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yql210/ontology-driven-agent](https://github.com/yql210/ontology-driven-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
