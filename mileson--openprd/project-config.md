---
trigger: always_on
description: <!-- OPENPRD:AGENTS:START -->
---

<!-- OPENPRD:AGENTS:START -->
## OpenPrd Harness

本项目由 OpenPrd 管理。Agent 应优先遵循 repo-local skills 和 hooks；`AGENTS.md` 只保留轻量入口合同。

### Scope

- skill 路由放在 `openprd-router`，命令清单放在 command catalog，强约束放在 hooks。
- `AGENTS.md` 只说明入口、默认行为和高风险门禁，不再承载静态长清单。

### Entry Points

- 先读 `skills/openprd-router/SKILL.md`；在生成的 Codex / Claude 环境里，优先读同名 `openprd-router` skill。
- 需要具体命令时，优先读 `.openprd/harness/command-catalog.md`，不要继续把命令清单膨胀回 `AGENTS.md`。
- `$openprd-shared`：共用语言、文档影响、敏感信息、浏览器安全、小程序验证、产品文案与 i18n 规则。
- `$openprd-requirement-intake`：需求入口分流、用户可见需求类型与内部 L0/L1/L2 路由码对照、PRD 场景视角选择，以及创业验证闭环。
- `$openprd-test-strategy`：测试策略分流、分层验证、任务级 evidence-plan、升级原因与豁免理由。
- `$openprd-harness`：主工作流、`run/loop`、review/change/tasks 与执行节奏。
- `$openprd-frontend-design`：前端设计框架、审美资产库、设计主题/骨架/组件/配方/模板，以及事实、素材、图片和方向前置门。
- `$openprd-benchmark-router`：外部技术、公开 GitHub 仓库、benchmark/对标/最佳实践路由。
- `$openprd-standards` / `$openprd-quality`：`docs/basic/`、就绪验证、EVO 门禁、知识沉淀。
- `$openprd-diagram-review` / `$openprd-discovery-loop`：可视评审与长时间只读挖掘。

### 默认行为

1. 动手前先从 `.openprd/` 重建状态，并先运行 `openprd run . --context`；它是建议上下文，不是自动执行指令。
2. 规划、分析、架构评审、“怎么改”或“会动哪些文件”类请求保持只读；只有用户明确要求实现、继续任务、深度调研、对标复刻或提交时才进入执行。
3. 先分流再执行：`openprd-requirement-intake` 按影响面、未知数、决策成本和验证成本判断需求类型，并保留内部路由码对照：直接处理=L0，现有功能优化=L1，新功能/新流程方案=L2。用户审查默认把路由码并进“需求类型：直接处理（L0）”这类标签里；只有内部排障确实受益时，才额外附“内部路由码”。直接处理类需求可直接处理并事后说明，不打开正式 PRD/review/change/tasks；现有功能优化先在对话内给 mini-plan 再执行，默认不生成正式 PRD/change/tasks；如果用户刚刚已经确认了 L1 mini-plan、范围边界或正式产品边界，后续承接要写成“已确认，我按这个继续”，不要用“确认，我们就按这个……”这类像再次索取确认的句子。只有新功能/新流程方案才先走 requirement intake、对话内 requirement 摘要确认，再 `review/change/tasks`，最后才实现。L2 的 requirement 摘要默认按“需求判断 / 需求理解 / 功能范围 / 技术方案”来写，其中“功能范围”和“技术方案”优先用 Markdown 表格，帮助用户一眼看清；`需求判断` 和 `需求理解` 先用 1 到 2 句轻量主句说清这次是什么、核心问题和第一版目标，边界、风险、异常例子和技术细节下沉到后面的分项或表格，不要把它们都塞进一整段长话里，也不要把某条示例文案写成固定模板。若当前仍在 0 到 1 探索、脑暴或值不值得做的判断，摘要里还要主动补上“验证与创业闭环”：第一批最容易触达的社区或种子用户、你为什么算这个社区里的自己人、当前替代方案和痛点证据、先怎么手工交付、手工作战卡怎么写、能不能先用 spreadsheet / 表单 / no-code 跑起来、如果必须开始做产品也只自动化最重复的一步并先压成 forms / lists / CRUD 骨架、第一版只做哪一件事、能不能压成周末级 MVP、第一批客户路径、从第一个客户开始怎么收费、客户 1 如何打平成本、有没有 10 个样本和更强付费信号、达到什么条件才允许产品化、增长阶段守什么纪律、这条路是否可逆、是否真在解决客户问题、以及是否符合团队价值观、是不是你愿意长期住进去的业务形态。L2 的首轮澄清只能承诺“我先整理需求摘要给你确认”，不能把 requirement 摘要确认、review 和实现压成一句“你回我一句我就开始实现”。如果 `openprd run . --context` 仍然建议 `clarify-user`，当前这轮回复的目标就只能是 `需求摘要` 或 `1 个最高价值澄清点`，不要写成“我先按默认方案实现”。如果用户的下一条回复只是承接上一轮 requirement 摘要的短跟进，而不是提出新范围、改目标或重新发起分析请求，就把它当成对上一轮摘要、默认方向或选项的继续确认，不要重新开一轮泛化 clarify；应直接按当前对话上下文把已确认事实用 canonical capture 路径、`user-confirmed` 来源写回，而不是继续写 `agent-inferred/project-derived` 的用户澄清字段。单纯的“请帮我实现/继续实现”只表示有执行意图，不表示可以跳过 requirement 摘要确认、`capture/classify/synthesize` 写入路径或 review；只有用户明确表示“不需要进行任何确认”时，才允许静默走完整 requirement write path。`review.html` 是稳定评审 artifact，不再默认等于唯一的人类停顿点；默认按 decision-points approval policy 执行，只有当前 lane 仍要求人类决策时才在 final answer 主体里停下请求确认；当 review 已确认且 tasks 已就绪但还需要执行授权时，先给执行确认清单再请用户确认。
4. change/tasks 就绪后，用 `openprd-test-strategy` 按风险选择单元、集成、端到端、人工、视觉、小程序、性能或安全验证组合，并在任务或报告中保留 evidence-plan；同时根据任务边界记录 execution strategy：小范围修正保持 `serial`，中等规模 L1/L2 可推荐 `parallel-workers`，高风险或大规模实现再升级到 `parallel-workers-isolated`；70/20/10 只作健康形状参考，不作硬门禁。
5. 纯图片、封面图、配图、海报、插画、图标、贴纸、mockup 或“先看样子”请求默认直接使用 `imagegen`，也就是 Codex 原生 Image 2；Image 2 是工具路径，不是审美豁免，生成前先写清用途、受众、气质、约束和记忆点，并用 anti-slop 避免默认紫白/蓝紫渐变、通用字体、白底卡片堆叠和无语境装饰；其中 logo、icon、avatar、badge 等开发素材在用户未明确要求场景化展示时，默认按独立素材输出（standalone asset）生成：全画布单主体，不额外添加卡片、设备框或其他展示容器；只有实际发生 `imagegen` 调用后，才能汇报生图结果、失败或限流。生图结果先当候选效果图，不要默认登记到 `.openprd/harness/visual-reviews/`。Agent 要主动确认是否符合预期、是否纳入后续效果图/实现截图对比、以及是否按此继续实现；只有确认后，才把选定方向、整张图或其中子图整理成 reference-set 并进入实现。进入实现阶段时，已有确认参考图用 `openprd visual-compare --reference/--actual`；如果参考图是一张整板、网格图或多对象组合，先运行 `openprd visual-prepare --reference <效果图> --grid <列>x<行>` 或 `--boxes <plan.json>`，确认 contact sheet 后再逐项对比；没有参考图时先判断新建界面还是修改既有界面，新建界面先按用户目标、信息架构变化、视觉决策成本和验证风险完成 3 方向方案评审，修改既有界面用 `openprd visual-compare --before/--after`；局部细节重点则补 `openprd visual-compare --board <focus-board.json>`，多方向实验则补 `openprd visual-compare --board <parallel-board.json>`；普通截图、Computer/Browser/Playwright 实测截图要作为证据时补 `openprd visual-compare --board <verification-board.json>`；同构列表、卡片、网格、表格或用户反馈没对齐时补 `openprd visual-compare --board <alignment-board.json>`，并同时覆盖容器轨道与内部内容槽位轨道；单个 logo、icon、avatar、badge、按钮图形或图片内部居中/视觉重心/偏心问题补 `openprd visual-compare --board <centering-board.json>`，并同时覆盖画布中心、主体外接框中心和视觉重心偏移。visual evidence 同时检查气质、层级、字体/色彩/表面角色和记忆点，不只检查截图是否存在。用户后续如果说“跟效果图”“不一致”“好丑”“复刻”，不能只口头说对比过了，至少先产出一份视觉证据图。如果用户目标是把工作转成可学习、可复用、可回看、可教学或可沉淀的材料，先按期望产物是否需要章节结构、证据锚点、图文讲解、检索练习、工作示例或长期阅读体验来判断；需要时优先走 `openprd learn .` 生成学习包和阅读器，不要用关键词表触发；“仙侠风格的学习材料”这类短请求也按学习型交付物处理，风格只作为 `--genre` 题材参数。
5a. 卡片宽度、间距、留白、对齐、颜色、圆角、字号、按钮或图标等轻量 UI 可视优化，仍可按 L0/L1 小范围修正推进，不自动升级成大界面 3 方向方案评审；但它是用户可见变化，动手前要有一句审美意图和记忆点，收口必须补 `visual-compare` 修改前后图、局部焦点证据板、截图实测证据板、对齐辅助线证据板或内部居中证据板，并检查气质、层级、颜色、字号、间距和表面角色是否成立。只要界面里有同构列表、卡片、网格或表格，就把容器轨道以及标题、副标题、描述、标签、状态、价格、按钮、图标、操作区等相同文案类型/相同组件槽位的对齐当作默认验收项，不等用户先投诉；只量外框、列宽或行顶不算完整对齐验收。只要任务在判断单个素材/图标/头像/徽标/按钮图形的内部居中、偏心或视觉重心，就把 centering-board 当作默认验收项；单张原始截图或主观“看起来居中”不算完整居中验收。build、package、`openprd dev-check` 和单张原始截图不能替代视觉证据。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mileson/openprd](https://github.com/mileson/openprd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
