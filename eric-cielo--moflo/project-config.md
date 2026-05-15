---
trigger: always_on
description: **This is an open-source library.** The package is installed as a `devDependency` into consumer projects to make Claude Code more effective in those projects. It is NOT a standalone application — every change you make ships to N consumers via `npm install moflo` and runs from their `node_modules/moflo/...` on their machines.
---

## ⚠ MoFlo is a library shipped to other projects — READ FIRST, every change

**This is an open-source library.** The package is installed as a `devDependency` into consumer projects to make Claude Code more effective in those projects. It is NOT a standalone application — every change you make ships to N consumers via `npm install moflo` and runs from their `node_modules/moflo/...` on their machines.

**Before writing any code or opening any PR**, articulate three things to yourself (and to the user, if non-trivial):

1. **Consumer surface touched** — which file class is this? `bin/`, `src/cli/`, `.claude/scripts/`, `mcp-tools/`, hook handlers, init/, settings-generator, `claudemd-generator.ts`, anything synced to `node_modules/moflo/`. Editing a test or internal helper has near-zero blast radius. Editing the launcher, a hook, or anything `flo init` writes touches every consumer.
2. **Failure mode for the on-the-current-version consumer** — what breaks for someone on `moflo@<previous>` who picks this change up via `npm install moflo@<latest>`? Is there a migration? Does the consumer need to do anything? Will their existing `.moflo/` state still parse? Their hooks still wire?
3. **Round-trip cost** — does this require *publish-then-reinstall* before it takes effect (most runtime fixes do — see the dogfood note below), or does the source edit alone suffice (build/test/internal code)?

**Concrete examples of what "consumer impact" looks like in practice:**

- **#860** — missing `CLAUDE_CODE_HEADLESS` guard in the launcher → daemon-spawned headless Claude in *every* consumer triggered the indexer chain, pegging CPU on a 15-min loop. Cost: a stale-since-shipped guard fired in N consumer environments before anyone noticed.
- **#854** — silent `catch {}` in launcher upgrade flow → consumers stuck across moflo 4.8.x → 4.9.2 with partial migrations and no diagnostic crumbs. Cost: 4+ versions worth of consumer-invisible breakage.
- **#586 / collapse epic** — workspace renamed `@claude-flow/*` → `@moflo/*` → every consumer with bare imports needed coordinated migration; 5-invariant drift guard now blocks regressions.
- **#798** — MCP swarm/agent handlers stubbed out as a "simplification" → headline product surface broke in every consumer; 10-story epic to repair.

If you can't name the consumer surface and failure mode for a non-trivial change, **stop and re-scope** before writing code. "I'll just clean this up while I'm here" is the antipattern that produced every bullet above.

See `feedback_consumer_blast_radius.md` (auto-memory) for the full posture.

---

## ⚠ Diagnosing runtime symptoms — read this first

Before opening any issue or "fixing" anything observable in the editor (statusline, daemon, hooks, indexer, upgrade UI), MUST read `.claude/guidance/internal/dogfooding.md` § Runtime Symptom Diagnosis. MoFlo dogfoods itself; runtime symptoms come from `node_modules/moflo/...`, not source. Skip this and you'll waste a session debugging code that isn't even running.

---

<!-- MOFLO:INJECTED:START -->
## MoFlo — AI Agent Orchestration

### FIRST ACTION ON EVERY PROMPT: Search Memory

Your first tool call MUST be `mcp__moflo__memory_search` — before any Glob/Grep/Read. Search `guidance`, `patterns`, and `learnings` every prompt; add `code-map` when navigating code, `tests` when looking for test inventory or coverage. When the user says "remember this", call `mcp__moflo__memory_store` with namespace `learnings`.

### Auto-enforced gates

- **TaskCreate-first**: Call `TaskCreate` before spawning the Agent tool
- **Task Icons**: `TaskCreate` entries MUST use ICON+[Role] format — see `.claude/guidance/moflo-task-icons.md`

### Tools

Prefer MCP (`mcp__moflo__*` — memory, swarm, agent, task, hooks, hive-mind, neural) over the CLI. CLI binaries: `flo` (main), `flo-search` (semantic search), `flo doctor --fix` (heal). Full catalog: `.claude/guidance/moflo-core-guidance.md`.

### After upgrading MoFlo

After `npm install` touches moflo, check `.moflo/restart-pending.json` — if present, surface its `message` field to the user verbatim, then delete the file. (Claude Code only loads new hooks/MCP/launcher at session start.)

### Full Reference

- Subagents protocol: `.claude/guidance/moflo-subagents.md`
- Task + swarm coordination: `.claude/guidance/moflo-claude-swarm-cohesion.md`
- CLI, hooks, swarm, memory, moflo.yaml: `.claude/guidance/moflo-core-guidance.md`
<!-- MOFLO:INJECTED:END -->

## ⚠ Editing guidance — read both rule sets first

Before creating, rewriting, or editing any `.claude/guidance/**/*.md` file, MUST read both:

1. `.claude/guidance/shipped/moflo-guidance-rules.md` — universal writing rules (Purpose line, imperative voice, decision tables, concrete examples, 500-line cap, specific H2 headings, anti-patterns, RAG chunking, See Also).
2. `.claude/guidance/internal/guidance-rules.md` — moflo-only extensions (`moflo-` prefix on shipped files, shipped/internal partition contract, bucket-decision rules).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eric-cielo/moflo](https://github.com/eric-cielo/moflo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
