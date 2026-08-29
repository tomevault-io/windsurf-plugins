---
trigger: always_on
description: Use Context7 MCP to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library do
---

<!-- context7 -->
Use Context7 MCP to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

## Steps

1. Always start with `resolve-library-id` using the library name and the user's question, unless the user provides an exact library ID in `/org/project` format.
2. Pick the best match (ID format: `/org/project`) by exact name match, description relevance, code snippet count, source reputation, and benchmark score. Use version-specific IDs when the user mentions a version.
3. `query-docs` with the selected library ID and the user's full question.
4. Answer using the fetched docs.
<!-- context7 -->

--- project-doc ---

# AGENTS.md - 模镜协作与 Harness Engineering 规则

本文件是模镜仓库内 AI Agent、人类开发者和自动化任务的项目级操作说明。任何代码生成、重构、测试、提交和发布都必须优先遵守本文档。

最后更新日期：2026-08-25
维护人：模镜团队

## 1. 项目边界

模镜 ModelMirror 是 AI 资源浏览与协作平台，当前主要模块包括：

- 前端：React + TypeScript + Tailwind CSS + Vite。
- 后端：FastAPI + httpx + Pydantic。
- 模型调用：优先通过 `LLM_GATEWAY_URL` / `LLM_GATEWAY_KEY` 接入 newAPI 或其他 OpenAI 兼容网关，未配置时回退 OpenRouter。
- 聊天：`/api/chat` 使用 SSE，支持文本、多模态输入和图片生成模型输出。
- 智能体：`/agents` 是智能体入口，`/agents/studio` 管理可保存、版本化发布的 Xpert，`/agents/meta-agent` 是元智能体任务工作台。
- 工作流：`/workflow` 默认使用经典自研 React Flow 画布，`/workflow-native` 是实验线。
- 协作 Runtime：AgentTask、HandoffExecutor、Conversation Goal 与 RunRegistry 共同提供单进程文件型协作闭环。
- RAG：`/rag` 是本地资料库、版本化 Knowledge Pipeline 与检索增强页面。
- Data X：`/datax` 提供文件快照、语义模型、版本化指标、受限分析查询和指标提案审批。
- Agent Table：`/data-tables` 提供本地类型化业务记录、Schema 版本和人工 CRUD；它不是 Data X 或外部 Database MCP。
- 上下文：Xpert Chat 支持会话附件、文件理解、显式记忆和待确认记忆候选。
- 运行观测：`/runtime` 聚合 MCP、Tool Registry、RunRegistry、Skill 和脱敏环境状态。
- 设置：`/settings` 提供 Provider 控制面和可选的 newAPI 外部管理链接；不得嵌入或代理 newAPI 管理界面。
- 资源页：模型、智能体、MCP/Toolset、Skill、版本化 Prompt Command、声明式 Plugin、专家团。
- 平台自编写：私有 Xpert 可创建版本化 Xpert/Skill 提案；批准只写草稿，发布与安装必须由用户另行确认。

稳定入口：

- `/models`
- `/agents`
- `/agents/meta-agent`
- `/agents/studio`
- `/agents/goals`
- `/agents/xpert/:xpertId/chat`
- `/apps/:appSlug`
- `/chat/:modelId`
- `/workflow`
- `/workflow-native`
- `/rag`
- `/datax`
- `/data-tables`
- `/mcps`
- `/skills`
- `/studio`
- `/toolsets`
- `/runtime`
- `/settings`

## 2. Harness Engineering 原则

Harness Engineering 的意思是：先搭护栏，再做功能。任何变更都必须有明确范围、验证方式、回退路径和可观测结果。

强制原则：

1. 小步交付：一次只改一个可验证目标。
2. 先读代码：实现前必须确认真实文件、接口和数据结构。
3. 先定义验收：每个任务必须有可运行的 acceptance check。
4. 稳定路径优先：实验功能不得替换主入口，除非用户明确要求并完成验证。
5. 可回退：影响主路径的变更必须写明回退方案。
6. 不泄密：不得提交 `.env`、API Key、token、日志中的敏感信息。
7. 不破坏：不得重置、删除或回滚用户未授权的文件。

### 2.1 证据等级

仓库内的工程判断必须标注其证据等级，不得把推测写成事实：

- **已证实事实**：能指向当前仓库中的代码、配置、测试、命令输出或已批准需求。
- **合理推断**：由多个事实推导，但尚未获得产品或运行证据；必须写明推断依据。
- **建议方案**：尚未实施的设计选择，不得写成已经存在的能力。
- **待确认**：缺少负责人决定或可靠证据，必须保留给用户或产品负责人确认。

目标客户、用户故事、商业目标、SLA、组织权限和合规承诺若没有明确输入，统一标记为“待确认”，禁止 AI 自行补写。

### 2.2 任务开工契约

开始修改前必须完成并记录：

1. 当前分支、工作树状态和未跟踪文件。
2. 与任务直接相关的入口、实现、数据结构和测试证据。
3. 本次目标、允许修改路径、禁止修改路径、公共接口变化和数据迁移影响。
4. 最小验收命令、完整回归命令和失败回退方式。
5. 风险等级及是否涉及密钥、网络、文件、子进程、持久化或公开 API。

默认单批最多修改 5 个文件。超过时必须说明为什么无法安全拆分，并保持单一可验收目标。任务卡模板见 `docs/templates/task-card.md`。

### 2.3 停止条件

出现以下任一情况时停止写入并先处理或请求确认：

- 发现用户未说明的同文件冲突，且无法在不覆盖其改动的前提下继续。
- 需求依赖未知产品规则、目标客户、权限模型、数据保留策略或外部契约。
- 必须读取、打印或提交真实密钥才能继续。
- 需要破坏性迁移、删除持久化数据、重写 Git 历史或扩大公共 API。
- 关键测试失败且失败原因不属于本次改动，无法证明继续修改是安全的。
- 实际变更范围显著超过任务卡，或回退路径不再成立。

### 2.4 受保护路径

| 路径或资源 | 保护原因 | 修改要求 |
| --- | --- | --- |
| `server/.env`、任何 token/key 文件 | 真实凭据 | 只允许本地配置，不得暂存、提交或输出原值。 |
| `server/*/storage/`、`new-api-data/`、上传与索引目录 | 用户持久化数据 | 不纳入提交；删除、迁移或清空必须获得明确授权。 |
| `client/package-lock.json`、`server/requirements.txt` | 依赖与供应链 | 只有任务确需依赖时修改，并记录许可证、版本和回退。 |
| `SECURITY.md`、`.github/workflows/` | 安全报告与自动化证据 | 必须保留私密报告入口、最小权限和真实门禁状态；不得把普通检查冒充 required gate。 |
| `docker-compose.yml`、Dockerfile、sidecar 配置 | 部署与隔离边界 | 必须运行 Compose 配置/构建与安全冒烟。 |
| `/api/chat`、classic workflow runner、Xpert/App 执行链 | 稳定主路径 | 必须有针对性测试、全链路回归和兼容说明。 |
| 已发布 Xpert/Toolset/Knowledge 版本 | 不可变线上快照 | 只创建新版本或显式回滚，不得原地修改。 |

### 2.5 依赖和验证真实性

- 优先使用现有依赖；新增依赖必须说明必要性、固定版本、许可证、镜像体积与安全影响。
- 不得通过关闭类型检查、跳过校验、删除测试或降低断言来制造“通过”。
- 验证结果只允许使用：`通过`、`失败`、`未运行`、`不适用`。
- “通过”必须附实际执行命令和可核对摘要；不得把预计结果写成已运行结果。
- 修改后必须检查 `git diff`、`git diff --check`、暂存文件清单和敏感信息扫描结果。
- 本治理 PR 包含既有 `.github/workflows/multimodal-readiness.yml` 和新增 `.github/workflows/quality.yml`；后者执行前端 typecheck、单元测试与构建、后端测试和 Compose 配置检查。`main` 的 required rules 尚未开启，因此工作流结果是自动化证据，不是强制合并门；只有实际 GitHub run 成功后才能表述为“CI 已通过”。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PinkElf-Elysia/ModelMirror](https://github.com/PinkElf-Elysia/ModelMirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
