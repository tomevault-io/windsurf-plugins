---
trigger: always_on
description: 本文件适用于 `decision-lab` 仓库中的全部代码、配置、测试、脚本、数据、文档和交付物。它把已确认的产品计划转换为开发时必须执行的约束，供人类开发者和 AI 开发代理共同遵守。
---

# Ludus / decision-lab 项目开发约束

本文件适用于 `decision-lab` 仓库中的全部代码、配置、测试、脚本、数据、文档和交付物。它把已确认的产品计划转换为开发时必须执行的约束，供人类开发者和 AI 开发代理共同遵守。

## 1. 规则级别与事实来源

- **MUST / MUST NOT**：发布阻断规则，不满足时任务不得标记完成。
- **SHOULD / SHOULD NOT**：默认规则；偏离时必须在变更说明中记录具体理由和影响。
- **MAY**：在产品、架构、安全和已选择的交付档位内可自行选择的实现方式；72 小时只对应 Hackathon Prototype，完整 MVP 使用 108/144 小时或重新估算。
- 权威计划位于 `docs/product-plan/README.md`、活动领域文档 `01` 至 `24`、`26`、合同修复完工审计 `28`、accepted CCR 和 `agent-work-manifest.yaml`。`25/27` 是 superseded 历史审计。本文件是执行索引，不替代领域规格。
- 开始开发前必须先阅读计划 `README.md`、`17-product-design-v2.md`、`18-detailed-development-plan.md`、`19-mcp-data-sources-and-launch-constraints.md`、`20-conversation-led-method-routing.md`、`21-existing-asset-reuse-and-conversion.md`、`22-contract-generation-and-security-plan.md`、`23-multi-agent-capacity-execution-plan.md`、`24-frontend-visual-theme.md`、`26-decision-os-invariants-and-agent-engine-contract.md`、`28-contract-repair-completion-audit-20260721.md`、`docs/contract-changes/CCR-20260721-003.md`、`docs/contract-changes/CCR-20260722-004.md`、`docs/contract-changes/CCR-20260724-005.md`、`docs/contract-changes/CCR-20260724-006.md`、`docs/contract-changes/CCR-20260724-Ways-01.md`、`docs/contract-changes/CCR-20260724-SIM-01.md`（含 ADDENDUM-A1）、`docs/contract-changes/CCR-20260724-ENG-02.md`（含 ADDENDUM-A1）、`docs/contract-changes/CCR-20260724-SIM-02A.md`（含 ADDENDUM-A1）、`docs/contract-changes/CCR-20260725-ANALYSIS-01.md`（含 ADDENDUM-A1）、`docs/contract-changes/CCR-20260725-GUEST-01.md`、`docs/contract-changes/CCR-20260726-MOUNT-01.md`、`docs/contract-changes/CCR-20260726-MOUNT-02.md`（含 ADDENDUM-A1）、`docs/contract-changes/CCR-20260726-READ-01.md`、`agent-work-manifest.yaml`，以及本次变更对应的领域文档。
- 计划文档之间出现冲突时，必须把它视为文档缺陷，停止受影响实现，先统一所有相关合同；不得按文件编号、修改日期或个人理解自行选一个版本。
- 新的产品或架构决定必须先同步到所有受影响的计划文档，再修改代码、schema、API、事件或 UI。禁止通过实现默默改变已锁定合同。字段、状态、API、事件或错误码变化还必须创建并获批 CCR，再重新生成 OpenAPI/TypeScript 合同。
- 用户最新的明确决定可以改变计划，但必须完成上述文档同步后才成为新的开发依据。
- no_automatic_decided_transition、decision_record_append_only、report_requires_qualifying_run 和 responsibility_semantics_enforced_in_code 是发布阻断规则。

## 2. 产品身份、边界与金路径

- 展示品牌 MUST 使用 **Ludus**，中文定位为“企业战略决策沙盒”，标准标语为“Ludus — 预见未来，保障您的事业。”
- 仓库、目录、包和产品自身的代码/配置标识 MUST 使用 `decision-lab`；需要 snake_case 的数据库标识使用 `decision_lab`。环境变量按领域使用计划中已定义的 `MODEL_*`、`EXA_*` 等名称。不得把展示品牌和技术标识混为一个命名空间。
- Ludus 帮助决策人暴露假设、因果路径、风险、可控杠杆和建议翻转条件；它不替用户做决定，也不宣称精确预测未来。
- P0 首个用户是硬科技初创团队中承担最终责任的决策人。P0 不以多人审批、投票或实时协作为主流程。
- P0 唯一预置案例和端到端验收案例是：资金与研发资源有限的球形机器人项目，应优先进入救援市场还是家庭服务市场。
- 登录后的第一屏 MUST 是可工作的日常问答，不得先展示营销落地页、模板选择页或模板卡片墙。
- 用户只选择 `quick`、`focused`、`full` 三档分析深度；系统负责选择和解释方法包，不把方法论选择责任转嫁给用户。
- P0 必须跑通：日常问答 -> 候选档案确认 -> Charter 确认 -> 正式分析 -> 证据与质量门 -> 条件化报告 -> 因果沙盘 -> 分支/比较/非破坏性回滚 -> 最终决定 -> 结构化 Review 保存与读取。
- P0 不实现真实支付、积分账本、社区市场、方法编辑器、任意方法组合、自动生成方法包、跨行业正式深度分析、DecisionEpisode 历史检索投影、自动训练、跨租户学习、SSO、复杂 RBAC、多人实时协作、项目管理、自动外部监控、完全私有部署或任意 MCP 执行。

## 3. 已锁定技术栈

除非先修改权威计划，否则 P0 MUST 使用以下主栈：

- Node.js 22、pnpm、Next.js 15、React 19、TypeScript、Tailwind CSS。
- TanStack Query 管理服务端状态，Lucide 提供图标，`@xyflow/react` 实现因果沙盘，Zod 用于前端边界校验。
- Python 3.12、uv、FastAPI、Pydantic 2、SQLAlchemy 2、Alembic、httpx、OpenAI SDK、PyYAML。不得使用系统默认的其他 Python 版本创建环境或锁文件。
- PostgreSQL 16 作为正式业务、租户、事件和任务数据源。
- 独立 Python Worker 通过 `SELECT ... FOR UPDATE SKIP LOCKED` 领取 `AnalysisRun`。
- SSE 提供分析进度，事件 ID 单调递增并支持 `Last-Event-ID` 恢复。
- OpenAI-compatible `ModelProvider` 和 deterministic fixture provider；默认 `MODEL_PROVIDER=deepseek`、`MODEL_BASE_URL=https://api.deepseek.com`、`MODEL_NAME=deepseek-v4-pro`，Key 与覆盖值由环境注入。
- Exa 默认搜索、Firecrawl 默认抓取、Tavily 搜索备用，全部置于可替换 Provider Adapter 后。
- `StructuredReport` 先渲染 HTML，再由 Playwright 从同一表示生成 PDF。
- pytest/pytest-asyncio、Vitest/Testing Library、Playwright 组成测试栈。
- Docker Compose 是可重复的本地交付路径；线上托管平台保持可替换。

P0 MUST NOT 引入以下平行主路径：

- Prisma、Node Worker 或 SQLite 主业务持久化。
- Redis、Celery、Temporal、通用向量数据库或第二套任务状态机。
- 远程 MCP 协议运行时、任意 MCP URL、stdio/npx、自定义 OAuth、写工具或通用高权限工具集。
- 绑定某一家模型、检索或托管平台的业务实现。

SQLite 只 MAY 用于隔离单元测试或明确标识的离线 fixture，不得成为正式迁移起点或第二套业务合同。

## 4. 目标仓库与模块边界

仓库 SHOULD 保持以下顶层结构：

```text
decision-lab/
├── apps/web/                 # Next.js Web
├── services/api/             # FastAPI、领域服务、Worker 与迁移
├── ways/                     # 可审阅、唯一可编辑的方法源
├── method-packs/             # 安装生成、已发布且不可变的运行时方法包
├── fixtures/spherical-robot/ # 演示输入、外部 fixture 与期望结果
├── scripts/                  # seed、verify、运维脚本
├── e2e/                      # Playwright 金路径
├── compose.yaml
└── AGENTS.md
```

- FastAPI route MUST 保持薄：输入输出由 Pydantic 校验，业务规则进入 service，持久化进入 repository。
- 领域代码只能依赖稳定能力接口，不能直接依赖 Exa、Firecrawl、Tavily 或具体模型 SDK。
- 前端 API 层必须使用 canonical schema；不得为页面临时创造与后端平行的状态枚举或字段含义。
- 复杂 JSON、YAML、HTML 和模型输出 MUST 使用 schema 或结构化解析器，禁止用字符串切片或松散正则解析正式结构化数据。
- 参考资产按 `21-existing-asset-reuse-and-conversion.md` 的逐文件判定复用，不使用笼统的“全部重写”或“全部复制”。Hermes 是 MIT：标为 `Extract & adapt` 的纯函数/小模块 MAY 在保留版权许可、记录来源和补齐 Ludus 测试后抽取适配；状态化或同步胶水必须重写为 async、Workspace-scoped 实现。Open WebUI 0.10.2 因文件级提交许可来源未建立，P0 只 `Reimplement from verified behavior`，不得复制其源码、品牌受限界面或依赖树。不得嵌入 Hermes 的单体循环、CLI、Gateway、通用工具全集，也不得以临时 Markdown/目录作为生产唯一状态。
- `ways/hardtech-market-direction/1.1.0` 是 P0 唯一方法源，当前 `release_candidate` 不可直接执行。安装器必须校验并计算内容哈希，生成 `method-packs/hardtech-market-direction/1.1.0` 的不可变 `published` 副本；Router/Worker 只读取 `method-packs`，不得运行时回读 `ways` 或 `探讨`，也不得手改已发布包。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ludus-AdventureX/Ludus](https://github.com/Ludus-AdventureX/Ludus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
