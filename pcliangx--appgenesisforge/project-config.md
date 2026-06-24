---
trigger: always_on
description: > **Template Version: v6.8.0**（min Claude Code v2.1.154）。逐版本变更见 [CHANGELOG](CHANGELOG.md)。
---

# AppGenesisForge

> **Template Version: v6.8.0**（min Claude Code v2.1.154）。逐版本变更见 [CHANGELOG](CHANGELOG.md)。

## Project Overview

AI 应用开发项目，前端 Web、后端服务、AI Agent、Apple 原生（macOS / iOS）多方向并行。

**当前仓库形态**：AI 团队协作模板（脚手架）——`.claude/` 团队配置与 `docs/` 规范就绪，Tech Stack 已落地（见 ADR-000）；尚无应用代码，等待执行层在首个 feature 启动时创建 `backend/` 与 `frontend/`（Apple 轨为 `apple/`，按 ADR-007）。

## Tech Stack

技术栈**详尽决策**（备选、理由、版本查证）唯一来源是 [`docs/adr/000-system-architecture.md`](docs/adr/000-system-architecture.md)。本文件 `## Tech Stack` 仅放当前生效版本号摘要 + ADR 链接，不重复决策理由。

**当前摘要**：React + Vite、FastAPI + PostgreSQL、uv（Python 包管理 + `uv.lock` 依赖锁 + `.python-version` 解释器 pin）、SQLAlchemy + Alembic、orval（前后端 OpenAPI 契约同步，见 ADR-006）、DeepSeek/Doubao/Qwen/MiniMax 多 LLM、docker-compose 仅编排 Postgres；本地开发一键启动见根 `Makefile`（`make dev` / `make help`；UAT 仍走 deploy-engineer 的 `/agf-deploy-uat`）。详见 ADR-000。**Apple 原生轨**：Swift 6.2 + SwiftUI（[ADR-007](docs/adr/007-apple-native-stack.md)）、swift-openapi-generator（Apple 契约同步，[ADR-008](docs/adr/008-apple-backend-contract-sync.md)）、fastlane + notarytool 四渠道发布（[ADR-009](docs/adr/009-apple-release-pipeline.md)）。

**任何技术栈调整必须新开 ADR**（由 `tech-lead` 撰写）+ 同步更新本段摘要的版本号 / 链接。

## Project-Specific Rules

- 本文件只放项目特有规则，团队通用规则统一放在 `.claude/standards/`，结构性指引在 `.claude/rules/`。
- 项目涉及外部合规、部署约束、第三方集成限制时，在此处追加。
- 设计交付物路径约定：项目级 `docs/design/DESIGN.md`（**设计 token SSOT**，维护者 uiux-designer，各 feature 只引用其 token，禁内联重声明色板/间距）+ 每 feature `docs/design/[feature]/spec.md`（设计规范）+ `docs/design/[feature]/index.html`（自包含静态原型，Tailwind CDN 或内联样式，可直接 `open` 打开），资源放 `assets/`。设计 token 纪律见 `.claude/standards/coding.md` 设计 token 纪律，reviewer 审查项见 `.claude/agents/code-reviewer.md` 设计 token 审查项。
- 修改 `.claude/agents/*.md` 的职责或产出时，必须同步检查 `docs/team-capability-map.md`（"主要输出"列）与 `.claude/standards/team-roles.md`（仅当工具/skill/推荐 mode 变更时）。
- 前后端对接（前端调后端 API；防"按钮没事件 / 契约对不上"两类下游高频缺陷）→ 契约纪律见 `.claude/standards/coding.md` 前后端契约纪律 + [ADR-006](docs/adr/006-frontend-backend-contract-sync.md)（OpenAPI=SSOT，前端走 orval 生成，禁手写 fetch/类型/mock）；测试侧强制覆盖（含交互完整性 + E2E 控件遍历）见 `.claude/standards/testing.md` 前后端对接强制覆盖项。
- 涉及多 LLM SDK 接入（DeepSeek/Doubao/Qwen/MiniMax 切换、fallback、env 变量）→ 必须先看 skill `.claude/skills/agf-wiring-multi-llm-sdk/`。
- **Apple 原生轨（macOS / iOS）**：平台 target 是 task 参数（`macos` / `ios` / `universal`，派工必声明，见 `.claude/standards/apple-native.md` §2）；Apple 客户端契约纪律（swift-openapi-generator 生成 client，禁手写 URLSession / DTO / mock）见 `coding.md` Apple 契约纪律 + ADR-008；Apple SIT 自跑走 skill `agf-running-apple-sit`；app 内接 LLM 先看 skill `agf-wiring-apple-llm`（密钥永不进客户端）。
- Apple feature 合并到 main 后（apple code review 含 SIT Audit 通过 + PL 合并），product-lead 必须**提示用户**是否构建签名分发包；确认后派 `apple-release-engineer` 按 skill `agf-releasing-apple` 构建（fastlane match 签名 + 公证 + 渠道 lane）并冒烟自检，二元 gate（`✅ 构建成功（冒烟通过）` / `❌ 构建失败`）；通过后 apple-qa-engineer 对该分发包跑 E2E / UAT。渠道 → lane 映射权威源见 `.claude/standards/deployment.md` §7；手动触发 `/agf-apple-release`。apple-release-engineer Pool=1（禁 pool，唯一签名身份）。
- SIT 测试由 execution-layer dev 自跑（不是独立 phase）→ 必须按 skill `.claude/skills/agf-running-sit-tests/` 流程；证据落到 `progress/<role>.md` 的 `**SIT 证据**` 段，由 `code-reviewer` 在 code review 时 audit（参见 `.claude/agents/code-reviewer.md` `## SIT Audit` 节）。dev 报告前 + reviewer audit **step 0** 都先跑 `bash .claude/scripts/agf-sit-precheck.sh progress/<role>.md` 机筛证据（placeholder / 漏证据 / 标记矛盾，advisory 不阻断，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 2）。
- 写 E2E / UAT 报告 → skill `.claude/skills/agf-writing-qa-report/`（SIT 不在此 skill 范围，证据走 dev 自跑流程）。
- **UAT 执行前必有用例文档 + 用户审核**（MAJOR / MINOR 强制）：qa-engineer 先按模板 `docs/qa/uat-cases-_TEMPLATE.md` 生成 `docs/qa/<feature>-uat-cases-<date>.md`（每条 AC ≥1 用例、6 字段 + AC 覆盖矩阵 + 界面渲染核查矩阵），**用户审核确认（frontmatter `status: Approved`）后才开测**（用例可在 **dev 实现期并行起草**、审批移出关键路径，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 1）；证据执行时回填用例文档（SSOT），UAT 报告只引用用例 ID；**UAT 每个用户可见界面必须 chrome-devtools 真渲染 + 截图 + 读图四查（导航 / 裁切 / 控件可点 / 视觉达标——截图必须用 Read 视觉读回、对照 design spec 核是否达到可交付用户标准），纯 API 断言不构成含界面用例的 Pass**。PATCH 级 hotfix 可由 product-lead 显式豁免。细则见 `.claude/standards/testing.md`「UAT 用例文档」+「UAT 界面渲染核查」节。
- **交付 lane（full / fast，[ADR-011](docs/adr/011-delivery-pipeline-efficiency.md) 决策 3）**：PL 派单按规模 + 风险**显式选**——**full**（默认；Medium/Large、MINOR/MAJOR、任何高风险）走全套尾部门；**fast**（仅 Small + PATCH + 非高风险，显式选 + 记录风险接受）**只减不跳**（仍部署 + 冒烟 + P0 pass² + 受影响界面渲染核查，E2E 缩到改动面目标 AC）。高风险（auth / schema migration / LLM 切换 / cross-cutting）一律 full、不得 fast。SSOT 见 `.claude/standards/workflow.md` §交付 lane。
- 写 PRD → skill `agf-writing-prd`；写 ADR → skill `agf-writing-adr`。
- 程序化生成中文 docx 报告（决议书 / 评审 / 投标书等高密度文档）→ skill `agf-writing-docx-reports`（docx-js）；程序化生成中文 pptx（制度 / 党政 / 宣贯 deck）→ skill `agf-writing-pptx-reports`（python-pptx）。两者依赖 `.claude/skills/docx/` 与 `.claude/skills/pptx/`（Anthropic 第三方低层 skill，附带 `scripts/office/soffice.py` 做 PDF 预览闭环）。
- 在仓库提 GitHub issue（手工创建 / 报 bug / dev 在 SIT 中发现 P0/P1 自动 path / qa-engineer 在 E2E/UAT 中发现 P0/P1 自动 path）→ skill `agf-writing-github-issue`（含标签锁定 + 最小输入模式）。
- **Multi-instance Worker Pool**（dev / reviewer / qa 同 type ≥ 2 task 自动 fan-out）→ 见 [ADR-001](docs/adr/001-multi-instance-worker-pool.md) + `workflow.md` §Multi-instance Worker Pool；实例命名 `<type>-<N>`，pool 上限按 `team-roles.md` `Pool 上限` 列；PL 用 `bash .claude/scripts/agf-matrix.sh --type=progress|review|qa` fan-in 决策。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pcliangx/AppGenesisForge](https://github.com/pcliangx/AppGenesisForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
