---
trigger: always_on
description: > Agiwo 开发指南。供 AI 编码助手和开发者快速理解当前仓库结构、边界和约定。
---

# AGENTS.md

> Agiwo 开发指南。供 AI 编码助手和开发者快速理解当前仓库结构、边界和约定。
>
> 本文件只维护“目录级职责 + 稳定 API/边界”，不再逐个文件镜像源码；实现细节以源码为准。

## How To Use This File

- 如果文档与源码冲突，以源码为准，并顺手更新本文件。
- 优先用本文件定位“应该去哪个目录/模块看”，不要把它当成逐文件索引。
- 如果某条信息很难长期与源码同步，就应该提升抽象层级或直接删掉。

## Logging

统一日志格式：`logger.{level}("event_name", key=value, ...)`. 详见 [docs/logging-guidelines.md](docs/logging-guidelines.md).

## Config Hot Reload

配置热更仅在 agent 为 IDLE/COMPLETED/FAILED 状态时生效，运行中 agent 配置延迟生效。详见 [docs/config-hot-reload.md](docs/config-hot-reload.md).

## Repository Layout

### SDK (`agiwo/`)

| Path | Responsibility |
| --- | --- |
| `agiwo/agent/` | Canonical agent runtime。public API 只从 `agiwo.agent` 暴露；顶层只保留稳定入口与核心 orchestrator（如 `agent.py`、`definition.py`、`run_loop.py`、`llm_caller.py`、`tool_executor.py`、`prompt.py`、`trace_writer.py`）。纯数据模型收口在 `models/`，hook contract 收口在 `agiwo.agent.hooks`，nested-agent adapter 收口在 `nested/`，run/session runtime context、state helper 与 `RunStateWriter` 严格写路径收口在 `agiwo.agent.runtime`，termination logic 收口在 `termination/`，目标导向 review / step-back 优化收口在 `review/`，`storage/` 负责持久化。`run_loop.py` 使用 `RunLoopOrchestrator` 类封装运行循环逻辑，消除多层嵌套；`run_loop.py` 是唯一的单次 run execution owner，而 `agiwo.agent.runtime` 只承载 `RunContext` / `RunRuntime`、`SessionRuntime` 与 `RunStateWriter`，不得再暴露 execution-owner alias、commit-pipeline facade 或兼容壳。 |
| `agiwo/llm/` | Model 抽象、Provider 适配器、配置策略、消息/事件归一化，以及统一的 model factory。 |
| `agiwo/tool/` | Tool 抽象、最小执行上下文、builtin tools、后台进程 registry（`process/`），以及工具侧存储（如 citation）。 |
| `agiwo/scheduler/` | Agent 之上的编排层。`scheduler.py` 是 facade 与 loop lifecycle，`engine.py` 是唯一编排 owner，`runner.py` 负责单次 dispatch action 执行，`commands.py` 承载调度动作与 tool DTO，`runtime_state.py` 承载进程内 live state 与 tick helpers，`tool_control.py` 收口 child/sleep/cancel 的 tool-facing control，`runtime_tools.py` 是注入给 agent 的 scheduler runtime tools，`store/` 只负责持久化。`runner.py` 使用策略表驱动 output-handling 链，消除 chained responsibility。 |
| `agiwo/observability/` | Trace/Span 模型、查询接口与 trace storage 实现；agent runtime 的 Trace 投影层收口在 `agiwo/agent/trace_writer.py`，并以 committed `RunLog` facts 为输入构建 Trace view。 |
| `agiwo/embedding/` | Embedding 抽象与 factory，包含本地/OpenAI 风格实现。 |
| `agiwo/skill/` | Skill 的发现、路径规则（`config.py`）、加载、注册、异常定义，以及 `SkillTool` 桥接。 |
| `agiwo/workspace/` | Agent workspace 路径语义、模板/bootstrap、工作区文档读取与变更 token。 |
| `agiwo/memory/` | 共享 MEMORY 索引/切块/搜索能力，以及 `WorkspaceMemoryService`。 |
| `agiwo/config/` | SDK 全局配置入口、Provider 枚举与共享设置。 |
| `agiwo/utils/` | 跨模块运行时工具。`storage_support/` 负责共享 SQLite/Mongo runtime、schema/index 初始化等基础设施。 |

### Console (`console/`)

| Path | Responsibility |
| --- | --- |
| `console/server/` | FastAPI 控制面与 runtime 集成。 |
| `console/server/routers/` | API/SSE 边界，只做 HTTP 路由与请求/响应装配。 |
| `console/server/services/` | 应用服务层。`runtime/`（agent factory、runtime cache、session runtime / session service、scheduler tree view）、`tool_catalog/`（tool reference / catalog / runtime builder）、`agent_registry/`（配置 CRUD + store 子包）、`session_store/`（Console 会话存储工厂与实现）、`runtime_config.py`（运行时全局配置查看/覆盖）、`storage_wiring.py`（存储 config builders）、`metrics.py`。 |
| `console/server/models/` | Console 数据模型目录。`view.py` 只放 API/SSE 视图模型；`session.py`、`agent_config.py`、`runtime_config.py`、`metrics.py` 放共享运行时/配置/聚合模型。不要再新增 `schemas.py` 或平级 `domain/`。 |
| `console/server/channels/` | 渠道适配层，负责批处理、消息解析、delivery，以及 Feishu 等渠道集成。 |
| `console/web/` | Console 前端。 |
| `console/tests/` | Console 后端测试。 |

### Supporting Directories

| Path | Responsibility |
| --- | --- |
| `tests/` | SDK 测试，按子系统分目录。 |
| `scripts/` | 低噪音 lint 入口与 repo guard。 |
| `lint/` | import-linter contract 等机器护栏配置。 |
| `docs/` | 设计文档与渠道说明，不是运行时源码真相。 |
| `templates/` | 运行时会消费的模板内容。 |
| `trash/` | 删除文件的落点；优先 `mv` 到这里，不要直接 `rm`。 |

## Core Components

### Agent

- `Agent` 是具体类，不是 ABC。
- 公开构造入口是 `Agent(config: AgentConfig, *, model: Model, tools: list[BaseTool] | None = None, hooks: HookRegistry | list[HookRegistration] | None = None, id: str | None = None)`；`hooks` 统一走 `agiwo.agent.hooks` 中的 phase-based registry；`tools` 是功能/用户自定义工具（受 `allowed_tools` 过滤），`system_tools` 是系统工具（不受 `allowed_tools` 过滤）；`AgentConfig` 只承载纯配置，不放 live object。
- **`id` 必须稳定**：在 Console 这类跨请求复用会话的场景里，每次构造 `Agent` 都必须传稳定 `id`（如 registry `config.id`），否则历史 steps 会丢。不传 id 时自动生成 `{name}-{hex[:6]}` 格式。
- `AgentConfig.allowed_skills` 进入 SDK runtime 前必须已经展开成“显式 skill 名列表”或 `None`；不再接受 wildcard/pattern。
- 对外执行原语是 `start(...)` 返回 live execution handle；`run(...)` / `run_stream(...)` 只是便利封装。
- 嵌套 agent 执行是内部协议，由 `nested/agent_tool.py` 通过 `Agent.run_child(...)` 进入；不要再暴露公开 `context` 参数。
- `SessionRuntime` 是 session 级 owner；`RunContext` 组合 immutable identity 与 mutable ledger。运行时事实写入统一经由 `RunStateWriter` 收口，`SessionRuntime` 只负责 sequence、append run-log、以及基于已提交 `RunLog` facts 投影 replayable `stream`/`trace` view。
- 单次 run 的主线只在 `agiwo.agent.run_loop.RunLoopOrchestrator`；runtime 包只承载 context、session 与 committed-state writer，不保留 `RunEngine` 这类别名入口，也不保留 runtime-level `commit_step` 包装层。
- `StepView` 是唯一的已提交 step 视图与运行时 step 数据模型；统一通过 `StepView.user()/assistant()/tool()` 创建，不要再引入第二套 step 记录模型。
- `UserMessage` 是 `UserInput` 的 canonical structured owner；input normalization 和 storage encoding/decoding 统一收口在它本身。
- public hook contract 统一由 `HookPhase` 驱动；`before_tool_call` / `after_tool_call` 是按单次 tool call 触发的公开 phase，hook 执行顺序固定为 `group -> order -> registration order`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xhwSkhizein/agiwo](https://github.com/xhwSkhizein/agiwo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
