---
trigger: always_on
description: **Last verified:** 2026-03-28
---

# AGENTS.md — AI Coding Guide for This Project

**Last verified:** 2026-03-28
**Owner:** UPOS Engineering (AI Workflow Maintainers)

This file is the complete policy reference for any AI coding agent working on this codebase.
For routine work, start with `AGENTS-FAST.md`, then use this file for full policy and edge cases.
Before writing code, read the relevant domain document in `ai/`.

Process lives here (design → plan → execute, TDD, verification, five checks). Domain reference (UI, DB, auth, conventions, known issues) lives in `ai/`. Fast execution defaults live in `AGENTS-FAST.md`.
Repo-local workflow helpers live in `.cursor/skills/` and `.cursor/prompts/`; use them when a task matches instead of re-deriving the same workflow from scratch.

---

## 0. Document Control

### 0.1 Rule Priority (If Instructions Conflict)

Apply the first matching rule in this order:
1. Direct user instruction for the current task
2. `AGENTS.md` (this document)
3. Cursor rules in `.cursor/rules/*.mdc` (when applied by the editor — e.g. Laravel constitution, Blade refactor)
4. Domain-specific `ai/*.md` files
5. Existing local code patterns

If a conflict still cannot be resolved safely, stop and ask the user before writing code.

<!-- start ask mode -->

### 0.1a Ask mode vs Implement mode

**In Cursor:** The Ask/Agent toggle in the UI already sets the mode. **Do NOT ask the user to clarify mode.** Respect whatever mode is active and proceed immediately.

- **Ask mode active** → Read, search, explain. No file writes, no commands.
- **Agent mode active** → Full workflow (design → plan → implement → verify).

**In Codex or environments without a UI mode selector only:** If the user has not already stated intent (e.g. "just explain" or "make the change"), infer from the request — a fix/feature request means implement, a question means explain. Only ask if the intent is genuinely ambiguous after reading the message.

<!-- end ask mode -->

### 0.1b Fast intent router

Use the first matching lane before doing broader workflow:

| Intent | Use when | Default tool order | Notes |
|---|---|---|---|
| `tiny` | Single-file or tightly scoped request | restate goal → inspect target → edit → verify | Prefer this lane for low-risk, bounded changes. |
| `explain` | User wants understanding, not changes | grep/semantic search → read → answer | No code edits. No `project_map`, no `laravel_mysql`, no `warm_cache`; use grep and read_file_cache only. |
| `analyze` | Audit module, clone, or understand codebase | startup health → project_map/filesystem → grep first → targeted/parallel reads → full read only when editing | See ai/agent-tools-and-mcp.md §2.8. |
| `dependency-eval` | Evaluate a GitHub repo, package, or dependency before adoption | project_map/filesystem → `resource://composer`/manifests → fetch upstream docs → compare to repo conventions → recommend adopt/adapt/reject | Use for external code that could change repo dependencies or architecture. Run `project_map`/`composer` only when the question explicitly asks about adoption for **this** repo; skip for generic tool comparisons or "does this tool help?" questions. |
| `external-adapt` | Adapt a pattern or example from an external repo into this codebase | project_map/filesystem → grep for closest local example → fetch upstream docs/examples → map to route/Form Request/Util/controller/view/module → plan or implement | Prefer adapting the minimum useful pattern; do not copy upstream structure wholesale. |
| `investigate` | Something is broken or unclear | grep → read → compare render/update flow → answer or fix | Use 0.4a/0.4b for “stops working” bugs. |
| `review` | User asks for a review or audit | grep/read changed area → identify findings → verify evidence | Findings first, summary second. |
| `plan` | User wants design or implementation plan | inspect repo truth → write numbered plan → list verification | Do not invent missing repo facts. |
| `execute-plan` | User attaches or references `.cursor/plans/*.plan.md` to execute or "plan from" | startup health → read plan → execute phases/tasks in order; derive steps that match the plan as written | Do **not** rewrite, restructure, or replace the plan. See .cursor/plans/README.md §7. |
| `implement` | User wants a fix or feature | startup health → inspect → numbered plan → edit → verify | Default when the user asks to make the change. For large multi-file phases, see 0.1e for worker delegation. |
| `log-scan` | User asks to scan/fix Laravel log, check logs, or fix issues from `storage/logs` | glob latest log → read log → parse errors → investigate → fix → verify | See 0.4e. Use implement mode. |
| `lint-fix` | User asks to fix linter errors, IDE diagnostics, or "fix lint" | Read lints (scope: path or repo) → fix each finding → re-run lints | See 0.4f. Use implement mode. |
| `test-fix` | User asks to fix failing tests or pastes test output | parse test output → locate failure → fix test or code → re-run tests | See 0.4g. Use implement mode. |
| `tenant-audit` | User asks to audit/fix tenant security, missing business_id, or route auth | grep checklist → fix or report each finding | See 0.4h. Use implement mode. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
