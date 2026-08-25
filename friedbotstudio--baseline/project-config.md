---
trigger: always_on
description: This file binds Claude's behavior — Claude Code, in this codebase, in this session. Every rule below is mandatory unless waived by an explicit `exceptions` entry in `.claude/state/workflow.json` (set only by `/triage`).
---

# Claude Code Baseline — In-Session Constitution

This file binds Claude's behavior — Claude Code, in this codebase, in this session. Every rule below is mandatory unless waived by an explicit `exceptions` entry in `.claude/state/workflow.json` (set only by `/triage`).

**Genesis prompt.** `docs/init/seed.md` is the governing specification of this baseline. When it and this constitution conflict, **seed.md governs** and you SHALL stop and surface the drift before acting. When this constitution and the implementation conflict, **this constitution governs** and the implementation SHALL be corrected.

**Enforcement.** The 26 hooks in `.claude/hooks/` are the enforcement layer, mapped to their Articles in **Article VIII**.

---

## Article I — Authority and precedence

1. **Genesis** — `docs/init/seed.md` is the source of truth for the baseline's shape, components, and rebuild protocol.
2. **Constitution** — `CLAUDE.md` is the source of truth for Claude's in-session behavior.
3. **Implementation** — hooks, skills, commands, subagent, MCP servers, and config actuate and enforce (1) and (2).
4. **Order of precedence** — `seed.md` > `CLAUDE.md` > implementation. Lower binds higher only via amendment in seed.md, which then propagates to this file, then to disk.
5. **Project amendments** — Article XI reserves space for project-owner amendments. They bind alongside the baseline Articles but **SHALL NOT** contradict them.
6. **Size cap (this file)** — `CLAUDE.md` SHALL NOT exceed **40,000 characters** and carries binding rules only; history, narration, and reference appendices live in the annex. `audit-baseline` enforces the cap, which also binds the byte-equal mirror `src/CLAUDE.template.md`. The governance suite pins a tighter advisory target (seed.md §14).

## Article II — Architectural principle

**Decisions live in main context. Subagents only execute pre-decided recipes in parallel or in the background.**

The baseline ships exactly **one** subagent: `swarm-worker`. Its sole sanctioned use is `Skill(scenario)` then `Skill(implement)` against a fully-specified recipe in an isolated git worktree during `/swarm-dispatch`. It SHALL NOT make design choices, pick abstractions, or expand scope, and SHALL NOT be invoked outside `/swarm-dispatch`.

A bounded maker/checker round-trip MAY run on the Workflow runtime under **§II.A** — oracle-bound read-only checkers may fan out; one maker, judgment checkers, and the one-subagent count stay bound. Full text: `seed.md §4.2`.

Every other capability — code authoring, scouting, research, security and spec review, prose, UI design — is a **skill** running in main context. Five declare a mandatory sub-skill contract:

| Skill | Mandatory sub-skill | Conditional |
|---|---|---|
| `scenario` | `code-structure` | — |
| `implement` | `code-structure` | current-docs check for third-party APIs (`context7`) |
| `design-ui` | `impeccable` | — |
| `prose` | `humanizer` (always) | `copywriting` / `documentation` / `technical-tutorials` by register |
| `technical-writer` | `technical-writing`, `reader-level`, `humanizer` | — |

You SHALL NOT route **binding judgment** — a written decision or production change (design tone, code architecture, security calls, scenario selection) — through a subagent. **Read-only advisory subagents** (Explore/Plan, scout/research gathering, §II.A checkers) MAY gather and advise; they write nothing and decide nothing. Full clause: `seed.md §4.2-A`.

## Article III — Session-start procedure (MANDATORY)

On every new session, before any work, you SHALL:

1. **Read** `.claude/project.json` and check the `configured` field.
2. **If `configured: false`** — `/init-project` has not run. The repo is in **project-agnostic mode**, a sanctioned state: hooks are active but `test_runner` and `lint_runner` run in guide mode. Greet with the verbatim framing in annex §2, then proceed with whatever the user asks — the mode is **allowed** and `/init-project` is not required. `setup_guard` reminds on writes but does **not** block; every other guard stays hard.
3. **If `configured: true`** — read `docs/init/seed.md` §16 if present, then tell the user:
   > "Configured for `<stack>`. Run `/triage \"<request>\"` to start a workflow, or `/harness` for the full pipeline."
4. **Memory check.** `memory_session_start` injects the index — entry counts, stale rows, concepts, resume snapshot. It reports the `_pending.md` count and prompts no action; Phase 10.7 flushes inside every workflow.
5. **Git-repo check.** Run `git rev-parse --is-inside-work-tree 2>/dev/null`. If non-zero, surface once per session that gate C and `commit` are auto-excepted and the workflow ends after `/archive`. A sanctioned mode — Art. IV phase 11 and Art. VII are git-conditional.
6. Once per session is sufficient. You SHALL NOT repeat the greeting on every prompt.

## Article IV — Workflow ordering (MANDATORY)

The 11-phase workflow is the only sanctioned path from request to commit. Phase ordering is enforced at the Write boundary by `track_guard` (Art. VIII).

| # | Phase | Invocation | Output |
|---|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [friedbotstudio/baseline](https://github.com/friedbotstudio/baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
