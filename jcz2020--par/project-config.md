---
trigger: always_on
description: [English](https://github.com/jcz2020/par/blob/main/sdk/agent.md) · **简体中文**
---

# Agent API 参考
[English](https://github.com/jcz2020/par/blob/main/sdk/agent.md) · **简体中文**

本文档描述 P-A-R SDK 的 Agent 配置、运行时管理和工具注册 API。

## 运行时配置

### runtime_config

运行时通过 `Par.Runtime.create` 创建，需要以下配置：

```ocaml
type runtime_config = {
  persistence : [ `Sqlite of string ];
  event_bus : event_bus_config;
  default_quota : resource_quota;
  shutdown : shutdown_config;
  llm_providers : (string * llm_provider_config) list;
  eval_limits : eval_limits;
  parallel_tool_execution : bool;
  bash_confirm : bash_confirm_config;
  event_retention_seconds : float;
}
```

`Par.Runtime` 提供以下默认配置值，可以直接使用：

```ocaml
Runtime.default_event_bus_config   (* buffer_capacity=10000, DLQ 开启 *)
Runtime.default_quota             (* max_concurrent_tasks=10 *)
Runtime.default_shutdown_config   (* drain_timeout=30s *)
Runtime.default_bash_confirm      (* Always 策略 *)
```

### 创建运行时

```ocaml
val Runtime.create :
  ?persistence:persistence_service ->
  ?event_bus:Types.event_bus_service ->
  ?llm:llm_service ->
  ?embeddings:embedding_service ->
  ?memory:memory_service ->
  ?vector_store_backend:Types.vector_store_backend ->
  ?bash_policy:(module Bash_policy.POLICY) ->
  ?workspace:Workspace.workspace ->
  ?mcp_servers:Mcp_types.server_config list ->
  ?mcp_process_mgr:_ Eio.Process.mgr ->
  ?mcp_net:_ Eio.Net.t ->
  ?mcp_clock:_ Eio.Time.clock ->
  ?mcp_startup_policy:Mcp_types.startup_policy ->
  ?net:_ Eio.Net.t ->
  config:runtime_config ->
  Eio.Switch.t ->
  (runtime, error_category) result
```

所有可选参数默认为 `None`。关键可选参数：

| 参数 | 说明 |
|------|------|
| `?persistence` | 持久化后端（如 `Sqlite` 或 `Noop`）。 |
| `?event_bus` | 自定义事件总线配置。 |
| `?llm` | 主 LLM 服务 provider。 |
| `?embeddings` | 嵌入服务，用于 RAG 管道。见 [RAG API](rag.md)。 |
| `?memory` | 内存服务，用于跨会话 agent 记忆（FTS5）。见 [Memory API](memory.md)。 |
| `?vector_store_backend` | 向量存储后端，用于 RAG 相似度搜索。见 [RAG API](rag.md)。 |
| `?bash_policy` | Bash 信任边界策略模块。默认：`Always`（允许所有）。 |
| `?workspace` | 文件系统沙箱的 Workspace。默认为 CWD。 |
| `?mcp_servers` | 创建时启动的 MCP 服务器配置。 |
| `?mcp_process_mgr` | MCP stdio 服务器的 Eio 进程管理器。 |
| `?mcp_net` | MCP HTTP/SSE 服务器的 Eio 网络能力。 |
| `?mcp_clock` | MCP 启动超时的 Eio 时钟。 |
| `?mcp_startup_policy` | MCP 服务器启动策略（阻塞 vs 延迟）。 |
| `?net` | Runtime 全局出站 I/O 的 Eio 网络能力（与 `?mcp_net` 分开，后者仅作用于 MCP 服务器）。 |

完整示例：

```ocaml
open Par

let config = {
  persistence = `Sqlite "par.db";
  event_bus = Runtime.default_event_bus_config;
  default_quota = Runtime.default_quota;
  shutdown = Runtime.default_shutdown_config;
  llm_providers = [];
  eval_limits = { max_depth = 10; max_node_visits = 1000 };
  parallel_tool_execution = true;
  bash_confirm = Runtime.default_bash_confirm;
  event_retention_seconds = 604800.0;
}

let () = Eio_main.run (fun _env ->
  Eio.Switch.run (fun switch ->
    match Runtime.create ~config switch with
    | Error _ -> Printf.eprintf "Runtime creation failed\n"
    | Ok rt ->
      (* ... 使用运行时 ... *)
      let exit_code = Runtime.close rt in
      exit exit_code
  )
)
```

## Agent 配置

### agent_config

```ocaml
type agent_config = {
  id : string;                            (* Agent 唯一标识 *)
  system_prompt : system_prompt;          (* 类型化记录：{ sp_raw : string; sp_zone : zone_tag } *)
  system_prompt_template : system_prompt_template option;  (* 可选模板化提示词（带变量）*)
  model : model_config;                   (* LLM 模型配置 *)
  tools : tool_descriptor list;           (* 可用工具列表 *)
  max_iterations : int;                   (* ReAct 循环最大迭代次数 *)
  middleware : middleware_hook list;       (* 中间件管道 *)
  retry_policy : retry_policy option;     (* 可选重试策略 *)
  context_strategy : context_strategy option;  (* 上下文窗口管理策略 *)
  resource_quota : resource_quota option;  (* 可选资源配额覆盖 *)
  max_execution_time : float option;      (* 可选最大执行时间（秒）*)
  early_stopping_method : early_stopping_method;  (* 达到迭代上限时：Force 或 Generate *)
  on_max_tokens : on_max_tokens_behavior option;  (* None=Auto（默认），或显式 Retry/Continue/Return_partial *)
  max_continuation_chunks : int option;           (* None=Auto（默认），或显式上限 *)
  tool_timeout : float option;            (* 可选单次工具调用超时（秒）*)
  context_compression_threshold : float option;   (* v0.6.3+：按比例自动压缩。None=手动模式，Some 0.8=默认 *)
  compression_cooldown_messages : int option;     (* v0.6.3+：两次自动压缩间最小迭代数。Some 6=默认 *)
  context_window_override : int option;           (* v0.6.3+：覆盖 context window 大小；None=用 provider capability 或静态表 *)
  cache_strategy : cache_strategy;        (* 提示词缓存策略：No_caching | With_cache_of of cache_ttl *)
  approval_handler : approval_context Approval.approval_handler option;
                                          (* v0.8.0+：可选 HITL 审批 handler。None=用 runtime 默认 handler。
                                             设为 Some _ 时，此 agent 的 ReAct 循环遇到 Approval_required
                                             工具结果会挂起，并通过 Runtime.resume_approval 恢复。 *)
}
```

### 自动上下文压缩（v0.6.3+）

当 `context_compression_threshold` 设置时（默认 `Some 0.8`），引擎在每次 LLM 调用前检查
`估算 tokens / context window` 比例。如果超过阈值且冷却已过，应用配置的 `context_strategy`
（或 `context_strategy = None` 时默认用 `Summarize`）。

Context window 大小通过三层 resolver 解析：
1. `context_window_override`（用户 supplied，优先级最高）
2. `llm_service.context_window_fn`（provider capability 函数）
3. 静态查表（`default_context_window`）：gpt-4o 系列=128K，claude-4 系列=200K，gpt-3.5-turbo=16385，未知=8000（保守默认）

两个可观测事件：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcz2020/par](https://github.com/jcz2020/par) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
