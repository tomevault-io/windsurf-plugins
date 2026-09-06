---
trigger: always_on
description: 本仓库是团队大语言模型技能库。技能统一放在标准目录 `skills/` 下，`skills/` 的一级子目录代表团队职责域。
---

# Repository Guidelines

## 项目结构与模块组织

本仓库是团队大语言模型技能库。技能统一放在标准目录 `skills/` 下，`skills/` 的一级子目录代表团队职责域。

- `skills/product/`：产品定义职责，包括产品概念白皮书、需求细化、规格评审和 PRD 固化。
- `skills/product/team-concept-whitepaper/`：用于在产品规划阶段定义产品机会、定位、价值、能力边界和演进方向，并编写产品概念白皮书。
- `skills/product/team-spec-refine/`：用于与用户反复确认并打磨规格。
- `skills/product/team-spec-review/`：用于评审规格风险和 ready 状态。
- `skills/product/team-spec-to-prd/`：用于把 ready 的规格固化成 PRD。
- `skills/product/team-spec-archive/`：用于把已完成、废弃或暂停的 active 需求产物归档，避免新需求误改旧规格。
- `skills/config/`：跨流程运行时配置职责，包括 `team-spec/config.yml` 的初始化、校验和增量补全。
- `skills/config/team-config-init/`：用于集中创建、校验和安全补全语言、版本控制、访问策略及写作风格入口配置。
- `skills/codebase/`：代码库理解与说明职责，包括第三方代码库接手、源码走读、能力简报和自有项目 README 编写。
- `skills/codebase/team-codebase-onboarding/`：用于从第三方或陌生代码库提取可追溯的功能清单、架构说明和 AI 接手上下文。
- `skills/codebase/team-codebase-walk/`：用于基于 onboarding 产物和源码进行功能走读、问答和证据追踪。
- `skills/codebase/team-codebase-brief/`：用于把代码库事实转化为面向业务、产品和管理者的能力说明与影响分析。
- `skills/codebase/team-codebase-readme/`：用于为团队自行开发和维护的项目创建、审阅和优化 `README.md`。
- `skills/delivery/`：交付执行职责，包括 PRD 评审简报、Task 拆解、实现、验证和 Spec 级远端交付。
- `skills/delivery/team-prd-to-brief/`：用于把 AI 结构化 PRD 转换为需求、研发和项目管理可评审的演示文稿式简报。
- `skills/delivery/team-prd-to-tasks/`：用于把 PRD 拆解成可独立实现、验证并提交的工程 Task。
- `skills/delivery/team-task-batch-implement/`：用于在同一 Spec 分支按依赖顺序批量实现和验证多个 Task，并在用户逐个检查差异、确认后提交。
- `skills/delivery/team-task-implement/`：用于按行为测试和 TDD 循环实现单个 Task，验证后等待用户检查差异并确认，再形成一个本地 commit。
- `skills/delivery/team-task-verify/`：用于验证单个 Task 实现是否满足验收标准、PRD 和 commit 边界。
- `skills/delivery/team-spec-create-issue-github/`：用于把完整 Spec 创建或同步为一个 GitHub Issue，Tasks 作为 checklist。
- `skills/delivery/team-spec-create-issue-gitlab/`：用于把完整 Spec 创建或同步为一个 GitLab Issue，Tasks 作为 checklist。
- `skills/delivery/team-spec-create-pr-github/`：用于推送 Spec 共享分支并为全部 Task commits 创建一个 GitHub Pull Request。
- `skills/delivery/team-spec-create-mr-gitlab/`：用于推送 Spec 共享分支并为全部 Task commits 创建一个 GitLab Merge Request。
- `skills/tech-debt/`：技术债治理职责，包括技术债分析、细化、评审和工程拆解。
- `skills/tech-debt/team-tech-debt-analyze/`：用于对项目或模块进行只读技术债分析，输出证据化债务候选清单。
- `skills/tech-debt/team-tech-debt-refine/`：用于把模糊技术债诉求细化为可评审规格。
- `skills/tech-debt/team-tech-debt-review/`：用于评审技术债风险、优先级和可执行性。
- `skills/tech-debt/team-tech-debt-to-tasks/`：用于把已评审技术债拆解为工程 Task。
- `skills/harness/`：从已归档 spec 中提炼可复用规则和决策模式，独立于主线交付流程。
- `skills/harness/team-archive-distill/`：用于从 `team-spec/archive/` 下已归档的 spec 中提取决策模式和工程惯例，高度抽象为规则后写入 `AGENTS.md`。
- `skills/writing/`：跨产品、代码库、交付和技术债流程复用的写作职责，包括公共语言风格和代码注释规范。
- `skills/writing/team-writing-style/`：用于建立和维护目标项目的统一写作风格，通过 `team-spec/config.yml` 为其他技能提供单一公共规则入口。

每个技能目录必须包含 `SKILL.md`。只有当辅助文件被 `SKILL.md` 明确引用时才添加，例如 `CONTEXT-FORMAT.md`、`DECISION-FORMAT.md`。

`team-spec/config.yml` 的结构、初始化、校验和增量补全统一由 `team-config-init` 负责。其他技能只读取配置并声明当前操作所需字段；配置文件不存在或缺少必需字段时，先使用 `team-config-init`，不得在业务技能中复制完整配置模式或自行回写配置。纯对话、只读分析和不依赖稳定配置的预览不因配置缺失而阻塞。

如果技能需要稳定执行 API 调用、文件解析、批量发布、幂等检查、拓扑排序或其他容易因大模型临时生成代码而出错的操作，应在技能目录下新增 `scripts/` 目录沉淀固定脚本。脚本必须由 `SKILL.md` 明确引用，且路径按相对 `SKILL.md` 的形式书写，例如 `./scripts/publish_github_issues.py`，不要在技能说明中硬编码本仓库源码路径。

## Team Spec 工作空间

`team-spec/` 是技能安装到业务项目后的运行时工作空间，不是本技能库需要提交的业务产物。不要在本仓库沉淀真实需求、PRD、风险报告或工程 Task。

技能运行时，所有产物应统一写入目标项目根目录下的 `team-spec/`。`team-spec/active/` 是所有尚未归档需求的集合，不再表示唯一活跃需求；单个需求工作区必须放在 `team-spec/active/{slug}/`。`team-spec/archive/` 保存已完成、废弃或暂停的历史需求。

- `team-spec/CONTEXT.md`：跨多个需求复用的全局产品语境，包括规范术语、角色、通用流程和通用业务规则。
- `team-spec/STYLE.md`：可选的项目级公共写作风格，约束文档、用户可见说明和代码注释；路径由 `team-spec/config.yml` 的 `writing_style.guide` 指定。
- `team-spec/decisions/`：跨多个需求长期有效的产品决策记录，仅在决策影响后续多个需求且反悔成本较高时创建。
- `team-spec/active/{slug}/spec/`：单个需求的规格阶段产物，包括可选的机器人场景发现文档 `discovery.md`，以及 `CONTEXT.md`、`decisions/`、`refine.md`、`reviews.md`。
- `team-spec/active/{slug}/concept/`：单个产品或产品体系的概念阶段产物，默认白皮书为 `whitepaper.md`，作为规格细化和后续 PRD 的上游输入。
- `team-spec/active/{slug}/prd/`：单个需求的 PRD 固化产物，是需求到工程的正式交接边界，包括 `prd.md` 与可选 `brief.md`。
- `team-spec/active/{slug}/tasks/`：单个需求 PRD 或技术债规格拆解后的工程 Task。
- `team-spec/active/{slug}/DELIVERY.md`：可选的 Spec 级交付记录，包括共享分支、远端 Issue、Task/commit 映射和 PR/MR；不得加入产品代码 commit。
- `team-spec/active/{slug}/design/`：单个需求的可选功能设计说明书，默认文件为 `functional-design.md`。当前技能库不提供通用功能设计生成技能；该文件由用户或团队另行提供时，下游 Task 拆解、实现和验证必须读取，但不得因文件不存在而阻塞已 ready 的 PRD。
- `team-spec/active/{slug}/STATUS.md`：可选状态文件，只记录整个工作区的生命周期状态，不记录阶段评审结果或单个 Task 的交付状态。
- `team-spec/archive/{slug}/`：单个历史需求的归档目录，包括 `spec/`、`prd/`、`tasks/`、`design/`、`DELIVERY.md`、`STATUS.md` 和 `ARCHIVE.md`。

### 状态合同

所有机器可读状态使用小写 kebab-case。状态按写入对象分为三类，不得混用：

1. 工作区生命周期状态：写入 `team-spec/active/{slug}/STATUS.md`。产品需求链路使用 `concept-drafting`、`concept-review`、`concept-ready`、`refining`、`spec-ready`、`prd-ready`、`implementing`、`paused`、`blocked`；技术债链路使用 `debt-analyzed`、`debt-refining`、`debt-ready`、`implementing`、`paused`、`blocked`。
2. 阶段评审结果：写入 `team-spec/active/{slug}/spec/reviews.md` 或其他阶段报告，不写入工作区 `STATUS.md`。统一使用 `ready`、`needs-refinement`、`blocked`。
3. Task 状态：写入对应 Task 文件，不写入工作区 `STATUS.md`。统一使用 `draft`、`implementing`、`needs-changes`、`blocked`、`verified`、`committed`。`pr-created`、`mr-created` 不属于 Task 状态；Spec 级远端信息写入 `DELIVERY.md`。

同一个 `blocked` 可以出现在不同对象中，但只表示该对象被阻塞；读取方必须结合文件位置判断是工作区、阶段评审还是 Task 被阻塞。用户可见回复可以使用自然语言，写入文件的状态值必须使用上述机器值。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coolbeevip/team-ai-skills](https://github.com/coolbeevip/team-ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
