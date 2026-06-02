---
trigger: always_on
description: 仓库给 Agent / 新线程使用的首读入口。详细行为描述见 `docs/agents-reference.md`。
---

# AGENTS.md

仓库给 Agent / 新线程使用的首读入口。详细行为描述见 `docs/agents-reference.md`。

## 信息优先级

1. 代码实际行为 > 2. `AGENTS.md` > 3. `README.md` > 4. `CLAUDE.md`

## 项目快照

- 项目名：`t-code`
- 定位：面向商业使用的 Java Agent CLI 产品，对标 Claude Code
- 已交付 21 期（ReAct → Plan+DAG → Memory → RAG → Multi-Agent → HITL → 并行工具 → 多模型 → 联网 → MCP 核心 → MCP 高级 → 长上下文 → Chrome DevTools → CDP 会话复用 → Skill → TUI → LSP 诊断 → Side-Git 快照 → Prompt 分层 → Runtime API → 图片输入）
- 下一步：OAuth / sampling / recovery 作为后续 MCP 增强
- 公开版本：`v1.0.0`，Maven 产物：`t-code-1.0-SNAPSHOT.jar`

## 运行前提

- Java 17+ / Maven
- 至少一个 API Key：`GLM_API_KEY` / `DEEPSEEK_API_KEY` / `STEP_API_KEY` / `KIMI_API_KEY`

## 常用命令

```bash
cp .env.example .env
mvn clean package        # 默认跳过测试，优先产出可手工验收 jar
java -jar target/t-code-1.0-SNAPSHOT.jar
mvn test -Pquick          # 常规回归
mvn test -Pphase16-smoke  # TUI 相关
mvn test -Dtest=XxxTest -DskipTests=false   # 针对性
mvn test -DskipTests=false                  # 全量回归
```

## 架构概览

三条主执行路径，共享 ToolRegistry / MemoryManager / SnapshotService：

| 路径 | 入口 | 触发 |
|------|------|------|
| ReAct | `Agent.java` | 默认模式 |
| Plan-and-Execute | `PlanExecuteAgent.java` | `/plan` |
| Multi-Agent | `AgentOrchestrator.java` | `/team` |

核心内置工具 11 个：`read_file` / `write_file` / `list_dir` / `glob_files` / `grep_code` / `execute_command` / `create_project` / `search_code` / `web_search` / `web_fetch` / `revert_turn`

`ToolRegistry` 已提供 `ToolProvider` / `ToolRegistrationContext` 扩展口；内置工具声明已全部迁移为独立 provider（File / FileSearch / Project / Shell / RAG / Web / Browser / Memory / Skill / Snapshot）。文件操作、项目脚手架、Shell、实时文件搜索、RAG 检索与 Web 访问实现已分别下沉到 `FileService`、`ProjectScaffolder`、`ShellService`、`FileSearchService`、`RagSearchService` 和 `WebService`；其余实现可继续从 Registry 私有方法逐步下沉，不要把新工具声明直接堆进 `ToolRegistry` 大类。

Browser / Memory / Skill 的可变依赖统一通过 `ToolRuntimeBindings` 管理；Provider 注册、参数 schema 和 LLM 工具定义导出统一通过 `ToolDefinitionCatalog` 管理；MCP 动态工具 descriptor / invoker 目录统一通过 `McpToolCatalog` 管理；并行工具批次、顺序回收、超时和取消处理统一通过 `ToolBatchExecutor` 管理；单工具执行、审计和浏览器策略挂钩统一通过 `ToolExecutionPipeline` 管理。`ToolRegistry` 对外保留兼容 setter、MCP 注册 API、`executeTool()` 与 `executeTools()`，内部主要负责组装和 facade 转发。

`CoreRuntime` 会把 `ToolRegistry` 生命周期事件桥接到 Runtime API：工具执行前发 `tool.started`，执行完成后发 `tool.completed`。配置 HITL handler 时，还会桥接 `hitl.requested` / `hitl.resolved` 观察事件。核心事件 payload 统一通过 `RuntimeEventPayloads` 构造并携带 `schema_version: 1`；TypeScript CLI 解析 SSE 时显式暴露 `schemaVersion`，缺少版本的旧事件按版本 `1` 兼容，并使用 thread 级 `RuntimeEventCursor` 增量消费事件，不能在每轮 turn 内把 `after` 重置为 `0`。HTTP 模式使用 `RuntimeHitlHandler` 保存待审批项，开放 `GET /v1/approvals` 与 `POST /v1/approvals/{id}/decision`；成功与错误 JSON 响应统一通过 `RuntimeApiResponses` 构造，错误格式为 `{"error":{"code":"...","message":"..."}}`。`RuntimeApiServer` 只保留 HTTP 适配；thread / turn / events 和 approval 路由分别由 `RuntimeThreadRoutes` 与 `RuntimeApprovalRoutes` 管理，并通过 `RuntimeApiRequest` / `RuntimeApiResult` 与网络层连接；Bearer 与 `X-TCode-API-Key` 鉴权规则、API key 配置读取统一由 `RuntimeApiAuthPolicy` 管理。TypeScript CLI 通过 `RuntimeApiError` 兼容消费新结构和旧字符串错误，并通过 `RuntimeRequestRetrier` 只重试 events / approvals 的幂等 GET 网络异常；`createThread`、`submitTurn`、审批决策写请求不得自动重放。事件监听属于 fail-soft 观察侧，写事件失败不能阻断工具主路径。

交互 CLI 的 ReAct / Plan / Team Agent 装配统一走 `CliAgentFactory`：共享现有 `ToolRegistry` 与 `MemoryManager`，注入 MCP resource prompt 上下文，并把 Skill registry / buffer 同步到工具层和 Agent 层。不要在 `Main` 中重新散落这组装配逻辑。

交互 CLI 的 HITL、Browser session、Browser guard、Browser connector 与 MCP manager 统一由 `CliSessionInfrastructure` 创建。`Main` 负责按启动阶段加载和启动 MCP server，但不再直接拼装这些基础设施依赖。

交互 CLI 的 Skill 缓存目录、内置 Skill 解压、状态存储、registry reload 与 context buffer 创建统一由 `CliSkillInfrastructure` 管理。`Main` 只消费初始化结果，并把可选启动提示合并进首屏。

交互 CLI 的 MCP 配置 bootstrap、server 加载与限时启动、shutdown hook、MCP resource mention expander 和本地路径 mention expander 统一由 `CliMcpInfrastructure` 管理。后台任务 manager 的打开、启动和 shutdown hook 统一由 `CliTaskInfrastructure` 管理。

交互 CLI 的 `LineReader` 构造统一走 `CliLineReaderFactory`：持久化 history、completer、highlighter、基础 option 和 slash / autosuggestion / autopair widgets 在这里集中配置。Renderer 绑定和运行期快捷键仍按启动阶段留在 `Main`。

交互 CLI 的 Renderer 创建、Renderer HITL delegate、inline LineReader 绑定、`Renderer.start()` 与首个状态更新统一由 `CliRendererInfrastructure` 管理。首屏内容安装和运行期快捷键仍按启动阶段留在 `Main`。

交互 CLI 的首屏安装和运行期快捷键 wiring 统一由 `CliInteractiveUiInstaller` 管理：inline 模式把首屏挂到 `CALLBACK_INIT`，降级渲染器直接输出；运行期为 inline 绑定 Ctrl+O，并为所有模式绑定 Ctrl+V 与 Esc。

交互 CLI 的低耦合控制命令统一由 `CliControlCommandDispatcher` 分发：输入历史清理、HITL 开关、Memory、Policy、Audit、Snapshot、MCP 管理、Browser、后台 Task 和 Skill 管理不再堆在 `Main` 的主循环中。会改变 Agent 执行模式或依赖对话状态的命令仍由 `Main` 协调。

交互 CLI 的 RAG 代码检索命令统一由 `CliCodeSearchCommandDispatcher` 分发：`/index`、`/search` 与 `/graph` 不再堆在 `Main` 的主循环中。`/index` 完成后仍需同步更新 ToolRegistry 与 MemoryManager 的项目路径。

交互 CLI 的模型切换统一由 `CliModelCommandDispatcher` 分发：`/model` 状态展示、provider 解析、client 创建、配置保存与 Agent client 更新不再堆在 `Main`。只读 `/config` palette 统一由 `CliConfigCommandDispatcher` 分发。

交互 CLI 的 `/browser` 子命令实现统一由 `CliBrowserCommandHandler` 管理：status、autoConnect、旧式端口连接、disconnect 与 tabs 不再作为 `Main` 私有 helper；`CliControlCommandDispatcher` 和 `CliSessionInfrastructure` 共享同一入口。

交互 CLI 的会话级命令统一由 `CliConversationCommandDispatcher` 分发：`/cancel`、`/clear` 与 `/context` 不再堆在 `Main` 的主循环中。`Main` 的命令 switch 只保留 unknown、exit 与 Plan/Team 模式协调。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OperationT00/T-Code](https://github.com/OperationT00/T-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
