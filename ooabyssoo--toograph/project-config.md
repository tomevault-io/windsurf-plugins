---
trigger: always_on
description: 这些说明适用于本仓库中的所有工作，并应在新的 Codex 会话中持续生效。
---

# TooGraph Agent 说明

这些说明适用于本仓库中的所有工作，并应在新的 Codex 会话中持续生效。

## 分支工作流

- 所有开发工作都应在本地 `dev` 分支上进行。
- `dev` 分支只保留在本地，不设置 upstream，不同步远端，也不要创建远端 `dev` 分支，除非用户明确改变这条规则。
- `main` 用作与 `origin/main` 对齐的远端基线分支。开始开发前，如果当前不在 `dev`，应从当前 `main` 创建或切换到本地 `dev`。
- 仓库改动和本地提交默认落在 `dev` 分支；不要把日常开发提交直接放到 `main`。

## 提交信息

- 为本项目创建 git commit 时，提交信息必须使用中文。
- 修改仓库后，默认不要自动创建 git commit；只有当用户明确要求提交或明确批准提交时，才提交到本地 git。
- 除非用户明确要求推送或批准推送，否则不要把 commit 推送到远端。

## 启动流程

- 修改代码后，使用仓库标准跨平台命令 `npm start` 重启 TooGraph。
- `node scripts/start.mjs` 是仓库底层标准启动命令；`npm start` 应解析到它。
- TooGraph 使用单端口启动模型。默认公开地址是 `http://127.0.0.1:3477`，可用 `PORT=<port> npm start` 覆盖端口。
- 本地验证和截图优先使用默认地址 `http://127.0.0.1:3477`；不要仅因为 3477 被占用就换到其他端口。
- `npm start`、`scripts/start.mjs`、`scripts/start.sh` 应在启动前释放占用配置端口的既有 TooGraph 进程。如果启动报告 3477 被一个无法安全识别为 TooGraph 的进程占用，应停止并报告 PID/命令，而不是在第二个端口启动 TooGraph。
- 只有当用户明确要求使用备用端口，或在看到 3477 无法复用的原因后批准临时 fallback 时，才使用 `PORT=<port> npm start`。验证结束后不要留下备用端口上的 TooGraph 实例。
- 当 `frontend/dist` 的构建 manifest hash 与当前前端输入一致时，`npm start` 应复用已有构建而不是每次启动都重新构建。需要强制重新构建时使用 `TOOGRAPH_FORCE_FRONTEND_BUILD=1 npm start`。
- `npm run dev` 不是受支持的项目命令。
- 在 Windows PowerShell 中，如果执行策略阻止 `npm.ps1`，使用 `npm.cmd start`。
- `scripts/start.sh` 仍是 Linux、macOS、Git Bash 和 WSL 的标准 Bash 包装脚本，并应与 `scripts/start.mjs` 保持行为一致。
- 如果任务只涉及文档或其他非运行时代码变更，可以按实际情况判断；对代码变更，默认使用上述标准启动流程重启。

## 本地 LLM 运行时

- 本地 LLM 和运行时说明应统一放在 Model Providers 页面。
- 首选本地或私有网关流程：
  - 启动要使用的 OpenAI-compatible gateway。
  - 在 Model Providers 页面配置 `local` / Custom OpenAI-compatible Provider；当前本地默认 base URL 是 `http://127.0.0.1:1234/v1`。
  - 在 UI 中保存或发现模型列表，然后在那里选择默认文本模型。
- 模型执行只读取已保存的 Model Providers 配置和 UI 中的默认模型选择。不要重新文档化或恢复通过启动环境变量配置模型 provider 的方式。
- TooGraph 自身启动说明仍然是 `npm start` 和 `node scripts/start.mjs`；这些命令不会被本地运行时说明替代。

## UI 实现策略

- UI 工作应始终优先使用项目已经采用的组件库，再考虑自定义组件或控件。
- 只有当现有库无法合理满足需求，或确实需要自定义行为时，才手写 UI。
- 必须自定义 UI 时，应尽量缩小自定义层，并尽可能构建在现有库 primitives 之上。

## 用户体验与视觉质量

- 用户体验和视觉质量是每个用户可见改动的必需部分，不是可选润色。
- 修改 UI 前，检查附近页面和组件，遵循现有美术方向、间距节奏、颜色、字体、图标风格、动效和交互语言。
- 不要交付原始浏览器默认控件、拥挤布局、不清晰标签、意外视觉退化，或只有实现者自己知道怎么用的流程。
- 每个用户可见流程都应包含清晰 affordance、必要的加载/保存/成功/错误反馈，并避免令人意外的状态变化。
- 对重要 UI 改动，除运行测试外，也应尽量通过浏览器截图进行视觉验证。

## 产品与工程质量

- 变更范围应贴合请求，但被触及的区域要保持一致和清晰：移除令人困惑的重复、过时 UI 状态，以及本次工作引入或暴露出的明显隐患。
- 添加任何功能前，必须先查看原有架构、数据链路和附近实现。优先复用既有接口、协议路径、存储 API、校验器、命令总线、图/运行时 primitives 和 UI 模式。不能为了单独完成功能需求而添加产品特化的特殊代码、旁路或不符合架构设计的链路；架构一致性和清晰的数据/控制流比单个功能“能跑”更重要。一个功能即使表面完成，但违反仓库架构、重复已有接口，或形成不合理的执行链路，也不能接受。
- 保护用户数据和本地配置。除非明确要求，不要提交本地运行状态、日志、生成构建产物、凭据或机器特定设置。
- 将 `backend/data/settings`、`.toograph_*`、`.dev_*` 日志、`dist` 和 `.worktrees` 视为本地/运行时产物，除非任务明确针对它们。
- 当自动化、可发现的行为能改善用户工作流时，优先采用它，但副作用必须可见且可逆。
- 完成前，为改动面运行最小但有意义的验证集；UI 工作在可行时包含视觉检查。清楚说明跳过或失败的验证。

## 图优先产品架构

- TooGraph 产品行为应在可行时由图模板表达。持久化操作、本地文件编辑、记忆更新、伙伴自配置和其他副作用，应因为指定图/模板运行而发生，而不是由隐藏的产品特化命令式代码决定。
- 保持节点职责清晰：
  - 整张图才是 Agent。不要把单个节点当成自治的多步骤 agent。
  - LLM 节点执行一次模型回合。它们负责推理、分类、规划、生成结构化 state，或准备一次 capability 调用。
  - 一个 LLM 节点最多只能使用一个显式 capability 来源：无 capability、一个已选 Action，或一个输入的 `capability` state。`capability` state 是单个互斥对象，其 `kind` 为 `action`、`subgraph`、`tool` 或 `none`；不能是列表。如果工作流需要多个 capability，用多个节点和边表达顺序。
  - 手动选择的 LLM-node Action 必须存储为标量 `config.actionKey`，绝不能存为 `config.skills` 或任何数组。数组意味着多 capability 语义，属于旧的无效协议。
  - 当 LLM 节点使用 Action 或动态 Subgraph 时，LLM 只在执行前准备调用输入。运行时执行 capability，并把原始结构化输出写入 state；同一个 LLM 节点不应总结、重包或继续发起后续 capability 调用。
  - 静态手动选择的 Action 使用 `config.actionKey` 和协议拥有的 `actionBindings.outputMapping`。该映射由图/编辑器/运行时创建，在运行审计详情中可见，不应暴露为由 LLM 选择或重写的内容。
  - Action instruction capsule 只是节点级 override 面。默认 capsule 来自选中 Action manifest 的 `llmInstruction`；只有用户编辑后的文本才持久化为 `actionInstructionBlocks.<actionKey>`，并带有 `source: "node.override"`。运行时只有一份有效 Action-use instruction：节点 override 优先，否则使用 manifest `llmInstruction`，注入到 Action-input planning system prompt 中，不在用户 prompt 中重复。
  - Action lifecycle scripts 使用固定文件名，而不是 manifest entrypoint 配置。如果存在 `before_llm.py`，运行时在 Action-input planning 前执行它，并把可审计上下文注入 LLM prompt。如果存在 `after_llm.py`，运行时在 LLM 产出结构化 Action 参数后执行它，并将其 JSON 结果视为 Action 输出。State binding 仍由运行时通过 `stateOutputSchema` 和 `actionBindings.outputMapping` 拥有；lifecycle scripts 不得直接写图 state。
  - 来自输入 `capability` state 的动态 capability 执行必须准确写入一个 `result_package` state。该包将输出包装为 `outputs.<outputKey> = { name, description, type, value }`；不要添加冗余的 `fieldKey` 字段。下游 LLM prompt assembly 解包这些虚拟输出后，使用与静态 state 相同的渲染规则。
  - 手动复用图嵌入属于 Subgraph 节点。`capability.kind=subgraph` 用于 Buddy loop 等模板内部的动态图 capability 选择，不应作为普通 LLM 节点卡片上的下拉选项。
  - 大型 Buddy 或自动化模板应在可行时把稳定阶段拆为 Subgraph 节点。优先使用可读的顶层图流，并通过可检查子图表达上下文打包、capability loop 和最终回复生成，而不是把所有逻辑塞进一张拥挤画布。回复后的 self-review 应作为一个独立、可审计的后台图/模板，从已完成 run snapshot 运行，不要延长可见回复路径并阻塞下一轮用户输入。
  - Action 节点执行受控 capability 和副作用，例如写本地文件、更新记忆存储、下载资源或创建 revision。
  - Output 节点负责显示、预览、导出或链接结果。它们不应拥有持久化 mutation 逻辑。
- Buddy 聊天运行胶囊必须只按 output 边界分段。边界是直接连接一个或多个 `output` 节点的上游非 output 节点；一个胶囊展示从上一个 output 边界之后到当前边界节点为止的运行过程，所有连接到同一个边界节点的 output 都跟在这个胶囊后展示。不要为父图决策分支、内部能力选择节点或其他没有连接 output 的图内部节点创建额外 Buddy 胶囊。例如线性流程 `A -> B -> C -> D -> E` 中，如果 `C` 连接两个 output 节点，`E` 连接一个 output 节点，应展示一个 `A, B, C` 胶囊和 C 的两个输出，再展示一个 `D, E` 胶囊和 E 的一个输出。
- 后端代码应提供可复用 primitives、存储 API、校验器、revision 机制和 Action runtime。避免把 Buddy memory policy、persona 更新规则或工作流决策等产品行为埋进后端 endpoint；当行为可以表达为图/模板时，应通过图/模板表达。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OoABYSSoO/TooGraph](https://github.com/OoABYSSoO/TooGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
