---
trigger: always_on
description: **How binding is a line in this file?** Three tiers: **Anchor** (never deviate), **Default** (the normal call; deviate by stating a `[DEVIATION]` line that cites a fact signal, then *keep working*), **Guidance** (advisory). This file's own baseline is **Default** and lines above that baseline are marked inline — @rules/discretion.md classifies the plugin-managed `rules/*.md`, not this file, but its **Anchor Register is the authority everywhere**: a line here that hits the Register is Anchor no m
---

# sd0x-dev-flow — Harness Engineering for Claude Code

**How binding is a line in this file?** Three tiers: **Anchor** (never deviate), **Default** (the normal call; deviate by stating a `[DEVIATION]` line that cites a fact signal, then *keep working*), **Guidance** (advisory). This file's own baseline is **Default** and lines above that baseline are marked inline — @rules/discretion.md classifies the plugin-managed `rules/*.md`, not this file, but its **Anchor Register is the authority everywhere**: a line here that hits the Register is Anchor no matter how it is worded or where it is restated.

Judgment inside the Default range is the expected behaviour, not a tolerated exception: decide from the change in front of you and continue. Uncertainty alone is not a reason to stop and ask — the human exits are the union of the ones enumerated in @rules/auto-loop.md and @rules/scope-discipline.md; those two enumerations are the closed list, not this sentence.

## Required Checks (Stop Hook reminded)

This table constrains the **end state**, not your choreography. How you batch edits, how deep you review, and when you run each gate are yours to choose; what is fixed is that every gate a change class requires has passed *after the last edit in that class*.

| Change Type | Must Run | Can Skip |
|-------------|----------|----------|
| code files | `/codex-review-fast` -> `/precommit` | - |
| `.md` docs | `/codex-review-doc` | `/codex-review-fast` |

Comment-only edits get no free pass: comments can carry compiler/lint/build directives (`scripts/lib/utils.js:142` proves it), so edits to code files are conservatively classified as code even when only comments changed. Before PR: `/pr-review`.

> The Stop Hook is a **reminder, not a gate** (hook-lightweighting, 2026-08-13): it prints which gates the reminder state still shows as owed and always exits 0. What binds is the behaviour layer — the terminal completion invariant in @rules/auto-loop.md — with verdicts recorded via `node scripts/review-state.js note`, digest-bound so an edit re-opens its plane. Details: `docs/features/hook-lightweighting/2-tech-spec.md` §3.

## Auto-Loop

| After editing... | Review | Then on pass |
|------------------|--------|--------------|
| code files | `/codex-review-fast` | `/precommit` |
| `.md` docs | `/codex-review-doc` | (done) |

The terminal completion invariant, tiers, sub-threshold handling, and sentinels live in @rules/auto-loop.md (highest priority). One reviewer — Codex — by default; `--dual` is `/codex-review-branch` opt-in only.

**What is yours to decide**: the effective tier (escalate above the configured baseline when the change warrants it — never below), when to batch and when to review, how deep to review, and when 80 is a passing grade rather than another round. **What is not**: the four Anchor corollaries — Declaring ≠ Executing, Summary ≠ Completion, Fixing ≠ Verifying, and an edit re-opening its own plane's gate. Naming a gate is not running it, and no context or session pressure outranks an open one. Sub-threshold findings are **logged and passed**, not weighed: @rules/auto-loop.md § Sub-Threshold Findings allows exactly two on-the-spot fixes (a one-line fix in a file already open, and a finding whose severity was mis-assigned to something that is really a security or data-integrity defect) — anything else is a `[DEVIATION]`, not a judgment call.

Skill discovery: no command table here by design — each skill's frontmatter `description` (`skills/<name>/SKILL.md`) is the dispatcher's discovery interface, and `docs/skill-catalog.yml` is the canonical registry. Typical flows: feature work → `/feature-dev`, bug fixing → `/bug-fix`, commits → `/smart-commit`. Tech stack: Node.js · JavaScript · node:test; key entrypoints: `scripts/run-skill.sh` (skill script runner), `package.json`.

## Development Rules

Tier is marked per rule; the unmarked ones are Default and you may deviate with a stated signal.

1. *(Guidance)* **Reference existing code** -- find similar files first, keep style consistent
2. **Test command** -- `npm test`（`node --test $(find test -name '*.test.js')` — npm scripts 走 `/bin/sh`，`**` glob 不展開巢狀目錄，勿用 `test/**/*.test.js`）
3. **⚓ Anchor** — **Author attribution** -- use developer's GitHub username, never AI names (exception: `/smart-commit --ai-co-author`). Forbidden patterns in commit messages **and PR title/body** (canonical source: `scripts/commit-msg-guard.sh`): Co-Authored-By AI, Generated-by tags, emoji robot tags. Commits: install `commit-msg-guard.sh` via `/install-scripts`. PRs: `/create-pr` Step 4b enforces sanitization automatically.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sd0xdev/sd0x-harness](https://github.com/sd0xdev/sd0x-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
