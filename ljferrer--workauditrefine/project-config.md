---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

📚 **Durable engineering lessons live in `docs/learnings/`** — one fact per Markdown file, provenance-tagged frontmatter. Before changing a subsystem, read the lessons that name it (plain Read/Grep, or ranked retrieval via the `work-audit-refine` plugin's `war-memory` query).

In this repo the plugin ships at `skills/_shared/`, so that ranked query is `node skills/_shared/war-memory.mjs query '<terms>' --repo docs/learnings` (needs Node >= 24; plain Read/Grep works without it).

## What this repo is

WAR (Work · Audit · Refine) — a Claude Code **plugin**; this repo *is* the plugin (`.claude-plugin/plugin.json` + `skills/` + `agents/` + `hooks/`). It orchestrates multi-phase plan execution with worker / auditor / refinery / servitor agents over git worktrees and GitHub issues. There is no application build: developing here means editing skill prose (`skills/*/SKILL.md`), the per-phase Workflow engine (`skills/war/assets/workflow-template.js`), shell hooks/floors, the memory CLI, and their tests. `CONTEXT.md` is the glossary (ubiquitous language); `docs/adr/` records the binding decisions, numbered sequentially — `ls docs/adr/` for the current head.

## Commands

Node ≥ 24 required (`node:sqlite`/FTS5). No package.json, no Makefile, no top-level runner.

```bash
# All JS tests (quoted glob REQUIRED — a directory arg fails on Node 24)
node --test 'skills/**/*.test.mjs'

# One JS test / one shell test (shell tests are bash-3.2-safe and cwd-independent)
node --test skills/war/assets/war-config.test.mjs
bash hooks/validate-worktree-scope.test.sh

# All shell tests — anchor to hooks/ and skills/; a repo-root find picks up
# ~100 stale duplicates under .claude/worktrees/
for f in $(find hooks skills -name '*.test.sh' | sort); do bash "$f" || exit 1; done

# Redaction lint (exactly what CI runs — the only thing CI runs)
node skills/_shared/war-memory.mjs lint docs/learnings/

# Memory queries / index render — ALWAYS pass --local; the cwd fallback
# silently creates a stray ./memory dir in the repo
node skills/_shared/war-memory.mjs query '<terms>' --repo docs/learnings
node skills/_shared/war-memory.mjs render-index --local "$CLAUDE_MEMORY_LOCAL" --repo docs/learnings
```

Local plugin iteration: `claude --plugin-dir /path/to/WorkAuditRefine`, then `/reload-plugins` after each edit (local paths resolve to version `unknown`, so reloads always pick up changes).

**Releasing:** a version bump must update all four slots together or the release is a silent no-op — `.claude-plugin/plugin.json` `version`, `.claude-plugin/marketplace.json` `metadata.version` **and** `plugins[0].version`, and the `README.md` `## Status` line (replace-in-place, no badge). `skills/war/assets/version-slots.test.mjs` locks the four slots in lock-step (fail-closed — a partial bump is a red test, and it also guards the README Releasing prose against the "three files" undersell) **and enforces a monotonic floor**: `plugin.json`'s version must be ≥ the highest version in a bounded first-parent window of slot-touching history, so a coherent all-four-slots *downgrade* — which lock-step passes — goes red until a Lead/operator restore commit lands. The floor fails open only outside a usable git context (no git, non-zero git exit, or an empty window), never on a broken parse. Version literals inside plans/roadmaps are non-authoritative — resolve the next free patch from the slots at land time.

## The pipeline is gospel: interview → plan → red-team → war

**One interview, one merged artifact.** The pipeline's execution artifact is the **merged plan** (`docs/plans/YYYY-MM-DD-<slug>.md`): Part 1 the ratified *decision record* — `## Context` with evidence-tagged claims (D4 tag forms), `## Pivotal constraints`, `## Resolved design tree`, a required `## Assumptions ledger` — and Part 2 the decomposed phases, flat H2s carrying today's exact extraction headings: `## Commander's Intent` (Purpose / Method / numbered individually-checkable End state — plan slice is the floor, intent is the ceiling), `## Build order`, per-task `Files:` / `Plan slice:` / `requiresTest` / `requiresPackaging` / `deps` / `target repo`, and a required `## Deferred validations (backstops)` section (explicit `None` allowed). A **design spec** (`docs/specs/YYYY-MM-DD-<slug>-design.md`) is an *input shape* only — how `/survey-corps` synthesis and externally-brought drafts arrive; it carries **no dispatch structure**, `/war` cannot execute one, and conversion turns it into the merged plan (legacy spec + plan pairs are grandfathered in place).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ljferrer/WorkAuditRefine](https://github.com/Ljferrer/WorkAuditRefine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
