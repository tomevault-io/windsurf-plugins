---
trigger: always_on
description: > 本文件面向参与 Mutsumi 开发的 AI 编码 Agent，介绍各模块职责、模块间的接线方式，以及非平凡/反直觉的设计约束。
---

# Mutsumi 项目开发指南（Agent 贡献者版）

> 本文件面向参与 Mutsumi 开发的 AI 编码 Agent，介绍各模块职责、模块间的接线方式，以及非平凡/反直觉的设计约束。
> Mutsumi（若叶睦）是一款 VS Code 多 Agent 笔记本环境插件：Agent 会话以 `.mtm`（JSON）文件持久化，经 VS Code Notebook API 呈现，工具/上下文/编排围绕该模型展开。

---

## 1. 架构总览

```
src/
├── adapters/          # 适配器层：把 AgentRunner 与 UI/传输解耦（Notebook / HTTP / Lite）
├── agent/             # Agent 核心逻辑（执行循环、LLM 客户端、编排、标题生成）
├── codebase/          # 代码库服务（RAG 向量搜索）
├── config/            # 配置系统（AgentType、toolSets、MCP Server 的合并/校验/解析）
├── contextManagement/ # 动态上下文管理（模板引擎、历史装配、幽灵块、Skills）
├── httpServer/        # HTTP API 服务（复用同一套 Agent 执行链路）
├── mcp/               # MCP 宿主（连接 registry、ITool 适配、结果投影）
├── notebook/          # Notebook UI 实现（serializer、自定义渲染器、工具栏命令）
├── sidebar/           # 侧边栏视图（Agent / 审批 / Context / Shell 任务）
├── registry/          # 配置运行时注册表（ToolSetRegistry、AgentTypeRegistry）
├── tools.d/           # 内置工具实现与工具运行时（ToolRegistry、ToolSet、权限）
└── types.ts / utils.ts / i18n.ts / controller.ts / extension.ts
```

### 核心分层原则

1. **适配器解耦**：`AgentRunner` 只依赖 `IAgentSession` 接口，不感知 Notebook 还是 HTTP。Notebook/HTTP 各自通过 adapter 复用同一执行链路。
2. **URI 优先**：所有文件操作使用 `vscode.Uri`，支持多根工作区与其他扩展的 FileSystemProvider 特殊 schema。Mutsumi 自身数据（`.mutsumi/`）固定在工作区列表 `[0]`。
3. **工具分层**：内置工具（静态注册）与 MCP 工具（动态发现）是两套体系，在 ToolSet 构建时组合；`task_finish` 独立于一切工具集配置。
4. **装配点唯一**：`extension.ts` 是唯一激活/装配入口（初始化顺序、配置监听、事件订阅都在这里）。
5. **禁止反向依赖**：`mcp` 不 import `sidebar`/`notebook`；`serializer`/`fileOps` 不发起 MCP 连接；`ToolExecutor` 对 MCP 零特判；sidebar 只读依赖 mcp registry 状态。

---

## 2. Notebook 系统与 UI 层

### 2.1 `.mtm` 文件模型与 Serializer（`notebook/serializer.ts`）

`.mtm` 是 JSON：`{ metadata: AgentMetadata, context: AgentMessage[] }`。VS Code 通过 `NotebookSerializer` 把它与 Notebook 文档互相转换。

核心算法是 **messages ↔ generic cells 双向映射**（`messagesToGenericCells` / `genericCellsToMessages`），并被 HeadlessAdapter 复用，因此协议是"与 UI 无关"的：

- User 消息 → Code cell（kind 2）
- Assistant 消息 → Markup cell（kind 1）
- **紧随 user 的 assistant/tool 消息组不单独建 cell**，而是存入该 user cell 的 `mutsumi_interaction` metadata，渲染为该 cell 的输出区（这是最反直觉的点）
- System 消息 → 带 `**System**: ` 前缀的 Markup cell，反序列化时剥前缀
- 孤儿 assistant/tool 消息（无前置 user）→ 直接拍平成 markdown 存 cell value，**不写** `mutsumi_interaction`
- `mutsumi_interaction` **只存在于 user cell**，永不写在 assistant cell 上
- cell value 中保存的幽灵块 markdown 在反序列化时剥离（`stripGhostBlockFromCell`），结构化版本在 metadata 中

序列化时 `sub_agents_list` 与 `AgentOrchestrator` 内存注册表**双向同步**（打开时注入 childIds，保存时回写）。

### 2.2 自定义渲染器（`notebook/renderer.ts` + `renderTypes.ts`）

Agent 输出不是 HTML 字符串，而是**结构化中间表示 `RenderData`**（`MUTSUMI_AGENT_CHAT_MIME`）：

```text
RenderData = { committed: RenderBlock[], active: {...} | null }
RenderBlock = content | reasoning(collapsed) | toolCall(isStreaming, result?)
```

渲染器对 `committed` 做 DOM 缓存（渲染一次、永不再渲染），只重渲 `active` 流式区。**协议方（adapter）只负责产出 RenderData JSON**，HTML 生成全部在渲染器侧。

### 2.3 UIRenderer 三级锁定（`agent/uiRenderer.ts`）

`AgentRunner` 侧的流式状态机，保证"已提交块不重渲"：

- **L1（轮次）**：`commitRoundUI()` 把整轮剩余内容锁入 committed
- **L2（轮内）**：content 出现 → reasoning 锁定；tools 出现 → content 锁定
- **L3（工具）**：`appendBlock()` 提交单个完成的工具调用

### 2.4 工具栏与命令（`notebook/toolbar.ts` + `commands/`）

工具栏命令（selectModel、renameSession、debugContext、toggleAutoApprove、testRagSearch、compressConversation、pruneGhostBlocks）经 `registerToolbarCommands` 统一注册；每个命令一个文件，依赖 `buildInteractionHistory` 或 metadata 读写。

命令式交互遵循的通用模式：

- 读 metadata → 不可变展开 → `WorkspaceEdit` 写回
- 涉及上下文缩短的操作（pruneGhostBlocks、removeFile、MCP 开关）默认使前缀缓存失效（见 §5.2）

### 2.5 侧边栏（`sidebar/`）——四个 TreeView 的设计模式

| View | Provider | 数据源 | 结构 |
|------|----------|--------|------|
| Agents | `agentTreeProvider` | `AgentOrchestrator`（内存注册表 + childIds） | 树：parent → children，双向引用 |
| Approvals | `approvalTreeProvider` | `approvalManager`（permission.ts） | 平铺，pending 优先、新的在前 |
| Context | `contextTreeProvider` | Notebook metadata + 文件系统 + SkillManager + McpRegistry | 分类树 |
| ShellTasks | `shellTaskTreeProvider` | shell 任务注册表 | 平铺 + 状态 |

统一的 TreeView 模式：

- provider 持有 `_onDidChangeTreeData` EventEmitter，`refresh()` 即 `fire(null)`
- **子节点缓存在 item.children 上**：`getChildren(element)` 大部分只是返回 `element.children`（只有根节点才构建）；ContextTree 的 `getChildren` 必须按 `element.data.type` 分发（category/directory/mcpServer 返回 children，其余为空）
- 数据变化来源多样（运行状态、审批、metadata、registry、配置），刷新触发点分散在 `AgentSidebarProvider.registerTreeView` 的订阅里
- `AgentSidebarProvider.update()` 是聚合刷新入口（agent + context + shell 三棵树）

**ContextTree 分类**（`contextTreeItem.ts`）：AGENT TYPE 节点 + RULES / SKILLS / MACROS / FILES / MCPS 分类。`ContextItemType`/`CategoryType` 是判别联合，新增分类必须同步：类型定义、`getIconPath`、`getContextValue`（决定菜单 `when`）、`buildTooltip`、package.json `view/item/context` 菜单、i18n。

菜单机制：树项的 `contextValue` 与 package.json `menus["view/item/context"]` 的 `when: viewItem == xxx` 匹配决定显示哪些内联命令；"只读态"用独立 contextValue（如 `mcpServerReadOnly`）隐藏操作。

---

## 3. Agent 执行核心（`agent/`）

### 3.1 执行入口：controller → runner（`controller.ts` → `agentRunner.ts`）

Notebook 执行链路（`AgentController.execute`）：

```text
用户运行 cell
  → notifyAgentStarted(uuid)
  → processCell：解析模型（metadata 完整对 → 用；缺 model → 全局默认；有 model 无 provider → 迁移错误）
  → 取凭据（getModelCredentials）
  → NotebookAdapter.createSession
  → createToolSetForAgent(metadata 快照)   ← 内置 + MCP + task_finish 在此组合
  → buildInteractionHistory(session)       ← 装配上下文（见 §5）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NERDSORG/Mutsumi](https://github.com/NERDSORG/Mutsumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
