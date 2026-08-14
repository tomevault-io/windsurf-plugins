---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目与当前主路径

OpenTrace 是一个以 Responses API 和可恢复 Agent Loop 为核心的企业提问系统，在线能力收敛为 RAG、企业大脑上下文和 DataAgent，并保留资料、数据库、记忆、任务、Skills、设置及管理员治理页面。后端使用 Python 3.11、FastAPI、PostgreSQL/pgvector、Redis；前端使用 React、TypeScript 和 Vite。默认部署方式是 Docker Compose。

当前在线对话主路径不是旧的 `CognitiveKernel → CognitiveSupervisor → RuntimeGateway`，而是：

```text
POST /api/v2/responses
  → API 校验租户、资源范围和幂等键
  → PostgreSQL Response / Item / Event / Outbox（同一事务提交）
  → Agent Worker 将 Outbox 投递到 Redis Streams，并以数据库租约领取 Response
  → AgentLoop：IntentPlan → ContextAssembler → Manager model/tool loop
  → typed tools / expert agents（写操作进入持久化审批暂停点）
  → PostgreSQL 持久化输出、事件与工具账本
  → SSE 从持久化事件投影，可按 sequence_number 断点续传
  → 会话摘要与记忆学习
```

`/api/v1/chat` 和 `/api/v1/tasks` 已退役并返回 `410 Gone`。`kernel/cognitive_supervisor/`、`kernel/runtime_gateway.py` 等旧认知运行时代码仍有架构合约覆盖，但不应被当作当前 Responses 请求入口；只有任务明确涉及该兼容子系统时才修改。

## 常用命令

### 安装与运行

```bash
# 后端开发环境
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -e ".[dev]"

# 首次准备配置
cp .env.example .env

# Docker 后端栈：API、Agent Worker、PostgreSQL、Redis
bash start.sh                    # 启动后会执行 alembic upgrade head
bash start.sh --build            # 强制执行缓存增量构建并启动
bash start.sh --rebuild          # 仅排查缓存问题时完全重建
bash start.sh --pull             # 主动更新基础或远程镜像
bash start.sh --verify           # 启动并做 Docker 快速验收
bash start.sh --with-observability
bash stop.sh
bash restart.sh

# 日志
bash scripts/docker_logs.sh api
bash scripts/docker_logs.sh agent-worker
```

Prometheus 和 Jaeger 仅在 `--with-observability` 下启动。Compose 不包含前端。

Docker 镜像通过 `COPY . .` 打包源码，并通过镜像 label 保存源码指纹。普通启动会在
指纹变化时自动增量构建；也可以显式强制构建：

```bash
bash restart.sh --build
```

日常 `start.sh` / `restart.sh` 会复用已有统一应用镜像，不再默认构建或拉取。
只有缓存异常时才使用 `bash restart.sh --rebuild`。

宿主机运行 API、Docker 只承载 PostgreSQL/Redis 时，要覆盖容器网络主机名：

```bash
DATABASE_URL=postgresql://postgres:<password>@127.0.0.1:5432/opentrace_v2 \
TOKEN_DB_URL=postgresql://postgres:<password>@127.0.0.1:5432/opentrace_v2 \
REDIS_URL=redis://127.0.0.1:6380/10 \
python -m uvicorn gateway.api_gateway.main:app --host 0.0.0.0 --port 14100 --reload
```

### 测试与静态检查

```bash
# 全部后端测试
python -m pytest -q

# 单个文件 / 单个测试
python -m pytest -q tests/test_responses_contract.py
python -m pytest -q tests/test_responses_contract.py::test_stream_disconnect_does_not_cancel_durable_response

# 格式、lint、类型检查
black --check .
black .
ruff check .
mypy .

# 架构边界与主要合约
bash scripts/check_import_boundaries.sh
bash scripts/run_vnext_final_tests.sh
bash scripts/run_enterprise_contract_tests.sh
bash scripts/check_gateway_silent_failures.sh
bash scripts/check_kernel_silent_failures.sh

# 运行中的服务栈验收
bash scripts/verify_all.sh
bash scripts/verify_all_docker.sh
bash scripts/verify_agent_bus_e2e.sh
bash scripts/verify_e2e.sh

# 发布前；默认 --full，也可用 --quick
bash scripts/preflight_release.sh --full
```

`verify_all.sh` 中包含 HTTP/Redis 等端到端检查，通常需要已启动的依赖；纯合约回归优先直接运行相关 pytest 或 `run_vnext_final_tests.sh`。

### 前端

```bash
cd frontend
npm ci
npm run dev                         # http://localhost:14108
npm run build                       # tsc -b && vite build
npm test                            # vitest run
npm test -- src/utils/__tests__/responsesStream.contract.test.ts
```

CI 使用 Node.js 20；不要只运行 Vite build 而跳过 TypeScript 构建。

### 数据库迁移

```bash
bash scripts/migrate.sh

docker compose exec -T api alembic history --verbose
docker compose exec -T api alembic revision --autogenerate -m "description"
bash scripts/verify_migration_idempotent.sh
```

开发环境允许 `ensure_runtime_schema()` 做兼容性 DDL；staging/production 只做 schema readiness 校验，正式结构变更必须通过 Alembic。

## 架构边界

### API 与持久化执行面

- `gateway/api_gateway/main.py` 创建 FastAPI 应用、注册中间件和路由；主对话入口是 `gateway/api_gateway/routers/responses.py`。
- Responses API 只验证命令并提交持久状态。模型和工具执行只能发生在 Worker，不能在 API 请求进程中通过 background task 偷跑。
- `ResponseRecord`、`ResponseItem`、`ResponseEvent` 和相关审批/工具账本是在线会话事实来源。旧 Message/TraceLog 只能作为迁移输入，不能重新引入在线读取。
- PostgreSQL 是事实来源，Redis Streams 只是投递与唤醒层。Redis 丢消息或不可用时，Worker 通过数据库 claim 恢复；流式客户端断开不会取消 Response。
- `infra/responses/repository.py` 管理事件、Outbox、租约和重试；`infra/responses/worker.py` 负责 Outbox 投递、Response 执行、恢复与最终持久化。`infra/response_worker.py` 只是兼容导入。

### Manager Agent Loop

- `kernel/agent_loop/runner.py` 是当前 Manager loop：先用严格工具调用生成 `IntentPlan`，再选择最小能力集合，调用统一 Model Gateway，并循环执行工具/专家 Agent。
- `kernel/agent_loop/context.py` 只沿当前 Response 父链组装上下文。优先级依次为平台/租户、工作区、Assistant Profile/用户指令、会话/回合指令、当前输入；记忆按 user/conversation scope 隔离。
- 只读工具可自动执行；write/destructive 工具必须写入 `ResponseApproval` 并返回 `requires_action`。副作用工具不自动重试，未知结果进入 reconciliation 状态；不要绕过持久化幂等账本。
- `model/model_gateway/gateway.py` 是模型调用统一入口，按 `LLMRole` 管理 adapter、fallback、重试、熔断和调用计量。不要在业务模块直接实例化 provider client。

### Capability、Agent 与后台 Worker

- `kernel/runtime/capability.py` 的 registry 负责在线专家 Agent 调度。内置 Agent 由 `agents/bootstrap.py` 按 `kernel/agent_runtime/agent_topology_manifest.yaml` 注册。
- Tier-1 在线能力只包括 `data` 和 `rag`；企业大脑由 `ContextAssembler` 注入上下文，不作为可调用 Agent。拓扑 manifest 是 bootstrap/worker/bus eligibility 的单一真相；变更时同步其版本和 Agent Runtime 合约测试。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SongJok/OpenTrace](https://github.com/SongJok/OpenTrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
