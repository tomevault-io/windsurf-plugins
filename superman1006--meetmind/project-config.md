---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

MeetMind 是一个多 Agent RAG 协作系统的 demo（**TypeScript 项目**，monorepo：`apps/runtime` 后端 + `apps/desktop` 前端）：5 个角色 Agent（架构师、后端、前端、测试、产品经理）围绕一个项目需求展开讨论。架构师作为入口和终止者，其他角色在 LangGraph 条件边的驱动下自由路由。每个 Agent 拥有独立的 PostgreSQL 表 + 私有 RAG 工具（**PostgreSQL 混合检索：pgvector 向量 kNN + pg_trgm 关键字召回**，再过 **本地 cross-encoder rerank**）。存储已从 Elasticsearch 迁移到 PostgreSQL（pgvector）。

LLM 通过 OpenAI 兼容协议调用（默认配置小米 MiMo），所以 `new ChatOpenAI({...})` 里有一个 `modelKwargs: { thinking: { type: "disabled" } }`（给后端关掉 thinking 模式）不能改。

## 常用命令

包管理用 **pnpm**，不要用 npm / yarn。运行用 **tsx** 直接跑 `.ts`，开发期不需要预编译。

```bash
# 第一次（或换机）启动:
docker compose up -d                   # 起本地单节点 PostgreSQL+pgvector (pgvector/pgvector:pg17, 宿主机端口 5433)
pnpm install                           # 安装依赖（无 torch；主要是 onnxruntime + langchain + pg，首次较慢）
pnpm dev                               # tsx src/index.ts 启动 CLI
pnpm start                             # 同上

# 生产 / 编译:
pnpm build                             # tsc 编译到 dist/
pnpm start:prod                        # node dist/index.js
pnpm typecheck                         # tsc --noEmit，只做类型检查

# 重置某个 agent 的 PostgreSQL 表并重灌（开发时常用）
pnpm exec tsx -e "import('./src/database/ingestion/initializer.ts').then(m => m.resetAgentDb('backend'))"

# 完全抹库后重灌（删 docker volume，下次启动重新建表 + 灌种子）
docker compose down -v && docker compose up -d
```

环境变量在 `.env`（参考 `.env.example`）：LLM 三件套 `API_KEY` / `BASE_URL` / `MODEL_NAME` 是调用 LLM 的必需项；`PG_URL` 默认 `postgresql://meetmind:meetmind@localhost:5433/meetmind`（宿主机 5432 常被本地原生 postgres 占用，docker-compose 把容器映射到 **5433** 避让）。**rerank 已改为本地 cross-encoder，不再需要 `COHERE_API_KEY`。** 启动时唯一的硬退出是 `pingDb()` 失败（`process.exit(1)`）；LLM 的 key/baseUrl 缺失不会在启动时被拦截，会在第一次调用模型时抛错（zod 对这些字段给空字符串默认值，不报错）。

## 高层架构

### 调用链一句话

```
src/index.ts (load dotenv) → bootstrap() → buildGraph() → startServer(3002)   ← 默认入口（HTTP/SSE 服务）
                                              ↓
   POST /api → handleRpc → chat.send → runExecution(graph, …) → graph.stream(["custom","values"], thread_id)
                                              ↓
   START → rewrite_node → intent_node → route_node ──┬─► assistant_node → END      （右侧回答助手）
                                                     └─► architect_node → … 团队   （左侧 5 角色协作）
                                              ↓
              createNode(agent) 闭包 → agent.invoke()（Phase1 工具循环 + Phase2 结构化收尾）
                                              ↓
                                    routeToWhichAgent(state) → 下一个 _node 或 END
```

> `apps/runtime/src/cli/main.ts:main()` 仍是保留的旧交互式 CLI（非流式、无服务，`pnpm dev:cli`）；默认入口已是 HTTP 服务。预处理流水线（rewrite → intent → route）每轮入口跑一次、不计入 `iteration`，详见下文。

### 三层职责分离

1. **`src/agents/`** — 角色定义。子类只重写 `get systemPrompt()`，公共能力都在 `BaseAgent` 里。`BaseAgent` **在构造阶段**就 `bindTools(allTools)`（`allTools` 由 `src/tools/toolRegister.ts` 在模块加载时用 `ToolRegister` 类逐个 `register()` 拼好并导出，base.ts 直接 import 这个数组引用；MCP 工具由 bootstrap 阶段 `initMcpTools()` 异步 push 进同一数组）存进 `this._modelWithTools`，不在每轮 invoke 里反复 bindTools。`BaseAgent.invoke()` 是核心，**分两阶段**：(0) 用 `cleanBadChars` 清掉孤立 surrogate 码点，并把会话主人的个人记忆经 `memorySection` 拼到 systemPrompt 最前；(1) **Phase 1 工具循环**（抽到 `agents/toolLoop.ts` 的 `runToolLoop`，base 与 assistant 共用）—— 复用 `this._modelWithTools` 让 LLM 自主调工具，最多 `_MAX_TOOL_ITERATIONS` 轮，按 tool_call 名字在 `allTools` 里找工具执行（通过 `config.configurable.agentName` 传自己名字 + `expansionTerms` 传检索扩展词）；工具带 `metadata.risk`，`> low` 的执行前先发 `tool_approval_request` 等用户审批（HITL）；模型幻觉的未知工具跳过、只回占位 ToolMessage；(2) **Phase 2 结构化收尾** —— `withStructuredOutput(ModelOutputSchema)` 强制 LLM 输出 `{content, next_agent, done}` 三字段，`_buildAgentResponse` 把 `done` 字符串 coerce 成 bool、校验 `next_agent` 合法性。右侧 `AssistantAgent.answer()`（`agents/assistant.ts`）复用同一工具循环，但 Phase 2 改成纯自然语言流式收尾（不结构化、不路由），答完即 END。

2. **`src/graph/`** — LangGraph 编排。`AgentStateAnnotation`（`Annotation.Root`）是共享状态（`messages` 用 `concat` reducer 追加、其他字段用 `(_e,u)=>u` 覆盖）。图入口先过 **预处理流水线** `graph/preprocess/`：`rewrite_node`（改写独立句 + 检索扩展词）→ `intent_node`（本地 NLI 意图识别 + 规则短路）→ `route_node`（按 top-1/top-2 **间距** `INTENT_ROUTE_MARGIN` 分流），再由 `routeAfterPreprocess` 条件边按 `state.route` 走 `assistant_node`（右侧回答助手单节点，答完即 END）或 `architect_node`（左侧团队）。左侧团队内 `routeToWhichAgent` 是 5 个角色节点共用的条件边，按 `iteration >= maxIterations` → `state.done` → `isAgentName(state.next_agent)` → `architect_node` 兜底的顺序决定下一节点。图 `compile({ checkpointer: PostgresSaver })`，被打断/崩溃的一轮留 checkpoint，可经 `chat.getResumable` / `chat.resume` / `chat.discardResumable` 续跑或放弃。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superman1006/MeetMind](https://github.com/superman1006/MeetMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
