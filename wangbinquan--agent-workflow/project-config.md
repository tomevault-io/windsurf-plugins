---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repo is **mid-implementation** (M1 in progress; ~9/18 of M1 done as of last commit).

**Read in this order at session start:**

1. `STATE.md` — **session-to-session execution log**. Always read first; tells you what's done, what's next, current caveats.
2. `design/plan.md` — 81-issue roadmap (M0–M5). Pick next issue from here once you know the state.
3. `design/proposal.md` — product spec (authoritative).
4. `design/design.md` — technical design (authoritative).
5. `proposal/init.md` — original Chinese proposal, preserved for history. When it disagrees with `design/*.md`, `design/*.md` wins.

When a batch of issues completes, commit + push and update `STATE.md` so the next session can pick up seamlessly.

`bun install` then `bun test` to verify dev environment works.

## RFC workflow（新增 / 修改前的强制流程）

任何超出 `design/plan.md` 已列 issue 范围的**新功能、非平凡重构、产品行为变更**，必须先走 RFC，再写代码：

1. **落档**：在 `design/RFC-NNN-{slug}/` 子目录下创建三件套
   - `proposal.md` —— 产品视角：背景、目标 / 非目标、用户故事、验收标准
   - `design.md` —— 技术设计：接口契约、数据流、与现有模块的耦合点、失败模式、测试策略
   - `plan.md` —— 任务分解：编号子任务（`RFC-NNN-T1...`）、依赖、PR 拆分建议、验收清单
2. **编号**：递增分配，从 `RFC-001` 起；在 `design/plan.md` 的 "RFC 索引" 表里登记新条目（标题 + 状态：Draft / In Progress / Done / Superseded）。
3. **用户确认**：RFC 写完后必须用 `ExitPlanMode` 或显式询问得到用户批准，才能进入实现阶段。**不要边写 RFC 边改代码**。
4. **STATE.md 同步**：RFC 落档同时在 `STATE.md` 顶部追加一行"进行中 RFC"指向新目录。RFC 完工后把状态改为 Done 并在 `STATE.md` 已完成 issue 表里加一行（与 P-X-XX 同等级）。
5. **PR**：单个 RFC 默认对应单个 PR，commit message 前缀写明 `feat(scope): RFC-NNN 标题`；如确实需要拆分，在 `plan.md` 里说明并分别立 PR。
6. **不走 RFC 的例外**：拼写 / 单行 bug 修复、纯重命名、依赖升级、文档增删、测试补充、CI 微调。这些可以直接改 + 提交。

新 session 接手 RFC 时也按 `proposal → design → plan` 顺序读，规则与 `design/*.md` 一致。

## Multi-person collaboration（并发改动保留原则）

本仓常有多人并发开发——session 启动时 working tree 里可能已经有他人未提的修改 / 未追踪文件（典型场景：另一个 RFC 正在并行落地）。提交本人工作时必须遵守：

- **绝不删除别人的代码**：包括别人改过的行、新增的文件、`design/plan.md` / `STATE.md` 等共享索引里别人加的条目、`package.json` / lock 文件里别人加的依赖。如果不确定某段改动是不是自己的，宁可保留也不要删。
- **同一文件混了多人改动可以一起 commit**：不要为"剥离他人改动"去手动改回原内容再恢复——那种操作既危险又容易留脏。直接 `git add` 整个文件、在 commit message 里写清自己改动的范围即可，他人的部分作为附带保留。
- **新文件按归属处理**：自己的新文件正常 `git add`；他人留下的未追踪文件**不要主动加进暂存区**，让对方自己提。`git add .` / `git add -A` 这种全量加法在多人 working tree 下慎用，优先按路径精确 `git add`。
- **commit message 只描述自己的改动**：即便文件里包含了别人合并进来的零散行，commit 摘要 / body 也只写本次工作的内容；不要替别人写描述。
- **冲突优先调和**：如发现工作树里他人改动与本次工作有真实冲突（同一函数同一行），停下来先问用户，不要单方面覆盖。

## Test-with-every-change（测试用例随每次需求 / 修复落地）

**任何代码改动落 commit 之前必须带上对应的测试用例**——既包含新功能的正向覆盖，也包含 bug 修复的回归防护。
没有"先实现、之后补测试"这一档；测试用例是改动本身的一部分。

- **新功能**：实现的同时给所有正向 / 边界 / 错误路径写测试。RFC 的 `design.md §测试策略` 列出哪些 case 必写，PR 必须把它们都跑绿才算交付。
- **bug 修复**：先写一个能稳定复现该 bug 的测试用例（红），再写修复（绿）。把"为什么这条测试存在"写进 test 文件顶端的注释（链接 commit / RFC / issue），让未来任何 refactor 一旦把它变红能立刻看出意图。
- **首选可断言面**：抽出纯函数 / 纯数据预言（典型例子见 `affectsDefinition` / `affectsEdgeDefinition` / `selectionSig` / `deriveSelection` / `extractMissingRefs` / `hasConflict`），在用户层面 wire 进去后再写少量集成断言。运行时巨型组件难直接覆盖时，**最低限度也要保留一条源代码层文本断言**作为兜底（例如"`selectionOnDrag` 不得出现在 `WorkflowCanvas.tsx`"）。
- **回归防护命名**：测试文件 / describe 标题应能让人一眼识别它锁的是哪类回归（例如 `canvas-edge-changes.test.ts` 顶部直接写明"locks in EdgeInspector reachability fixes from commit 9b7ba31"）。
- **运行门槛**：`bun run typecheck && bun run test && bun run format:check` 必须全绿才能 push。GitHub Actions 同样会跑这三项 + 单二进制 build smoke + Playwright e2e；按 [feedback_post_commit_ci_check] 推完后立刻查 CI 状态。
- **flaky 不能掩盖红 case**：发现某测试间歇性失败，先确认是不是真 bug；如果确属环境 / 时序，要么修测试（首选 `findByRole` / class 选择器去掉 i18n race），要么显式用注释标记并开 issue，**绝不允许"重跑就过了"作为通过依据**。
- **不写测试的极少数例外**：纯文档 / 注释改动、依赖版本号 bump（且 lock 文件锁住了 minor）、CI 配置微调、prettier 自动 format。**任何触及生产代码或测试代码的改动都没有这个豁免**。

## Frontend UI consistency（前台界面统一风格强制原则）

任何新增/改动的前台界面——新按钮、新弹窗、新表单、新列表行、新页签、新空状态、新页面 header
——必须**优先复用既有公共组件 / 样式 class**，**禁止**为了"快一点"而落原生 HTML 元素 / 自写一套
chrome / 自写一套 CSS。整个系统的视觉与交互风格要保持一致，新功能不能成为视觉孤岛。

**已存在的公共组件**（持续增加；写代码前先在 `packages/frontend/src/components/` 下扫一遍，
不在这里写名字以免清单过时——以源码实际为准）：

- **Dialog** (`components/Dialog.tsx`) — 所有 modal / overlay 必走这一个：自带 overlay + portal +
  focus trap + ESC + outside-click + a11y。提供 `footer` 槽位放 Save / Cancel。**禁止**新写
  `.xxx__overlay` / `.xxx__panel` 之类的 modal chrome。
- **Form primitives** (`components/Form.tsx`) — `<Field>` (label + hint + 必填 \*) /
  `<TextInput>` / `<NumberInput>` / `<TextArea>`（含 `monospace`）/ `<Switch>`。表单字段一律
  走这套，**禁止**直接落 `<input className="form-input">` 或自写 border / focus ring。
- **Select** (`components/Select.tsx`) — RFC-036 自带 popover 的下拉，键盘 / a11y 完整。**禁止**
  在弹窗内直接落原生 `<select>`，原生弹层无法和周围 UI 风格对齐。
- **ChipsInput** (`components/ChipsInput.tsx`) — 标签 / 字符串数组输入：Enter / 逗号 commit +
  Backspace 删除 + dedup + validator。**禁止**自写"chip 输入 + × 删除"逻辑。
- **`.segmented`** (`styles.css`) — 2-N 个短选项的分段控件（同 LanguageSwitch / NodeInspector
  clarify sessionMode）。短列表互斥选择走这条，**禁止**自写 radio 按钮组。
- **页面骨架**：`.page` / `.page__header` / `.page__header--row` / `.page__actions` /
  `.page__section`；行级行动按钮 `.btn .btn--sm` / `.btn--primary` / `.btn--danger` /
  `.btn--xs`；状态 chip 走 `<StatusChip>` / `<TaskStatusChip>` 等既有组件。
- **错误 / 空 / 加载状态**：`<ErrorBanner>` / `<EmptyState>` / `<LoadingState>`，**禁止**写
  `<div className="error-box">…</div>` 自己拼。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangbinquan/agent-workflow](https://github.com/wangbinquan/agent-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
