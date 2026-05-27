---
trigger: always_on
description: 本文件是 AI 工程代理的项目入口协议，只定义工作方式、资料优先级和不可违反的边界。具体功能设计、UI 规范、API 字段与导出格式不在本文件重复描述，必须到对应资料中读取。
---

# WeRead Skill Desktop - AGENTS.md

本文件是 AI 工程代理的项目入口协议，只定义工作方式、资料优先级和不可违反的边界。具体功能设计、UI 规范、API 字段与导出格式不在本文件重复描述，必须到对应资料中读取。

AI 必须同时参考：

1. `AGENTS.md`：工程约束、执行顺序、验收入口。
2. `docs/current-context.md`：当前阶段、边界、默认阅读入口。
3. `docs/requirements-pool.md`：活跃需求、优先级、状态、下一步建议。
4. `feature_list.json`：当前可执行功能队列、依赖、状态和验收证据索引。
5. `progress.md`：当前会话进展、最近验证结果、下一步动作。
6. `session-handoff.md`：跨会话交接、恢复入口和未完成事项。
7. `mvp-design-doc.md`：稳定产品范围、页面、命令、数据流、导出格式。
8. `ui-style-guide.md`：UI 与交互规范。
9. `design.md`：UI 审计结论、设计 tokens、组件规则、页面统一方案和后续改造执行蓝图。
10. `~/.agents/skills/weread-skills/`：微信读书 Skill 原始 API 文档。
11. `/Users/duoshilin/duosl/forks/html-anything`：智能体报告输出形态与风格拓展参考，尤其是 deck / 小红书 / 卡片 / 海报类 HTML skill。
12. 飞书多维表格「微信读书 SKill 桌面端需求收集表」：外部需求收集入口，按第 9 节流程读取和回写。

---

## 1. 项目目标

构建一个微信读书数据导出与管理桌面工具，而不是微信读书客户端。

核心目标：

- 用户可以配置 API Key。
- 用户可以查看书架。
- 用户可以查看单本书的划线与个人想法/点评。
- 用户可以导出 Markdown（包含用于资料库索引的 Frontmatter）。
- 用户可以查看基础阅读统计。

技术方向：

- Tauri 2 + Rust 后端。
- React + TypeScript + Vite + Tailwind CSS 前端。
- UI 遵循 `ui-style-guide.md`。

---

## 2. 资料优先级

当资料之间冲突时，按以下顺序判断：

1. `~/.agents/skills/weread-skills/*.md`：API 参数、字段含义、分页、单位、统计口径的最终依据。
2. `mvp-design-doc.md`：产品范围、页面结构、命令清单、导出格式的最终依据。
3. `ui-style-guide.md`：UI 与交互的最终依据。
4. `design.md`：UI 实施与收敛方案的执行依据。
5. `/Users/duoshilin/duosl/forks/html-anything`：智能体报告输出形态、版式池、交互和截图友好结构的参考依据；只用于抽象设计原则和产物约束，不覆盖本项目隐私、数据口径、UI 气质和版权边界。
6. `AGENTS.md`：工程行为与协作规则。

如果本文件与以上专门文档出现实现细节冲突，优先遵循专门文档。

`html-anything` 中的模板包含第三方来源和各自 LICENSE。后续拓展 PPT、小红书、海报、卡片、数据报告等风格时，可以参考其 `next/src/lib/templates/skills/*/SKILL.md` 的版式池、比例、交互和输出结构，但不要直接整段复制第三方模板文本或视觉资产；如实质复用某个模板或来源作品，必须保留原始署名和许可证要求。

---

## 3. 工作原则

必须遵循：

- 先读文档，再写代码。
- 先建立清晰架构，再实现具体功能。
- 每个功能只实现 MVP 明确需要的范围。
- 前端、数据获取、导出、系统能力保持职责分离。
- API 行为必须以 `~/.agents/skills/weread-skills/` 为准，不凭字段名猜测。
- UI 相关决策必须以 `ui-style-guide.md` 为准。
- 产品面向普通用户，所有功能、设置、状态、错误和权限说明都必须用用户能理解的语言表达；不要把工程字段、内部模块名、协议名或实现细节直接暴露给用户。

工程原则：

- KISS：优先选择简单直接的实现。
- YAGNI：不为未来功能预留复杂结构。
- DRY：重复逻辑要抽象。
- SOLID：模块职责清晰，避免页面组件承担过多业务逻辑。

禁止：

- 不要扩展成完整微信读书客户端。
- 不要做在线阅读体验。
- 不要做推荐发现、相似书籍、公共书评浏览等非 MVP 功能。
- 不要交付伪功能。
- 不要为了假设中的历史包袱或未来扩展牺牲清晰架构。
- 不要主动提交 git。

---

## 4. 开始任务前

### Startup Workflow

开始任何实现前，必须完成：

1. 阅读 `docs/current-context.md`，确认当前阶段边界。
2. 阅读 `docs/requirements-pool.md`，确认当前需求状态、优先级和完成后应更新的位置。
3. 阅读 `feature_list.json`，确认同一时间只推进一个功能；如果用户没有指定任务，选择依赖已满足、优先级最高的 `not-started` 功能。
4. 阅读 `progress.md`；如果存在未完成工作，再阅读 `session-handoff.md`，按交接记录恢复，不要重新猜测上下文。
5. 如果任务涉及产品范围、页面结构、命令清单、数据流或导出格式，阅读 `mvp-design-doc.md`。
6. 如果任务涉及 UI、页面、组件、视觉、交互或布局，阅读 `ui-style-guide.md` 和 `design.md`，并按其中的 tokens、组件规则、页面统一方案和验收清单执行。
7. 如果任务涉及微信读书 API，阅读 `~/.agents/skills/weread-skills/` 中对应能力文档。
8. 如果任务涉及智能体报告输出形态、PPT 风格、小红书图文、海报、卡片、数据报告或新风格拓展，阅读 `/Users/duoshilin/duosl/forks/html-anything/next/src/lib/templates/skills/` 中对应 skill，提炼成符合本项目 Quiet Reading Ledger 的约束。
9. 只有在需要追溯已完成需求、历史决策或旧验收时，才读取 `docs/archive/completed-requirements.md`；不要把归档作为默认上下文。
10. 明确当前改动的边界：前端 UI、前端数据层、Rust API、导出、配置、系统命令中的哪一类。

One feature at a time：开始实现前，必须把选定功能在 `feature_list.json` 中标为 `in-progress`，同一时间只允许一个功能处于 `in-progress`。必须在 `progress.md` 写明当前目标、改动范围和预期验收入口。用户只是询问、规划或审阅时，不强制切换功能状态。

Stay in scope：实现期间只处理当前功能的必要文件、必要状态和必要文档；发现相邻优化时，先记录到 `docs/requirements-pool.md` 或 `feature_list.json`，不要顺手扩张当前交付范围。

不要把其他文档中的内容复制进本文件；需要细节时直接引用并遵循对应文档。

---

## 5. API 使用规则

本文件不维护 API 字段表。所有 API 参数、分页方式、字段单位和统计口径以 `~/.agents/skills/weread-skills/` 为准。

最低要求：

- 每个 API 实现前必须阅读对应 skill 文档。
- 网关请求必须遵守 skill 文档中的统一调用规范。
- 业务参数必须按 skill 文档要求传递。
- 时间、时长、计数口径不得凭直觉解释。
- 出现 `upgrade_info` 时必须中断当前 API 操作并向前端返回明确错误。

常用文档映射：

- 书架：`shelf.md`
- 笔记、划线、个人想法、笔记本：`notes.md`
- 阅读统计：`readdata.md`
- 搜索：`search.md`
- 书籍信息：`book.md`
- 总调用规范与深度链接：`SKILL.md`

---

## 6. UI 规则

本文件不维护 UI 细节。所有视觉、布局、组件、状态和交互要求都以 `ui-style-guide.md` 为准，具体执行、审计结论、tokens、组件规则和改造顺序以 `design.md` 为准。

如果实现中需要 UI 判断，先查 `ui-style-guide.md` 和 `design.md`，不要在本文件补充新的 UI 规范。

涉及全应用 UI 审计、页面风格统一、组件视觉重构或设计系统收敛时，必须额外使用 UI 相关 Skill（当前优先 `frontend-design` 与 `ui-ux-pro-max`）做分析和验收。必须参考 `design.md` 的视觉方向、tokens、组件规则和实现顺序；稳定规则仍回写 `ui-style-guide.md`，不要让 `AGENTS.md` 承载具体样式规范。

用户可见文案必须降低理解成本：

- 不要在界面中出现 `requiresRawNotesConsent`、`outputShape`、`jobId`、`templateId`、`CLI` 等工程字段或内部概念，除非目标用户确实需要并已有上下文。
- 涉及权限和隐私时，明确说明“会读取什么、为什么需要、是否默认开启、数据保存在哪里”，例如用“允许读取划线原文和个人想法”替代内部字段名。
- 设置项和操作按钮要表达用户目标，而不是实现机制；必要时用一句短说明补充影响和边界。
- 错误信息要告诉用户发生了什么、能否重试、下一步做什么，不要只展示底层异常或接口字段。

---

## 7. Rust 与导出规则

Rust 后端职责：

- 配置读写。
- API Key 管理。
- 微信读书 API 调用。
- 数据解析与错误映射。
- Markdown 导出。
- 打开文件夹和微信读书深度链接。

最低要求：

- 配置、API、导出、命令、状态管理分模块实现。
- 导出格式以 `mvp-design-doc.md` 为准。
- API 解析以 skill 文档字段为准。
- 错误返回要对前端可展示，且语义一致。
- 不要把系统错误伪装成成功结果。
- 文件写入、路径处理、目录创建必须可靠。

---

## 8. 验收入口

### Verification Commands

每个阶段完成后默认执行：

- `./init.sh`

`init.sh` 是当前仓库的统一验收入口，至少包含：

- `npm run frontend:typecheck`
- `npm run frontend:build`
- `cd src-tauri && cargo check`
- `git diff --check`

当前仓库尚未固定自动化单元测试入口；如果新增测试脚本，必须同步加入 `init.sh`。如果某个需求需要更强回归，可以临时运行对应的 test 命令，并把命令与结果写入 Evidence。

如果某条命令无法执行，必须说明原因和剩余风险。

完成后必须把验证命令、结果摘要和剩余风险写回 `progress.md`；如果功能完成，还要同步写入 `feature_list.json` 的 `evidence` 字段和 `session-handoff.md`。

### Definition of Done

需求 done only when：

- 当前功能代码、相关文档和状态文件已经同步。
- `./init.sh` 已通过，或失败/跳过原因和剩余风险已记录。
- `feature_list.json` 已写入最终 `status` 和 Evidence。
- `progress.md` 与 `session-handoff.md` 已更新到可 clean restart 的状态。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Duosl/weread-skill-desktop](https://github.com/Duosl/weread-skill-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
