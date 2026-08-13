---
trigger: always_on
description: 本文件是给 AI 编码代理的项目级开发规范。保持简短、可执行、少歧义；细节优先从 `docs/`、`docs-internal/` 和现有代码中渐进读取。内部文档的分层与索引见 `docs-internal/README.md`；人类贡献者入口为 `CONTRIBUTING.md`。
---

# AGENTS.md

本文件是给 AI 编码代理的项目级开发规范。保持简短、可执行、少歧义；细节优先从 `docs/`、`docs-internal/` 和现有代码中渐进读取。内部文档的分层与索引见 `docs-internal/README.md`；人类贡献者入口为 `CONTRIBUTING.md`。

## 0. 开工前必读

按需阅读，但开始实现前至少确认这些文件：

1. `README.md`
2. `TODO.md`
3. `docs-internal/README.md`（内部文档分层与索引）
4. `docs-internal/01-产品与一体化方案.md`
5. `docs-internal/07-代码健康检查SOP.md`

## 1. Hard MUST

- 先读现有代码和文档，再修改。
- 不主动执行 `git commit`、`git push`、创建/切换分支等 Git 操作，除非用户明确要求。
- 对一次可完成的小任务采用“一个目标一轮推进”的节奏：每完成一个可独立验收的事项（如一次功能点、文档修订、定位与修复闭环），要形成可追溯记录并与该事项绑定。
- 编写新功能或有逻辑改动时，必须同步更新 `docs/` 文档站内容；仅涉及旧文档归档时更新 `docs-internal/`。影响计划、验收或状态时也必须更新 `TODO.md`。
- 当问题根因来自职责堆积、抽象缺失、旧模型残留或重复逻辑时，优先通过小范围重构消除根因；不要为了“最小改动”继续堆临时 patch 或特殊 case。
- 完成实现后按改动规模选择验证：小功能改动只做针对性检查（相关 Go 包测试、TypeScript 类型检查或局部 smoke），不强制全量 lint/build/浏览器验收。
- 当一次改动满足任一条件时，必须执行完整验证并优先用浏览器验收前端交互：修改文件数超过 8 个、同时跨 3 个及以上业务域、涉及认证/权限/Secret/SSRF/数据库迁移/构建部署运行时、或用户明确要求验收。验收通过后再把 `TODO.md` 对应项标记完成。
- **MUST 端到端调用链一致性**：新增功能或修改既有行为时，必须逐层审计前端、API 后端、Worker、Agent 四个边界并明确适用或不适用；凡实际参与调用链的层都必须在同一事项中同步完成。请求/响应 Schema、OpenAPI、前端类型与 API Client、Agent 工具 Schema、异步任务载荷、事件/SSE 协议、权限与审计、错误码、幂等语义及可观测字段必须保持一致，禁止只修改某一层后依赖运行时容错、宽松解析或人工约定维持兼容。验证至少覆盖一条从真实入口到最终副作用或权威回读的成功链路，以及涉及层之间的契约测试；存在异步、失败或取消路径时还必须覆盖对应终态。
- **MUST i18n**：前端任何用户可见文本常量必须走 `i18next/react-i18next`，不可硬编码。包括标题、描述、按钮、菜单、表单 label、hint、placeholder、toast、错误/空状态、确认弹窗、aria-label、schema 校验文案和状态 badge。产品名、文件名、API enum 原始值、URL/slug 示例可以保留为数据或示例；只要作为 UI 文案展示，就必须用 i18n label。
- **MUST i18n 边界**：能在前端本地化的内容必须由前端按稳定 `code`、枚举值或状态 key 映射 i18n 文案；后端只返回稳定 key、原始枚举和必要的原始 message/remark 备注，不返回面向用户的本地化文案。日志正文、第三方原始文本和用户输入内容作为数据展示时例外，但不能冒充 UI 文案。
- **MUST 品牌命名边界**：用户可见品牌统一使用 `Luna DevOps`；项目自有运行标识统一使用 `luna-devops`、`luna.devops`、`luna-gateway` 或 `luna_devops_`（metrics/代码中需要下划线时）。项目尚未发版，不保留旧品牌技术标识兼容层，也不要把品牌技术标识做成用户可配置项。开发者、仓库、文档站和镜像发布地址仍使用真实可达的 Liteyuki Studio 资源：`github.com/LiteyukiStudio/luna-devops`、`https://luna-devops.liteyuki.org`、`liteyukistudio/devops-*`。
- **MUST 后端适配外部平台**：涉及 GitHub、Gitea、GitLab、Harbor、DockerHub、OIDC、Kubernetes、Traefik、AI Provider 等第三方/外部平台的读取、探测、搜索、状态同步和写操作，必须由后端 provider/service/API 适配、聚合或反代。前端只调用平台后端 API，不允许在前端编排第三方平台 API、暴露底层外部平台能力，或用多个底层代理接口拼出业务流程。
- **MUST 实时状态单一事实源**：当前状态、健康度、实时资源数量、实时指标等有时效要求的数据，必须在请求时从 Kubernetes 或对应外部平台读取。数据库只保存期望配置、资源引用、工作流过程与结果、不可变历史，不得持久化、回写或用 Redis/进程内缓存保存上游当前状态。上游不可达时统一返回 `unavailable` 和稳定 `observationCode`；实时响应必须使用 `Cache-Control: no-store`，前端不得通过长 `staleTime` 延续旧状态。
- **MUST Agent Prompt 中文**：`luna-agent` 中的系统 Prompt、模型任务提示、上下文包裹说明、工具描述和配套 Skill 必须使用中文编写；工具名、参数名、枚举、路由名、协议字段和用户原始输入保持原值。Prompt 仍应要求模型按用户当前语言回复。项目未发版，只保留当前 Prompt 版本，不维护旧 Prompt 前向兼容分支。
- **MUST 全链路可观测**：新增或修改业务功能时必须遵守 `docs-internal/14-可观测插桩与验收标准.md`。每个 HTTP/SSE/WebSocket、数据库、Redis、异步任务、外部 Provider、模型和工具调用都必须处于有效 Trace Context 中；关键状态转换必须输出可关联的结构化日志；可聚合结果必须补充低基数 Metric。禁止只给接口入口建 Span 而把内部操作留作黑盒。
- **MUST Context 传播**：请求、Repository、Secret、审计、外部 Client、任务投递和 Worker 执行必须继续传递现有 `context.Context` 或 W3C `traceparent`/`tracestate`，不得在业务调用链中改用 `context.Background()` 截断父链路。跨服务新增 HTTP、消息队列、SSE 或 WebSocket 通道时，必须同时实现传播与父子关系测试；Trace Context 只携带遥测标识，不得复制 Cookie、Token、请求正文或用户输入。
- **MUST 遥测安全与稳定维度**：Span 名、日志事件名和 Metric label 必须使用稳定模板与有限枚举，不得包含用户输入、URL 查询值、资源名、用户/项目/请求/Trace ID 等高基数内容；Secret、Token、Authorization、Cookie、密码、模型 Prompt、工具敏感参数和进程命令行参数不得进入遥测。
- **MUST 可观测验收**：涉及新业务边界、数据库/Redis、异步任务、外部平台或跨服务通信的改动，测试至少断言父子 Trace 传播、失败 Span 状态、关键日志关联字段及敏感字段不出现；完整验收时使用临时外部 OTel 栈抽样验证一条成功链路、一条失败链路和一条跨服务/异步链路，临时可观测组件不得写入仓库。
- Secret、Token、Registry Credential 不允许明文落业务表；密钥类字段不回显给前端。

## 2. 文档编写规范

- `docs/docs/{zh,en}` 是面向 Luna DevOps 使用者和部署管理员的公开文档；`docs-internal/` 存放内部开发文档（长期规范与方案/记录，分层见 `docs-internal/README.md`）；`AGENTS.md` 与 `CONTRIBUTING.md` 分别是 AI 代理与人类贡献者的约束入口。不同受众的内容不得混写。
- 公开文档以“帮助用户完成一个任务”为目标，先给结论、前置条件和最短可行步骤，再按需链接到参考信息；不要按代码模块、内部架构或研发流程组织内容。
- 使用渐进披露：主流程只说明当前步骤必须知道的内容；高级配置、完整参数、兼容范围和排障细节放到独立参考页，避免在入门页堆叠所有选项。
- 一个页面只承载一个清晰目标。优先采用“用途 → 前置条件 → 操作步骤 → 预期结果 → 常见问题/相关参考”的结构；没有实际内容的章节不要保留。
- 公开文档默认不写 CI/发布门禁、覆盖率统计、内部仓库关系、分支策略、源码协作方式、实现架构、迁移过程、历史决策、内部待办或发版前剩余事项。这些内容应写入 `docs-internal/`、`TODO.md`、代码注释或仓库内部开发规范。
- 不在公开文档中用“下一步”展示团队尚未完成的研发计划。仅当用户完成当前任务后确实需要继续操作时，才提供与用户旅程相关的下一步入口。
- 必须保留会直接影响用户操作和风险判断的信息，包括必要前置条件、权限要求、数据影响、安全警告、兼容范围、失败原因、恢复方式和不可逆限制；精简不得以隐藏风险为代价。
- 描述稳定的产品行为和用户可观察结果，不展开内部实现。命令、字段和配置只解释用户需要提供什么、何时使用及其影响。
- 配置文档先给可运行的最小配置，再把可选项按场景分组；不要让高级调优项阻塞首次安装或首次使用。
- 避免重复维护同一事实。版本明细以 Release 为准，API 契约以 OpenAPI 为准，命令与参数以 CLI 帮助为准；公开文档只保留必要说明和稳定入口。
- 顶级导航保持少而稳定，以用户旅程和任务类别命名；内部开发、历史记录和发布流程不得进入公开导航。
- 中文与英文文档的目录、导航和事实必须同步；翻译可以适应语言习惯，但不得出现一侧独有的重要限制或步骤。
- 文档变更至少检查中英文导航、内部链接和 `pnpm --dir docs build`；修改导航、主要用户旅程或页面结构时，还必须用浏览器验收桌面端页面，必要时补充移动端验收。

## 3. 技术栈

后端：

- Go + Gin + GORM
- PostgreSQL，不使用 SQLite
- Redis + Asynq
- golang-migrate
- Kubernetes/client-go
- OpenAPI

前端：

- Vite + React + TypeScript
- Tailwind CSS + shadcn/ui
- TanStack Query + React Router
- React Hook Form + Zod
- i18next + react-i18next
- Sonner toast
- @antfu/eslint-config
- 包管理器必须使用 pnpm

Python：

- 必须使用 uv，不直接用 pip 管理项目依赖。

## 4. 目录边界

- 仓库是 monorepo。
- Go 后端在仓库根目录。
- 前端在 `web/`。
- 本地数据库依赖放 `docker-compose-dev-db.yaml`，只包含 PostgreSQL 和 Redis；本地可观测组件放 `docker-compose-dev-observability.yaml`。API、Worker、Agent 和 Web 均在宿主机手动启动，不纳入开发 Compose。
- `.env.*` 不提交；`.env.example` 可提交。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiteyukiStudio/luna-devops](https://github.com/LiteyukiStudio/luna-devops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
