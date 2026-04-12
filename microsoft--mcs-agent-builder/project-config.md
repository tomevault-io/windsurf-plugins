---
trigger: always_on
description: > **RULE ZERO — HARD STOP: Fire GPT-5.4 on every interaction. No exceptions.**
---

# MCS Automation — Core Rules

> **RULE ZERO — HARD STOP: Fire GPT-5.4 on every interaction. No exceptions.**
>
> Every user message triggers GPT co-generation via `node tools/multi-model-review.js`. This is enforced by hooks:
> - **UserPromptSubmit hook** injects a reminder on every prompt
> - **Stop hook** checks for attestation and blocks if GPT was not called
> - **Attestation file** written by `multi-model-review.js` on every successful (or attempted) call
>
> Pick the right command: `challenge` (before implementing), `ask` (design/questions), `review-code` (after code), `diagnose` (debugging), `generate-*` (MCS content). If GPT is unavailable (exit code 3), proceed alone — that counts as attempted. Full protocol: `.claude/rules/gpt-co-generation.md`

Automate Microsoft Copilot Studio (MCS) agent creation using a hybrid build stack: PAC CLI for listing agents, MCS LSP Wrapper for component sync, Island Gateway API for model catalog and eval upload, Dataverse API for agent creation and publishing, Direct Line API for testing, and user-guided manual steps for new OAuth connections.

Research Microsoft-first (MCS built-in > Power Platform > Azure > M365 connectors) because enterprise agents run best on the native stack. The **brief.json** is the single source of truth — everything flows from it.

---

## Workflow

```
INIT → CONTEXT → RESEARCH → [SOLUTION TYPE GATE] → BUILD (includes guard) → EVALUATE → [FIX] → [REPORT]
```

| Skill | Purpose | Dashboard |
|-------|---------|-----------|
| `/mcs-init` | Create project folder structure | — |
| `/mcs-context` | Pull M365 history via WorkIQ | — |
| `/mcs-research` | Read docs, identify agents, research components, enrich brief.json + generate evals | Research |
| `/mcs-build` | Pre-build validation (guard) + build agent(s) in MCS via hybrid stack | Build |
| `/mcs-eval` | Run eval tests, write results to brief.json | Evaluate |
| `/mcs-fix` | Analyze eval failures, apply fixes, re-evaluate | Fix Failures |
| `/mcs-refresh` | Refresh knowledge cache files | — |
| `/mcs-report` | Generate reports (brief/build/customer/deployment) | — |
| `/feedback` | File bug reports or feature suggestions via GitHub CLI | Sidebar |

Each skill has detailed instructions in its own `.claude/skills/*/SKILL.md`.

---

## Installed Plugins (Auto-Routing Rules)

8 plugins installed from `claude-plugins-official`. MCS skills remain primary — plugins handle tooling, code quality, and developer experience.

**Auto-firing (no manual invocation needed):**

| Plugin | Triggers When | What It Does |
|--------|---------------|-------------|
| `context7` | Claude needs non-Microsoft library docs (React, Vite, Zustand, etc.) | MCP server provides up-to-date library documentation. Complements `microsoft-learn` MCP. |
| `typescript-lsp` | Any `.ts`/`.tsx`/`.js`/`.jsx` file is edited | Background LSP for type checking, go-to-definition, error detection. |
| `frontend-design` | Frontend/UI work requested ("build a component", "redesign the dashboard") | Distinctive, production-grade UI generation with bold aesthetics. |
| `skill-creator` | User asks to create, improve, or eval a skill | Skill authoring framework with built-in eval benchmarking. |
| `claude-md-management` | User mentions CLAUDE.md quality, audit, or maintenance | Audits CLAUDE.md against codebase state, scores quality. |

**Routed by Claude (invoke automatically at these moments):**

| Plugin | When to Auto-Invoke | Slash Command |
|--------|-------------------|---------------|
| `code-review` | After creating a PR via `/commit-push-pr` or `gh pr create` — run automatically on the PR | `/code-review` |
| `commit-commands` | When user says "commit", "push", "create PR", or after completing a coding task and user approves | `/commit`, `/commit-push-pr` |
| `session-report` | When user asks about token usage, costs, or session efficiency; or at end of long sessions | `/session-report` |
| `claude-md-management` | At end of sessions that revealed missing CLAUDE.md context or after significant codebase changes | `/revise-claude-md` |

**Routing priority:** MCS skills (`/mcs-*`) always take precedence over plugins. Plugins handle code-level work; MCS skills handle platform-level work.

---

## Dual Model Co-Generation (Hook-Enforced)

Fire GPT-5.4 on **every interaction** — not just "non-trivial" tasks, **everything**. Enforced by three layers:
1. **UserPromptSubmit hook** (`.claude/hooks/gpt-reminder.js`) — injects reminder on every prompt
2. **Stop hook** (`.claude/hooks/check-gpt-attestation.js`) — blocks response if no attestation found
3. **Attestation** — `multi-model-review.js` writes `$TMPDIR/claude-gpt-attestations/<session>.json` on every call

Skip only when GPT is unavailable (exit code 3) — the tool writes an "unavailable" attestation, satisfying the hook. Full protocol, commands, merge rules, and value patterns in `.claude/rules/gpt-co-generation.md`.

---

## Core Philosophy

1. **Brief-driven build** — brief.json drives every build because a single source of truth prevents drift between design and execution. Fill gaps before building.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/microsoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
