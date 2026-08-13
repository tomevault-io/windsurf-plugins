---
trigger: always_on
description: 这个 repo 是 Erik 的个人 AI 自动化写作 harness。它用项目级 skills、docs runbooks 和可追踪 Markdown / MDX source，帮助 Codex / Claude Code / Hermes / OpenClaw / Pi 等 agents 完成选题、研究、写作、润色、配图、排版、分发与复盘。
---

# AGENTS.md

这个 repo 是 Erik 的个人 AI 自动化写作 harness。它用项目级 skills、docs runbooks 和可追踪 Markdown / MDX source，帮助 Codex / Claude Code / Hermes / OpenClaw / Pi 等 agents 完成选题、研究、写作、润色、配图、排版、分发与复盘。

本文件只保留高频规则和文档路由。低频细节使用 progressive disclosure：先读 `AGENTS.md`，再按任务读取 `docs/` 中的对应文档。

## Always

- 保留用户 edits；不要 revert 用户改动，除非用户明确要求。
- 不要打印、提交或泄漏 secrets、本地运行态、账号态数据和依赖目录。
- Current events、company/product facts、pricing、laws、fast-moving tech topics 必须查证，并写清具体日期。
- repo 内长期 canonical source 放在 `content/origin/`，格式是 Markdown / MDX。飞书文档、Notion 等可以作为上游写作入口；进入 repo 后要同步或转换成可追踪文本。
- 内容写作、选题构思、文章润色、改稿、标题和风格判断时读取 [SOUL.md](SOUL.md)，对齐 Erik 的作者写作气质、register、anti-style 和审美边界；非写作任务不要默认加载。
- 图片生成优先使用系统 `$imagegen` skill，不要重建项目重复的 `gpt-image-gen`。
- 任何最终发布动作都需要 user final review，除非用户明确授权自动发布。
- 先做 small, practical automation；不要把未跑通的能力写成已可用能力。
- 遇到可复用的新坑点、新技巧、workflow 改进或 skill 缺陷，随手沉淀到项目 docs，或新增、修改 project-level skills。
- 临时想法、未确认价值的 todo、本机上下文先放 `.local-memory/`；验证为可复用后再迁移到项目 docs、`AGENTS.md` 或 `.agents/skills/`。
- 当某项能力已经值得 project-level skill 化时，主动建议并沉淀为 `.agents/skills/*`。
- Python 工具链统一使用 uv（`uv add` / `uv sync` / `uv run` / `uv lock`），禁止 pip / pipx / poetry / 裸 python 直接操作依赖。
- 新建或修改 project skill 时遵守 [docs/skills/skills-guide.md](docs/skills/skills-guide.md) 的规范：PEP 723、路径引用、脚本接口设计、SKILL.md 写作原则。

## Operating Principle

这些规则是为了保护安全、可追踪性、发布边界和长期复用，不是为了限制 agent 的创造力。Agent 应在边界内主动提出更好的选题、结构、表达、视觉方案、工具组合和 workflow 改进；新做法先以小实验跑通并验证，再沉淀为项目 docs、scripts 或 skills。

## Decision Autonomy

- 可逆、低风险的写作、研究、整理、preview、本地验证和小型项目 docs 或 skills 改进，agent 应主动推进。
- 涉及最终发布、外部发送、账号态、付费、secrets、删除或移动用户稿件、改写 canonical source 但意图不清时，先向用户确认。
- 不确定时优先做小实验、保留原始材料、记录假设和验证结果，而不是把探索写成长期规则。

## Memory Layers

- `AGENTS.md`：高频规则、行为边界和 docs router，保持简短。
- `docs/`：长期项目 memory，存放项目技术文档、workflow runbooks、复盘、规范和已验证的可复用经验。
- `.local-memory/`：本机短期临时 memory，不入 Git，不是 canonical source；验证为可复用后再迁移到 `docs/`、`AGENTS.md` 或 `.agents/skills/`。
- `.agents/skills/`：可执行 memory，沉淀可重复执行的项目级能力、脚本、checklist 和 workflow。

## Writing Storage Paths

- `content/inbox/`：原始输入和待整理材料 scratch。
- `content/drafts/`：本地写作草稿 scratch。
- `content/origin/YYYY-MM-DD-topic/`：可追踪 canonical Markdown / MDX 文章源稿（目录名必带日期，不写无日期歧义的 slug）。
- `content/wechat/`、`content/blog/`：从 `content/origin/` 派生的渠道版本。
- `content/origin/YYYY-MM-DD-<slug>/assets/`：article-local assets，跟随 origin article 使用；图片、封面、正文插图等应放此处并由 `index.md` 引用。大体积二进制素材默认不提交（受 `.gitignore` 全局忽略），除非任务明确需要 repo 追踪。
- `content/wechat/YYYY-MM-DD-<slug>/assets/`：微信渠道专用 assets；如果图片已在 origin assets 且体积较大，可用相对路径指回 origin，避免重复提交。
- `content/assets/`：跨文章复用的 prompts、metadata、manifest 和 sources；不要放单篇文章的一次性素材。
- `.local-archive/YYYY-MM-DD-<slug>/`：本机归档快照，`<slug>` 为裸 topic，目录名带日期前缀。写作收尾时由 `writing-task-closeout` 把最终 `index.md`、实际使用的图片、生成 prompts/metadata 统一归档至此，不入 Git。图片生成阶段不要双写，只保留一份在 `assets/` 工作副本中。
- 不要把文章草稿、渠道预览或一次性写作 scratch 放到 `docs/`；不确定时读取 [docs/project/directory-layout.md](docs/project/directory-layout.md)。

## Docs Router

根据任务读取最小必要文档：

| 任务 | 先读 |
| --- | --- |
| 了解项目愿景、写作场景 | [docs/project/vision.md](docs/project/vision.md) |
| 对齐 Erik 作者写作气质、register、anti-style 与审美边界 | [SOUL.md](SOUL.md) |
| 了解完整 AI 写作工作流、skill 分工、目录约定 | [docs/workflows/ai-writing-workflow.md](docs/workflows/ai-writing-workflow.md) |
| 理解 writing-agent-harness 身份和边界 | [docs/reference/writing-agent-harness-profile.md](docs/reference/writing-agent-harness-profile.md) |
| 查看当前建设 todo | [docs/project/todolist.md](docs/project/todolist.md) |
| 新建或整理文章目录（`content/origin/` 必须使用 `YYYY-MM-DD-topic` 格式） | [docs/project/directory-layout.md](docs/project/directory-layout.md) |
| 规划自动化能力 | [docs/project/automation-roadmap.md](docs/project/automation-roadmap.md) |
| 常规写作、研究、润色 | [docs/workflows/writing-overview.md](docs/workflows/writing-overview.md) |
| 开始写文章初稿（draft）前 | [docs/reference/format-standards.md](docs/reference/format-standards.md) 然后 [SOUL.md](SOUL.md) |
| 早期灵感脑暴、确定 writing brief / outline | [docs/workflows/writing-overview.md](docs/workflows/writing-overview.md#ideation-first) |
| 微信公众号排版、草稿箱、发布 | [docs/workflows/wechat-writing-publishing.md](docs/workflows/wechat-writing-publishing.md) |
| 新建或修改 project skill | [docs/skills/skills-guide.md](docs/skills/skills-guide.md) 然后本文件 "Always" 中的 Python 规范 |
| 查看项目 skills 边界 | [docs/skills/skills-list.md](docs/skills/skills-list.md) |
| 沉淀 memory、复盘、skill 自我进化 | [docs/reference/self-evolution.md](docs/reference/self-evolution.md) |
| 使用本机 scratch memory | [docs/reference/local-memory.md](docs/reference/local-memory.md) |
| 图片、封面、正文插图、视频素材/剪辑 | [docs/reference/visuals.md](docs/reference/visuals.md) |
| 微信发布复盘与坑点 | [docs/retrospectives/2026-06-05-wechat-publish.md](docs/retrospectives/2026-06-05-wechat-publish.md) |
| 剪藏网页文章（微信/博客/论文）到 Notion | [`.agents/skills/article-to-notion/SKILL.md`](.agents/skills/article-to-notion/SKILL.md) |
| 直接读写 Notion 页面、上传图片/文件、设 cover/properties | [`.agents/skills/notion-cli/SKILL.md`](.agents/skills/notion-cli/SKILL.md) |
| ntn CLI 踩坑总结（文章剪藏 / Notion 写入前必读） | [docs/retrospectives/2026-06-28-article-to-notion-ntn-cli-refactor.md](docs/retrospectives/2026-06-28-article-to-notion-ntn-cli-refactor.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eriklee1895/writing-agent-harness](https://github.com/eriklee1895/writing-agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
