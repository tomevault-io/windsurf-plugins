---
trigger: always_on
description: - 这个仓库的目标是尽可能准确地复现上游 Codex（`https://github.com/openai/codex`），不是发明一个“类似 Codex”的简化替代品；凡是行为差异都应优先朝原版 Codex 收敛。
---

# pycodex AGENTS

## Repo Priors

- 这个仓库的目标是尽可能准确地复现上游 Codex（`https://github.com/openai/codex`），不是发明一个“类似 Codex”的简化替代品；凡是行为差异都应优先朝原版 Codex 收敛。
- 发布到 PyPI 时，distribution name 用 `python-codex`；代码包目录、`import` 路径和 CLI 命令仍保持 `pycodex`。
- 包结构直接放在 repo 根目录下的 `pycodex/`，不使用 `src/` layout。
- Rust 到 Python 的主映射：`submission_loop` -> `pycodex/runtime.py`，`run_turn` / `run_sampling_request` -> `pycodex/agent.py`，`ToolRouter` -> `pycodex/tools/base_tool.py` + `pycodex/tools/`.
- 会话协议先收敛到 4 类 item：`UserMessage`、`AssistantMessage`、`ToolCall`、`ToolResult`；只有这层稳定后再扩 richer event model。
- 优先保持主循环可测试、可替换：模型侧通过 `ModelClient` 协议接入；测试专用的 `ScriptedModelClient` 放在 `tests/fakes.py`，不要放进运行时包。
- `ResponsesModelClient` 直接复用 `~/.codex/config.toml` 的 provider 配置；当前已验证这里的 responses provider 需要 `stream = true`，否则会返回 `400` 和 `Stream must be set to true`。
- 现在 `ResponsesModelClient` 默认会对流式断连做 provider 级自动重试（`stream_max_retries` 默认 5）；写 CLI/REPL 测试时如果断言“先向用户报错，再靠下一句 `go on` 继续”，必须在测试 provider 配置里显式设 `stream_max_retries = 0`，否则测试可能一直等不到预期错误而卡住。
- `responses_server` compat 层应透传请求里的 `model`；不要再做 “取 downstream /models 第一个 id 并强制覆盖请求模型” 这种兜底兼容。
- 对 `model_provider = "vllm"`，`responses_server` 仍然走 `/v1/chat/completions` compat 路径，但要保留 reasoning：把 chat chunk 里的 `reasoning` / `reasoning_content` 翻回 Responses `reasoning` item，并把历史里的 Responses `reasoning` item 回放成下游 assistant message 的 `reasoning` 字段。
- `responses_server` 的 provider-specific chat payload 定制统一放在 `responses_server/payload_processors.py`：使用 `CompatServerConfig.model_provider` 选择 `provider_name -> proc_fn(outcomming_request)` 映射，并且只在真正发出 downstream `/v1/chat/completions` 前 post-process；`StreamRouter` 内部继续保留 canonical payload，避免 tool hydration / mock web_search follow-up 被 provider 改写污染。
- `responses_server` 如果要兼容下游 `/v1/messages`，也优先保持这条边界：内部继续用 canonical chat request / chat-like chunk 流，只有真正发请求和读取 SSE 时才做 messages 适配，这样 tool hydration、mock `web_search` follow-up、provider payload post-process 都能复用。
- 真实 vLLM `0.19.0` 的 `/v1/messages` 会对缺失 `max_tokens` 直接返回 `400`；messages 适配层必须总是补这个字段。当前约定是优先透传请求里的 `max_output_tokens`/`max_tokens`，否则回退到默认 `32000`。
- 对 vLLM chat-completions 打开 `return_token_ids=true` 时，streaming `prompt_token_ids` 只出现在首个 chunk，后续每个 chunk 的 `choices[*].token_ids` 都是 decode delta；要在 `responses_server` 侧导出 trajectory 时，按“首个 `prompt_token_ids` + 按序拼接所有 chunk 的 `token_ids`”重建即可。
- `pycodex` 默认是最小交互 CLI；无 prompt 时进入 REPL，并通过 `AgentRuntime` 跑外层提交循环。当前会显示最小事件流、assistant 流式输出、简单 title/history（`/title`, `/history`），并默认注册一组与原版一一对应的本地工具子集。
- 交互 CLI 的事件流展示优先表达用户可感知的阶段（例如工具开始/完成、模型回看工具结果），不要直接把内部 `iteration` 计数暴露成主要状态文案；`iterations` 应继续保留在 `TurnResult` 等程序化结果里。
- prompt/context 相关逻辑统一放在 `pycodex/context.py`：`AgentLoop` 只维护真实会话历史；每轮请求前由 `ContextManager` 注入 base instructions、developer message、`AGENTS.md` 指令和 `<environment_context>`，且这些注入项不写回 history。
- 对需要 model-specific prompt 的本地 model slug，直接在 vendored `pycodex/prompts/models.json` 补条目；当前 `step-3.5-flash` / `step-3.5-flash-2603` 已按这个方式接入。
- 交互 REPL 的 context 用量提示也应尽量贴近上游语义：展示“剩余 context 百分比”而不是原始 token 数；计算时按上游同款 `BASELINE_TOKENS=12000` 做归一化，并在模型元数据只有 `context_window` 时默认按 `95%` effective window 处理。只要当前模型能解析出 context window，初始 prompt 就先显示 `100%`，等首个 usage 回来后再刷新成真实值。
- 对交互 REPL 的 context 指示器，`model_context_window` 的取值优先级也要贴近上游：先吃 `config.toml` / profile 里的 `model_context_window` override，再回退到 vendored `models.json` 的 `context_window`；effective percent 继续沿用模型元数据，没有时默认 `95%`。
- `AgentLoop` 的 turn-loop 语义要跟上游 `codex-rs/core/src/codex.rs` 一致：按 follow-up / tool handoff 自然收敛，不要加固定 12 轮之类的 hard cap，也不要保留本地专用的 iteration-limit 参数。
- `README.md` 和 `docs/` 属于对齐工作的一部分：只要实现状态、对齐结论或使用方式发生实质变化，就应及时更新，不要让文档滞后于当前代码。
- 新工具必须继承 `BaseTool`，然后通过 `ToolRegistry.register(tool_instance)` 接入；不要再给 registry 传散装 name/description/handler 参数。
- request/tool schema 对齐应落在实际建模层（`ToolSpec` / `BaseTool` / request builder）本身，不要再引入 prompt 级别的 `serialized_tools` 旁路覆盖。
- 当前已接入的内置工具子集：`shell`、`shell_command`、`exec_command`、`write_stdin`、`exec`、`wait`、`web_search`、`update_plan`、`request_user_input`、`request_permissions`、`spawn_agent`、`send_input`、`resume_agent`、`wait_agent`、`close_agent`、`apply_patch`、`grep_files`、`read_file`、`list_dir`、`view_image`。后续继续新增时，仍然按原版 Codex 内置工具逐个补齐。
- `exec_command` / `write_stdin` 的 unified-exec 对齐要注意两层默认截断语义：省略 `max_output_tokens` 时也要按 upstream 默认 `10_000` token 预算裁剪 tool response；同时长时间未轮询的未读输出缓冲不能无限累积，需保留 upstream 同款 `1 MiB` head/tail。
- 当前协议层已经支持两类原版工具载荷：普通 function tools，以及 `apply_patch` 这类 freeform/custom tools；工具结果也支持结构化 `input_image` content items，用于 `view_image` 这类会把图片喂回模型的工具。
- 当前本地 runtime 已支持一个最小的 in-process sub-agent 管理层：`spawn_agent` / `send_input` / `resume_agent` / `wait_agent` / `close_agent` 通过共享的 `SubAgentManager` 驱动新的 `AgentRuntime` 实例，不依赖 CLI harness 自带的多 agent 基础设施。
- 当前交互工具 `request_user_input` / `request_permissions` 已接到 `pycodex` 交互 CLI：它们会在一次 turn 内直接复用同一个 stdin/input_fn 向用户提问，因此只有交互模式下有完整体验；非交互调用默认会返回取消/不可用类错误。
- `request_user_input` 对齐上游时要注意两层：Default mode 默认返回固定错误 `request_user_input is unavailable in Default mode`；Plan mode happy path 则要求每个问题都有非空 `options`、handler 会补 `isOther=true`，并把结构化答案序列化成 JSON 字符串加 `success=true` 塞回 `function_call_output`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Randomizez/pycodex](https://github.com/Randomizez/pycodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
