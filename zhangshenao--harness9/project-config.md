---
trigger: always_on
description: harness9 是一款基于 Go 语言构建的**轻量级、功能完备、生产可用**的 Agent Harness 框架，旨在提供简洁、高效、可扩展的 Agent 编排能力。
---

# AGENTS.md — harness9 项目开发指南

## 1. 项目概述

harness9 是一款基于 Go 语言构建的**轻量级、功能完备、生产可用**的 Agent Harness 框架，旨在提供简洁、高效、可扩展的 Agent 编排能力。

### 核心设计理念

| 原则 | 说明 |
|------|------|
| **简洁** | 最小化抽象层，代码直白易读；极少的直接依赖数 |
| **完备** | 覆盖 Agent 运行所需的全部核心模块（Engine / Provider / Schema / Tools / Env） |
| **生产可用** | 错误恢复、上下文管理、超时控制、并发工具执行、Path Traversal 防护等生产级特性 |

### 核心架构

- **标准 ReAct**: 每个 Turn 执行一次 LLM 调用（携带完整工具列表），工具调用结果作为 Observation 注入上下文
- **并发工具执行**: 同 Turn 内多个工具调用并发执行，每工具独立超时控制
- **双模式运行**: 阻塞式 `Run` + 流式 `RunStream`，共享同一引擎实例
- **自愈能力**: 工具执行失败时，错误信息原样回传给 LLM，触发自动重试
- **双重压缩策略**: SummarizationCompactor（默认，LLM 摘要，保留语义 + 增量更新）和 TokenBudgetCompactor（回退，字符截断），均在 80% 阈值触发，双向修复孤立工具对
- **实际 Token 用量**: 从 API 响应 usage 字段提取，LLM 调用后实时更新 TUI 展示
- **Planning（先规划后执行）**: Plan Mode（工具层权限过滤）+ TodoStore（状态机校验）+ 自动续跑 + 停滞检测
- **Sub-Agent（子代理委派）**: 主代理通过 `task` 工具把边界清晰的子任务委派给运行在隔离 Session 上的专门子代理（独立上下文 + 受限工具集 + 可选模型覆盖）；内置 general-purpose 通用子代理（对标 Claude Code / DeepAgents，继承父全部可用工具与模型），支持 `.harness9/agents/*.md` 文件式定义、前台/后台双模式、`@agent` 直跑、TaskTracker 后台任务管理；安全保障：禁止递归 + 权限只能更严不能扩权 + 上下文完全隔离
- **文件系统能力**: OffloadHook（超大工具输出自动写入文件，context 保留摘要引用 + 分页检索）+ FilePlanWriter（todo 计划持久化到 markdown）+ DeleteSession 级联 GC
- **推理内容展示（Reasoning Display）**: Anthropic extended thinking（StreamChunkThinkingDelta）和 OpenRouter/DeepSeek reasoning_content 均路由为 EventThinkingDelta，TUI 以 `│` 前缀深灰色块流式渲染，与正文回复形成视觉层次区分
- **Shell 执行（`!` 前缀）**: 输入框以 `!` 开头进入 Shell 模式，命令通过 `bash -c` 异步执行（30s 超时），输出 inline 追加到对话流，并在下次 LLM dispatch 时前置注入为上下文；已知交互式命令（vim/ssh 等）自动拦截
- **Human-in-the-Loop 权限控制**: HookDecision（allow/deny/ask）三级决策 + DangerHook（19 条高危模式）+ PermissionHook（JSON 白名单，按需重载）+ 敏感路径硬保护（~/.ssh、~/.aws 等）+ TUI 五选项审批对话框 + PermissionMode 枚举
- **Long-Term Memory（跨会话长期记忆）**: `internal/ltm/` 包；SQLite `long_term_memories` + standalone FTS5 `memories_fts`，复用 `state.db` 连接；MEMORY.md 物化视图（top-N 有界注入，≤5KB，规避 token bomb）+ `memory_search` 按需 FTS5 检索；三路触发：显式 `memory_write`/`memory_search` 工具 + 压缩前 `Extractor`（LLM 提取，fail-open）+ `WithMemoryNudge`（每 N 轮注入提示，防御性副本，不持久化）；SHA256 内容签名去重 + TTL 过期 + 软删除（signature=NULL 释放槽位）+ 命中强化（use_count/last_used_at）+ 陈旧识别（`StaleCandidates`）；Phase 3 接缝：Provider/Embedder/Consolidator 接口 + noopProvider
- **Sandbox（Docker 容器级隔离）**: `internal/sandbox/` 包；`Environment` 接口（LocalEnvironment 进程级 / DockerEnvironment 容器级）+ `Container` 五状态生命周期（Pending→Running→Stopping→Terminated/Failed）+ `Manager`（Create/Destroy/DestroyAll/ReapOrphans/ListAll，并发安全）；工具透明路由（bash 命令通过 docker exec 进容器，文件工具通过 bind mount 共享 workDir）；安全加固：`--cap-drop all` + `--cap-add DAC_OVERRIDE/SETUID/SETGID`（包管理器所需最小能力）+ `--security-opt no-new-privileges:true` + `--pids-limit 256` + tmpfs nosuid/noexec/nodev；Agent 级隔离（主 Agent 和每个 Sub-Agent 各自拥有独立容器）；TUI SandboxBar 实时展示状态（颜色编码）；`label=harness9=1` 标记 + 启动时孤儿回收；`SANDBOX_ENABLED=true` 启用，关闭时行为与引入前完全一致（向后兼容）
- **Observability（OpenTelemetry 可观测性）**: `internal/observability/` 包；三条非侵入式接入路径——`OTELEngineObserver`（实现 `EngineObserver` 接口，管理 Interaction Span + Turn Span）+ `TracingProvider`（包装 `LLMProvider`，为每次 LLM 调用创建 Span + Token Metrics）+ `ObservabilityHook`（实现 `ToolHook`，为每次工具调用创建 Span）；Span 四层嵌套：`harness9.interaction → harness9.turn → harness9.llm_request / harness9.tool`；6 个关键 Metrics（LLM 延迟/Token 消耗/工具调用次数/工具执行耗时/Turn 总数）；`langfuse.trace.input`（trace 根节点 prompt）/ `langfuse.observation.input/output`（observation 层 LLM 消息与回复 + 工具参数与结果）/ `gen_ai.usage.*`（Token 用量，Langfuse 自动换算费用）；非法 UTF-8 字节自动净化，防止 OTLP 序列化失败；三种 Exporter：`noop`（默认零开销）/ `stdout`（开发调试）/ `otlp`（生产接入 Langfuse / Grafana / Jaeger）；通过环境变量 `OTEL_ENABLED` / `OTEL_EXPORTER_TYPE` / `OTEL_EXPORTER_OTLP_ENDPOINT` 驱动，默认关闭向后兼容
- **网页搜索与抓取**: `internal/tools/web_search.go` / `web_fetch.go` / `web_safety.go` / `web_content.go`；`web_search` 工具（DuckDuckGo HTML 端点 POST，无 API Key，20s 超时 + 10s dial 超时，`golang.org/x/net/html` DOM 解析，`decodeUDDG` 还原真实 URL）+ `web_fetch` 工具（HTTP GET，15s 超时，5 次重定向上限，`text/html` → `go-readability` 提取主内容 → `html-to-markdown` 转 Markdown，`text/*` → 原始文本，其他 → 不支持提示）；`isSafeURL` 共享 SSRF 安全门（scheme + userinfo + DNS 解析 + 9 个 IP 段检查（含 IPv6 ULA `fc00::/7` 和链路本地 `fe80::/10`）+ IPv4-mapped IPv6 规范化 + IPv6 loopback，DNS 失败 fail-closed，重定向链每跳复检）；`DefaultPromptBuilder` 实时注入 `当前日期：YYYY-MM-DD`，防止 LLM 因训练截止日期偏差产生陈旧搜索词；主 Agent + 所有 Sub-Agent 均可使用，零额外配置
- **Test & Eval（自动化测试与评估）**: `internal/evals/` 包；`ScriptedProvider`（确定性 LLM mock，按预设 Turn 序列返回回复，不发起真实 API 调用）+ `Assertion` 接口（Hard 断言：ToolCalled/ToolNotCalled/OutputContains/OutputExcludes/NoError/Error；Soft 断言：MaxTurns/MaxToolCalls，仅记警告不影响通过率）+ `EvalHarness`（`RunCase` 构建最小化隔离引擎 + `recordingHook` 记录工具调用轨迹 + `Suite` 批量运行）+ `SetupHermeticEnv`（清除所有 API Key，标准 Hermetic 隔离环境（密封测试，防止 eval 调用真实 API），本地与 CI 环境一致）+ `BuildReport`/`WriteJSON`/`WriteMarkdown`（JSON + Markdown 评估报告生成）；`internal/evals/dataset/` 黄金数据集（16 个用例：工具调用准确性 × 4 + Planning 完成率 × 4 + Context Engineering × 3 + Error Handling/Self-Healing × 3 + Memory 持久化 × 2）；`.github/workflows/eval.yml` CI Quality Gate（PR 触发 hermetic eval，失败则阻断合并）

### 参考框架

| 框架 | 来源 | GitHub |
|------|------|--------|
| DeepAgents | LangChain | https://github.com/langchain-ai/deepagents |
| OpenHarness | HKUDS | https://github.com/HKUDS/OpenHarness |
| OpenCode | Anomaly | https://github.com/anomalyco/opencode |
| OpenClaw | OpenClaw | https://github.com/openclaw/openclaw |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhangShenao/harness9](https://github.com/ZhangShenao/harness9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
