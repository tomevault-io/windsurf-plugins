---
trigger: always_on
description: 本文件是 SparkArc 项目的强约束指南。
---

# SparkArc AGENTS 指南（给 AI 助手与贡献者）

## 1. 文档目标

本文件是 SparkArc 项目的强约束指南。

首先是几条铁律：

- 这是一个庞大的项目，在任何更改前，你必须确保拿到足够的信息，对该链路足够了解，防止堆屎山。
- 任何改动都要优先接入现有统一管线，避免同一能力在多处重复实现。
- 任何新增能力都要做到“改一处，全链路受益”。
- 任何短平快修补都不能以破坏长期可维护性为代价。

### 1.1 Python 环境边界

`server/.runtime/python/` 是 Windows 一键启动脚本生成的便携运行时环境，仅服务于 `start.bat` 这类免配置启动链路。它不是开发者默认 Python 环境，也不是 AI 运行开发测试时的首选解释器。

开发 / 测试应优先使用 VSCode 当前选中的解释器、用户显式指定的 conda / venv / uv 环境（根目录找不到环境就考虑诸如conda env list寻找，需要灵活判断用户当前使用的是哪种包管理器）。

## 2. 统一收口，不复制实现

SparkArc 现有架构已经有清晰收口层。新增功能必须先判断是否能接入现有收口点，而不是新开平行管线。

 后端收口重点：

 - 通讯层底座：server/agents/communication.py
 - 执行协议层：server/agents/agent_utils.py
 - 工具门面层：server/agents/agent_tools.py（统一门面） + server/agents/tools/*（内部实现）
 - 公共工厂 / 服务层：server/agents/agent_factory.py + server/agents/project_content.py + server/agents/auto_write_service.py
 - 多 Agent 调度层：server/agents/director_graph.py
 - 流式桥接层：server/agents/routes/streaming_utils.py
 - 业务语义层：server/agents/routes/stream_semantics.py + server/agents/routes/execution_core.py
 - 路由聚合层：server/agents/routes/__init__.py
 - 上下文布局层：server/agents/prompt_layout.py + server/agents/context_budget.py
 - **大统一工具性底层（大统一基建）**：
   - **局部替换与增量修改（Patch）**：统一收口在 `server/agents/tools/common.py` 的 `_apply_patch`。无论是剧本复写、大纲局部修改还是设定更新，凡是涉及“在已有文本中定位并替换”的逻辑，必须复用此底层，严禁各 Agent 自行实现正则或字符串替换。
   - **智能文本切分（Token Chunking）**：统一收口在 `server/core/file_ingest/chunking.py` 的 `TokenTextSplitter`（或通过 `server/agents/agent_style/text_splitter.py` 兼容重导出）。无论是上传附件、评审专家审稿、还是文风克隆分析，凡是涉及按 Token 数量切分文本的逻辑，必须复用此底层，避免 3 次以上重复实现。
   - **语义分块器（Semantic Chunker）**：统一收口在 `server/story/semantic_chunker/` 的 `SemanticChunker`。凡是涉及项目文件、知识图谱、向量索引的语义分块，必须复用此底层。
   - **基建扩展原则**：上述三项仅为当前最典型的工具性基建示例。**后续任何新增的、可能被多处复用的底层基础设施（如向量检索、缓存控制、文件解析等），必须遵循相似的“大统一”原则，先下沉至公共工具层或核心服务层，严禁在各业务线或 Agent 内部重复造轮子。**

前端收口重点：

- 流式任务入口：client/src/utils/streamingRuntime.ts（createStreamingTask）
- 全局遮罩统计：client/src/utils/loadingStats.ts
- 事件总线：client/src/eventBus.ts
- 全局加载 UI：client/src/components/share/GlobalLoading.vue
- 聊天流消费收口：client/src/components/stores/chatStore.ts

## 3. 两条主链路（必须分清）

### 3.1 聊天主链路（Chat NDJSON）

用途：自由对话、Director 调度、工具调用可视化。

标准链路：

1. 前端通过 chatStore/chatService 发起聊天流。
2. 后端路由在 server/agents/routes/chat.py。
3. Agent 侧通过 SparkBaseAgent.chat_stream 推送事件。
4. chat.py 为每个运行中任务创建 assistant 占位消息，并把事件写入 ChatTaskEntry 的 append-only event_log。
5. chat.py 输出 NDJSON 事件（task_snapshot、assistant_delta、reasoning_delta、tool_*、task_done 等）。
6. chatStore._consumeStream 统一消费并维护消息、segments、tool_traces。
7. chat.py 运行中持续 checkpoint 到同一条 assistant 消息，落盘 metadata.segments / metadata.tool_traces / stream_seq，保证刷新后时序可恢复。

关键事实：

- 聊天链路是 NDJSON，不是业务语义 onStart/onDelta 协议。
- 工具事件与正文可以交错出现，不能假设固定顺序。
- 前端刷新/重连恢复必须走 task_snapshot + afterSeq 游标回放；聊天链路不保留 progress_queue，禁止把 Queue 当 replay log 使用，也禁止用 get_nowait 这类破坏性读取作为恢复链路。
- 运行中的 assistant 消息必须复用同一条 DB 记录增量更新，完成后不可再 append 第二条助手消息。

### 3.2 业务任务主链路（SSE/语义流）

用途：长耗时业务任务，例如 production、style、auto_write、structure、lorebook、muse。

标准链路：

1. 前端创建 createStreamingTask(scope, target)。
2. 前端使用 consumeSSEReader/consumeTextReader/consumeNdjsonReader 消费流。
3. 后端路由通过 iterate_sync_iterable_in_thread 桥接同步生成器到异步响应。
4. 业务事件统一附加 onStart/onProgress/onDelta/onStats/onDone/onError/onCancelled。
5. 全局遮罩统一走 global-loading/cancel-loading 事件。

关键事实：

- 业务流由 streamingRuntime 统一托管，不要在页面里重复写一套“读取器 + 取消 + 统计”状态机。
- SSE 心跳、取消、统计逻辑已在主链路中沉淀，优先复用。

## 4. 后端扩展规则

### 4.1 新增 Agent：先复用双基座

新 Agent 默认应复用：

- SparkBaseAgent（通讯与聊天能力）
- SparkAgentExecutor（build_context -> execute -> write_result 执行协议）

参考文件：

- server/agents/setup_agents.py
- server/agents/communication.py
- server/agents/agent_utils.py

强约束：

- 不要把核心业务逻辑散落在路由函数里。
- 不要跳过 build_context 直接在多个入口拼 prompt。

### 4.2 新增 Agent 后必须同步注册

 后端必须更新：

 1. server/agents/registry.py（Agent 元数据）
 2. server/agents/routes/runtime.py（若涉及信标/号角/锁定策略）
 3. server/agents/agent_tools.py（统一门面导出）+ server/agents/tools/registry.py（工具分组 / 绑定真相源）
 4. server/agents/director_graph.py（若需要被 Director 委派）

### 4.3 工具扩展必须走工具门面

 新增工具必须统一经 server/agents/agent_tools.py 门面接入；具体 schema 与实现按域落在 server/agents/tools/*，统一在 server/agents/tools/registry.py 注册，再由 agent_tools.py 对外导出。

 禁止：

 - 在单个 Agent 内部私定义一套独立工具调用协议。
 - 在路由层直接执行“伪工具逻辑”绕过工具门面。
 - 在 `server/agents/tools/registry.py` 之外再造第二套工具注册表、Agent→工具映射或平行工具管线。
 - 工具层直接反向依赖 `server/agents/routes/*` 私有实现；若需要复用能力，应先下沉到 `agent_factory.py` / `project_content.py` / `auto_write_service.py` 这类公共层。

### 4.4 工具 UI 联动必须双端一致

工具事件中的 UI 提示由后端 communication.py 的 build_tool_stream_event 注入（ui_scope/ui_target/ui_refresh_events），前端 chatStore 读取。

### 4.4.1 AgentSkills 与 MCP 边界

- AgentSkills 是写作质量参考层，不是运行时插件执行层。导入逻辑在 `server/agents/skill_packs.py`，工具入口为 `search_skills` / `read_skill` / `read_skill_reference`，统一在 `server/agents/tools/registry.py` 作为共享 Skill 工具分配。
- Skill 读取视图必须保持 `quality_only`：只采纳写作质量、审美判断、检查清单和领域知识；不得采纳脚本、命令、工具调用、外部工作流、输出格式、字段结构或落盘规则。
- MCP 灵感服务通过 `server/mcp_server/spark_inspiration/server.py` 挂载到 `/api/mcp/`，对外提供 `capture_spark` / `list_sparks`；内部 `capture_inspiration` 属于 `MCP_ONLY_TOOLS`，禁止挂载到普通聊天 Agent。
- MCP 控制服务通过 `server/mcp_server/spark_control/server.py` 挂载到 `/api/mcp/control/`，9 个核心工具在该服务收口，12 个只读查询工具必须继续从 `MCP_EXPOSED_QUERY_TOOL_NAMES` 与 `TOOLS_BY_NAME` 真相源派生。写盘请求必须提交 Director 工单，不得直接暴露写盘工具。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letmeow/spark-arc-studio](https://github.com/letmeow/spark-arc-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
