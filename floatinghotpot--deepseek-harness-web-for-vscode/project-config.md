---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

Vibe Coding 八荣八耻

以臆猜接口为耻，以查档求证为荣。
以模糊开工为耻，以对齐需求为荣。
以脑补业务为耻，以请示规则为荣。
以新增冗余为耻，以复用存量为荣。
以省略校验为耻，以完备测例为荣。
以乱改架构为耻，以恪守规范为荣。
以不懂装懂为耻，以坦诚存疑为荣。
以批量乱改为耻，以分步迭代为荣。

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **项目速览**（2026-08-17）：
> - 产品：**DeepSeek Harness for VS Code** —— VS Code 扩展，一键启动 DeepSeek Harness（DSH）并把其 Web UI 内嵌进 IDE；同时兼容 Antigravity（VS Code fork，走 Open VSX）。
> - 技术栈：**Node.js + TypeScript**（VS Code 扩展 API）；npm 管理；vsce 打包；git 版本控制（仓库名 `deepseek-harness-for-vscode`，publisher `floatinghotpot`）。
> - 架构：传输桥方案（webview 文档直载 DSH 前端 + fetch/WS/剪贴板 shim，经 postMessage 转发到扩展宿主 Node 代发）。详见 `doc/feature/00-dsh-vscode/solution.md`。
> - 当前阶段：MVP 实现中（`doc/feature/00-dsh-vscode/plan.md` T1–T12）。

## 0. Thinking Discipline (MUST READ FIRST)

> "The models make wrong assumptions on your behalf and just run along with them without checking. They don't manage their confusion, don't seek clarifications, don't surface inconsistencies, don't present tradeoffs, don't push back when they should." — Andrej Karpathy

**Before answering any question about the codebase, ask yourself: "Did I read the code, or am I guessing?"** If you haven't read the relevant source file, DO NOT ANSWER. Run grep/read first. Naming conventions, prior experience, and "this is how it usually works" are NOT valid sources.

- **Manage confusion**: When something looks inconsistent or unclear, STOP. Name what's confusing. Ask. Do not silently pick an interpretation and proceed.
- **Push back**: If a simpler approach exists, say so. If the user's request contains scope creep, flag it. If a proposed change has hidden risks, surface them. Do not be a passive executor.
- **Present tradeoffs**: When multiple valid approaches exist, lay out the options before picking one. Let the user decide.

## 1. Communication & Language
- **User Correspondence**: ALWAYS respond to the user in **Chinese**.
- **Documentation**: ALL project-wide documentation (Markdown files) must be written in **Chinese**.
- **Technical Content**: Code identifiers, comments, and Git commit messages must be in **English**.
- **Transparency**: For complex refactoring or destructive actions, describe the plan in `Thought` and obtain approval first.

## 2. Risk, Production Safety & Code Quality
- **Quality First**: Do not rush. If unsure about the quality of the code, ask for clarification.
- **Simplicity First**: Minimum code that solves the problem. No features beyond what was asked. No abstractions for single-use code. No "flexibility" or "configurability" that wasn't requested. If 200 lines could be 50, rewrite it. Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.
- **Surgical Changes**: Touch only what you must. Don't "improve" adjacent code, comments, or formatting. Don't refactor things that aren't broken. Match existing style. If you notice unrelated dead code, mention it — don't delete it. Every changed line should trace directly to the user's request. Remove only imports/variables that YOUR changes made unused.
- **Code Review**: after any code changes, always check for bracket balance and syntax errors. if TypeScript changed, run `npm run compile` (tsc strict) and fix **ALL** issues — including `info` level. if plain JS changed, run `node --check <file>`. The target is zero issues.
- **Defer Requires Proof**: Every deferred issue MUST cite a concrete blocker (unavailable API, cross-module migration plan, Phase 2 feature gate not yet open). Severity (P1/P2), frequency ("low risk"), or effort ("too large") are NOT valid reasons to defer — avoid small issues accumulate into technical debt.
- **Partial Formatting**: ONLY format new or modified code. Global reformatting is FORBIDDEN.
- **Environment Isolation**: No `make`, real-device testing, or deployment scripts allowed without permission. **本仓库的 `Makefile`（compile/test/package/publish 等）已获用户许可（2026-08-17）**；发布目标从环境变量读令牌（`OVSX_TOKEN`/`VSCE_PAT`），令牌不进仓库。Publishing to VS Code Marketplace / Open VSX requires explicit user confirmation.
- **Side Effects**: For operations with external side effects (e.g., API calls, spawning `dsh` servers, npm install), notify the risk in `Thought` beforehand.
- **DSH 运行约束**: 扩展只能向 `127.0.0.1`/`localhost` 代发请求；不得弱化 DSH 的 `/api` 信任围栏；`dsh web` 不允许 `--host 0.0.0.0`。见 `doc/feature/00-dsh-vscode/spike-notes.md`（S2/F2）。

## 3. Pre-edit Check (Adaptive Gate)
**Before invoking ANY write/modify tools, conduct a scope assessment:**
- **Micro-edit** (typo fix, single-line CSS/comment): Output brief: `[Pre-edit OK] Scope trivial.`
- **Standard-edit** (logic change, refactoring, multi-line change): **MUST** output the full checklist (Note: Show the plan/checklist only; do not output proposed code in the chat):
  1. **Bracket Balance**: Are all `{}` `[]` `()` symmetric for this edit?
  2. **Symbol Dependencies**: Will any deleted/renamed symbols break other files?
  3. **Validation Plan**: What analysis/test command will run immediately after the edit? (`npm run compile` / `node --check` / `npm test`)
  4. **Path Safety**: Is the operation restricted to the target directory?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floatinghotpot/deepseek-harness-web-for-vscode](https://github.com/floatinghotpot/deepseek-harness-web-for-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
