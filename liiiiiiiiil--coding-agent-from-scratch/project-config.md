---
trigger: always_on
description: `agent-from-scratch` 是逐步生长的 Python 编程 agent（包名 `mini_agent`）。核心运行时和关键执行流程仅使用标准库；外围用户体验能力可以通过可选依赖增强。
---

# AGENTS.md

## 项目定位

`agent-from-scratch` 是逐步生长的 Python 编程 agent（包名 `mini_agent`）。核心运行时和关键执行流程仅使用标准库；外围用户体验能力可以通过可选依赖增强。
本文件只保存会直接影响 agent 运行、授权和代码修改的硬约束；版本路线、教程规范和完整架构见 `docs/`。

## 执行规则

- **先调查再修改**：先阅读相关实现、测试和计划，确认现有行为与边界，再提出最小改动。
- **核心标准库优先**：LLM 调用、agent loop、工具执行、权限、状态、上下文和验证流程不得引入第三方依赖；外围体验能力可以使用可选第三方库，但必须有标准库回退，不得成为默认安装或核心模块的硬依赖。
- **HTTP 客户端约束**：LLM 调用必须使用 `http.client`，请求显式设置 `Accept-Encoding: identity`；不要改用 `requests` 或 `urllib`。
- **配置安全**：真实的 `BASE_URL`、`API_KEY`、`MODEL` 只放本地 `config_local.py`，不得提交到版本库。
- **异常边界**：工具层/执行器负责把 handler 异常转换为错误结果并回灌模型；核心 agent loop 不对 LLM 或 CLI 顶层异常做兜底。
- **协议完整**：工具调用必须为每个 call 回灌对应的 `role=tool` 结果；单轮工具结果全部回灌后再进入下一轮。
- **持久化工具边界**：开启 `/save` 后，schema 3/4 必须在 handler 前提交 `handler_admitted`，每个 call 的 State、对应 `role=tool` 结果和边界状态必须按模型顺序原子提交；整轮 committed 前不得再次请求 LLM。提交失败不得进入 handler、后续 call 或下一次 LLM 请求；崩溃恢复不得重放 pending call。
- **Plan Contract**：复杂任务由模型通过 `commit_plan` 提交完整不可变 revision，通过 `update_plan_progress` 追加独立步骤进度事件；简单任务继续 Direct Path。计划校验失败只回灌 `plan_rejected`，不得创建 `FailureEvent`、推进 generation 或产生验证证据；计划写入不替代实际执行和独立 verification。
- **只读规划与交接**：普通任务可经 `begin_plan` 进入只读调查；`--plan` 任务必须先调查，提交后等待用户批准当前 revision。`exploring` 的副作用、verification 和混合提交在整轮与执行器两层拒绝；批准计划不得绕过 PermissionGate。用户驳回或继续调查的反馈由 CLI 记录，不能由模型伪造。
- **Shell 副作用分类**：所有 `run_shell` 调用均按可能有副作用处理并在获准后预留 generation；`purpose=verification` 只指定验证证据用途，不把命令降为只读。
- **完成与上限**：无 `tool_calls` 才能结束；有 active Plan Contract 时所有活动步骤必须完成，并满足修改后的验证条件；无计划的 Direct Path 沿用原有完成条件。默认最多 50 轮，超限返回明确结果。
- **回放只读**：Trace & Replay 只能消费当前进程、当前任务的结构化 State 快照；不得调用 LLM、执行工具、经过权限授权、修改 history、状态、预算或 generation。当前 generation 的验证证据用于完成判定，append-only verification history 用于跨 generation 回放；断链和跨 generation 证据必须标记为不完整，不得推测补全。
- **教程读者优先**：撰写或修改 `docs/tutorials/` 时，默认读者具备基础 Python 和命令行能力，但刚接触 Agent，也不了解本项目内部架构。必须先讲问题和直观含义，再讲模块、字段、协议与实现；术语、缩写和项目内部概念首次出现时必须就近解释，不得用代码、符号或文件清单代替教学说明。具体要求见[教程作者规范](docs/governance/tutorial-authoring.md)。
- **主 README 编辑**：修改 `README.md` 的学习路径、阶段名或版本主题前，必须遵守[主 README 编写规范](docs/governance/readme-authoring.md)：主题默认使用通俗中文，只有协议字段、代码标识和公认技术名词可保留英文；修改后运行 `PYTHONPATH=src python scripts/check_readme.py`。
- **修改后验证**：文件修改完成后，至少运行与改动相关的测试；交付前运行下列完整验证命令（或说明无法运行的原因）。
- **破坏性操作**：未经用户明确授权，不执行删除、重置、覆盖大量文件或其他难以恢复的操作。
- **Tag 操作专属权限**：Git tag 的创建、移动、覆盖、删除和远程推送只能由用户本人手动完成。助手不得代为执行任何 tag 操作，即使用户在任务中要求打 tag；如任务涉及 tag，只能说明步骤或提供命令，等待用户手动完成。

## 当前状态

稳定基线为 `v0.16.1`（计划驱动执行的完成提醒进展感知补丁）；主线当前开发版本为 `v0.49`（可续接子会话）。新增功能意图记录在对应 `docs/plans/`，只有运行时硬约束变化才更新本文件。

v0.46 MCP 硬约束：`MCP_SERVERS` 中只有显式 `agent_enabled=True` 的配置项才进入父 Agent Runtime；默认 `False` 的 Server 仍只供独立 `python -m mini_agent.mcp` 命令使用。父侧 MCP Tool 通过 Tool Registry、ToolExecutor、PermissionGate 和 `AgentRuntime.run()` 运行，默认 `effect_class="possible"`、默认权限 `ask`，只有同一 Server 的精确 `readonly_tools` 才能降为 `none`，仍须授权且不自动成为 verification evidence。MCP、Resource 和 Prompt 不进入 Subagent。
配置导入不启动 Server，命令 argv 不经过 shell。Client 固定 MCP `2025-11-25`，必须按 `initialize → notifications/initialized → tools/list → tools/call` 运行，完整读取分页并冻结工具目录；独立 CLI 只有在每次请求前获得交互式明确确认后才发送 `tools/call`。父 Runtime 在新任务和恢复任务中从当前配置重新连接与发现目录，退出、`/new`、`/reset` 和恢复失败都必须有界关闭连接。

HTTP 传输只接受逐请求 `application/json` 响应，使用标准库 `http.client`；默认要求 HTTPS，明文 HTTP 只有显式允许的回环地址可用。客户端拒绝 SSE、重定向、OAuth、服务端主动请求和自动重试，初始化后的 session ID 只在内存中携带，并尽力有界发送会话 DELETE。`resources/list`、`resources/read`、`prompts/list` 和 `prompts/get` 完整处理分页并冻结目录；Resource 只允许有界 UTF-8 文本，Prompt 只允许带原始 `user`/`assistant` 标签的有界文本。四个 Resource/Prompt 命令只由父 CLI 显式触发，读取/获取按精确 `alias:uri` 或 `alias:name` 授权；Resource 进入带来源和低信任标记的普通 history，Prompt 必须完整预览并确认后作为用户侧输入，服务端内容不得进入受保护 system、State、Trace 摘要或 verification evidence。

冻结后的工具目录不得再次从 Server 刷新；普通通知只保留最近 64 条。CLI 确认前必须展示完整参数，无法完整展示时拒绝调用；关闭直接子进程未完成必须报告失败。JSON-RPC 错误码必须是整数，终端输出中的控制字符必须转义。

stdio MCP 的 stdout 只允许逐行 UTF-8 JSON-RPC，单条消息最多 1 MiB、等待队列最多 64 条；stderr 独立排空并只保留末尾 16 KiB。启动/握手、目录页、工具调用和关闭分别遵守 10 秒、10 秒、30 秒和 2 秒默认边界。坏编码、坏 JSON、协议失步、EOF、超时或超限后连接不得复用，也不得自动重试 `tools/call`；关闭必须先关 stdin，再有界终止并回收直接子进程。MCP 错误只保留有界、脱敏的类别、方法和服务端错误码；工具结果中的 `isError=true` 仍是有效 MCP result，不等同于 JSON-RPC error。

模型绑定硬约束：provider/profile 只能从本地配置解析；父 binding 和子 binding 必须在对应 Runtime 创建前冻结，运行中不得通过全局变量切换模型。`delegate_task` 只能请求获准的本地 `model_profile` 别名，未知或越权别名必须在 HTTP 请求前拒绝；不得自动 provider fallback。State、Context、session、Trace、工具结果和用户可见错误只能保留无凭据的 profile/provider/protocol/fingerprint 来源摘要，不得持久化真实 endpoint、model ID、API key 或认证头。摘要请求必须使用同一 binding 并计入其 usage；provider usage 缺失时保守估算并标记来源。

崩溃恢复硬约束：`active + schema 3/4 pending tool_boundary` 只能派生新的 session；源 session 保持只读，同一源完整性只能 claim 一次。未进入 handler 的调用补入明确的未执行结果；已准入调用一律记录为不确定事实，不自动重放。所有 issue 必须逐项由用户 `/resolve`；调查只允许获准的无副作用观察，全部 `continue` 后必须重新规划、重新授权并独立验证。恢复期间旧 PID、stdin 和当前验证资格不可继承。

完成提醒硬约束：当 active Plan Contract 步骤未完成或仍需验证时，阶段性文本只触发当前
`progress_marker` 一次 Runtime Notice；计划状态、非计划工具事实、验证证据、
generation 或 `verification_required` 发生变化后才允许再次提醒。标记不变而再次
输出无 `tool_calls` 文本时必须将任务置为 `blocked`。Runtime Notice 要求下一回复
调用推进工具；确实无法继续时才说明具体阻塞原因。没有 `progress_marker` 的旧式
State 保持一次提醒兼容行为。

活动后台进程属于当前 `task_id`，必须阻止任务进入 `done`；stdin 写入在途时也必须阻止完成。模型无工具调用而进程仍运行或 stdin 写入未收束时使用 `awaiting_process` 交回 CLI；用户恢复前先同步进程。`/new`、`/reset`、EOF、`exit` 和异常退出必须先有界清理当前任务登记的进程及写入线程；清理不完整时保留旧任务并报告具体进程 ID、PID 和原因。管道 stdin 只有显式启用时可写，单次 UTF-8 输入最多 4096 字节，正文不得进入 State、Trace、工具结果、授权提示或终端输出；PTY 不属于当前能力。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liiiiiiiiil/coding-agent-from-scratch](https://github.com/liiiiiiiiil/coding-agent-from-scratch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
