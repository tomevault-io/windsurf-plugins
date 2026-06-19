---
trigger: always_on
description: 大禹治库 Skill（Dayu Harness Skill）是帮助项目低成本接入 Harness Engineering 理念的一次性部署工具。将以 AGENTS.md 为根的渐进式披露治理体系部署到目标项目。仅通过 /dayu-harness 显式命令激活。
---


# Dayu Harness Skill

## 定位

大禹治库 Skill 是管理和维护项目治理体系的**一次性部署工具**，不是治理体系本身。Skill 目录中的模板、脚本和资产只是部署来源；被部署到目标项目中的 AGENTS.md、docs/ 文档、hooks、CI 与维护脚本，才是 Harness Engineering 治理体系的实际载体。以 AGENTS.md 为根的渐进式披露文档体系是最终权威。初始化完成后，Skill 可安全删除——项目的治理体系已独立运行。

“大禹”取自大禹治水：不把洪流堵在一处，而是疏导、分流并建立长期秩序。被部署到目标项目的治理体系，其设计哲学源自 [Harness Engineering](https://openai.com/zh-Hans-CN/index/harness-engineering/)：工程师不再手写每行代码，而是设计约束环境、明确意图边界、构建反馈回路，让 AI 智能体可靠工作。目标项目内的文档和资产对应 HE 六大概念——AGENTS.md 是「地图而非手册」、docs/ 目录是「仓库即记录系统」、hooks + CI 是「机械化执行」、CLAUDE.md 渐进式路由是「智能体可读性」、archive/ + `docs/harness/maintenance.md` 是「熵管理」、ai-execution.md + ai-memory.md 是「人类掌舵，智能体执行，并把经验沉淀回项目」。

直接把治理规则只做成 Skill，只能让某个 Agent 在当前环境中按规则工作，属于 Agent-centric 约束。大禹治库 Skill 的目标是 Project-centric：把长期规则、项目知识/经验和机械化反馈部署进目标仓库，使它们可版本化、可审查、可迁移，并且不依赖某个 Skill、会话或工具长期存在。

## 激活条件

Skill 仅通过显式命令激活：用户输入 `/dayu-harness`。

Skill 不在日常 AI 协作中自动介入。Skill 删除后，治理体系的维护由 AI 读取项目中的 `docs/harness/maintenance.md` 自行处理。

为兼容 Claude、Codex 和通用 Agent Skills 客户端，canonical `SKILL.md` 不使用工具专属 frontmatter。具体适配策略见 [references/agent-compatibility.md](references/agent-compatibility.md)。

## 边界规则

- Skill 仅在用户显式调用 `/dayu-harness` 时工作
- 项目无 AGENTS.md → 进入脚手架模式
- 项目已有文档体系但不完整 → 进入融合模式
- 项目已有完整体系，用户要求增删改约束 → 进入维护模式
- 项目已有完整体系，用户要求检查完整性 → 进入诊断模式
- 所有操作前先分析项目现状，基于 [Q&A-TEMPLATE.md](Q&A-TEMPLATE.md) 适配提问；默认治理与 Git 能力不作为是否启用的问题，只在已有配置需要合并策略时确认
- 所有面向用户的提问和选项必须中英双语展示，格式为中文在前、英文在后；选项使用 `[1] 中文 / English`，默认项写明 `（默认，推荐）/ (default, recommended)`
- 初始化第一个阻塞问题必须是部署内容语言（中文 / English）。不得先询问项目编程语言、技术栈或项目类型；这些信息由 Skill 根据文件自动判断，空项目默认按 Node.js 治理工具链基线处理。
- 问答交互规则只约束 Skill 运行时，不写入目标项目部署产物。若宿主客户端暴露了可调用的原生交互机制（如选择器、确认对话或计划审批控件），必须优先使用；若当前客户端没有暴露可调用的结构化交互能力，则采用兼容降级：一次只提出一个阻塞问题，立即结束当前回复并等待用户输入，不得在同一轮继续运行命令、不得合并多个阻塞问题、不得假装已经进入交互控件
- 可选的 `github.repository-settings` 在用户选择启用、部署验证通过且本次流程明确追加 `--github-remote apply` 后，才调用 GitHub API 同步 `allow_auto_merge=true` 与 `delete_branch_on_merge=true`；dry-run 只预览，不修改远端。Issue/PR/TDD/发布能力仍以配置、策略文件、工作流和说明为主。
- 部署/融合前统一执行 `scripts/ensure-environment.sh <project-root> --check --capabilities "<resolved capability ids>"`；尚未确定可选能力时不传 `--capabilities`，脚本按默认必选能力检查。若返回 `needs_install`、`needs_initialization` 或 `needs_user_action`，先向用户展示安装/初始化建议并确认；若用户拒绝，当前流程终止
- 任何涉及已有配置的操作，按能力清单中的可管理项/联动组件逐项处理：installer-backed 组件（如 husky hook 片段、`.gitignore`）先走 manifest installer 的 `--check`；静态模板/资产文件组件（如 `commitlint.config.cjs`、ESLint/Prettier/lint-staged 配置文件、`.github/workflows`、ruleset JSON）改用 `scaffold.sh --dry-run` 输出变更预览；若可提供现有文件与目标文件对，优先调用 `diff-helper.sh merge-plan <existing> <incoming>`；否则继续基于 `scaffold.sh --dry-run` 做人工确认
- **不覆盖已有配置**，必须经用户确认

### 结构化提问约束（全流程）

以下阻塞场景必须以单问题块输出，一轮只问一个问题，且不得要求用户手输命令：

- 版本冲突（`package.json`/`package-lock.json`/`VERSION`/`CHANGELOG.md`/`.release-please-manifest.json` 版本不一致）：提示冲突类型并给出 `[1]` 回到 `0.1.0`、`[2]` 采用某个现有版本源、`[3]` 暂停本次流程三类选项。
- GitHub 远端创建：仅在用户选择开启时执行 `gh auth status`、`scripts/github-remote.sh --check/--apply`；若未登录或可见性冲突，先给出 `[1]` 重试登录、`[2]` 暂不创建远端、`[3]` 跳过 GitHub 相关能力。
- hooks/workflows merge：当 `.husky`、`.github/workflows` 已存在且 `manual_required` 时，必须提供 `[1] 保留现有`、`[2]` 合并策略、`[3]` 替换、`[4]` 跳过该项的选择。
- 已跟踪 `.claude` 或 `CLAUDE.md`：若仓库已跟踪本地 Agent 路由目录/文件，必须先确认 `[1]` 保留本地配置、`[2]` 使用 Skill 模板合并后继续、`[3]` 从 Git 索引移除但保留本地文件后继续、`[4]` 跳过该能力。
- 受保护分支：启用 `github.branch-protection` 且检测到既有保护策略时，必须先确认策略继承关系与合并策略（合并/替换/仅保留现有），不允许自动默认覆盖。
- 部署、融合、维护或生成操作完成后，必须执行收尾验证，并按 [docs/completion-report-template.md](docs/completion-report-template.md) 用自然语言向用户汇报结果

## 5 个模式

### 1. 脚手架

触发：项目无 AGENTS.md

1. 若用户未提供部署语言，先通过交互门禁确认部署语言并等待回答；这必须是第一个问题。随后执行 `scripts/ensure-environment.sh <project-root> --check --capabilities "<resolved capability ids>"`，由脚本自动判断技术栈、Git/Node 治理工具链初始化需求和 `.gitignore` 模板（见 Q&A 前置问题；尚未确定可选能力时使用默认必选能力检查）
2. 分析项目现状（读取文件结构、已有配置）
3. 按 [Q&A-TEMPLATE.md](Q&A-TEMPLATE.md) 的交互门禁询问 GitHub、发布、代码工具等可选能力（默认治理与 Git 能力直接纳入部署）
4. 展示确认汇总
5. 用户确认后：调用 `scaffold.sh --dry-run --enable <optional capability ids>` 预览变更 → 对已有配置确认策略 → `scaffold.sh --apply --finalize-git auto --enable <optional capability ids>`（如有冲突再加 `--strategy`，如用户已确认远端同步再加 `--github-remote apply`；启用 GitHub Issue/PR 且需跳过目标仓库 E2E 时必须显式追加 `--github-e2e skip`）复制默认与可选模板文档 + 安装联动的脚本资产 + 始终部署核心维护脚本
6. `scaffold.sh --apply` 必须先完成 `validate/audit/check-consistency/capability-smoke`，通过后再精确 stage managed paths、创建初始化提交，并按远端状态推送默认分支或创建初始化 PR；启用 GitHub Issue/PR 且远端同步成功后，必须创建测试 Issue、测试分支和测试 PR，等待 `issue-lint.yml` 与 `pr-lint.yml` 成功；验证通过后关闭测试 PR、关闭测试 Issue 并删除测试分支，避免目标仓库残留测试产物；随后使用完成报告模板向用户汇报

### 2. 诊断

触发：已有项目，检查完整性

1. 优先执行 `audit.sh --json` 获取结构化诊断报告
2. 将报告中的 `description_nl` 和 `results` 以自然语言呈现给用户
3. 若 audit.sh 不可用，按 `docs/harness/maintenance.md` 诊断清单手动逐项检查
4. 报告缺失/冲突，给出修复建议

### 3. 融合

触发：已有文档体系，需要合并

1. 诊断现有状态（调用 `audit.sh --json`）
2. 对每个已有组件按能力 manifest 中的联动项区分：
   - 有 manifest installer 的组件：调用对应 installer 脚本 `--check` 获取结构化 merge plan
   - 无 installer 的组件（静态模板/资产文件）：先用 `scaffold.sh --dry-run` 获取差异说明；若有源文件和目标文件对可用，再补充 `diff-helper.sh merge-plan <existing> <incoming>` 的结构化说明，再展示给用户
3. 将 merge plan 或差异说明中的 `description_nl` 以自然语言呈现给用户
4. 逐项双语询问：[1] 保留现有 / Keep existing [2] 替换 / Replace [3] 合并 / Merge [4] 跳过 / Skip
5. 用户逐项确认后，按组件类型执行：
   - installer-backed 组件：调用对应 installer `--apply <merge|replace|skip>`
   - 无 installer 组件：按用户确认方案，用 `scaffold.sh --apply` 或手工更新

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kinoward/dayu-harness-skill](https://github.com/kinoward/dayu-harness-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
