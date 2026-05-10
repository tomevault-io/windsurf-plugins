---
trigger: always_on
description: AI Workflow Orchestrator for Claude Code — systems-level orchestration of agents with per-plan cycles, tracer-first execution, calibrated review, and continuous improvement. **The context engine is an Obsidian vault.**
---


# Conducty

AI Workflow Orchestrator for Claude Code — systems-level orchestration of agents with per-plan cycles, tracer-first execution, calibrated review, and continuous improvement. **The context engine is an Obsidian vault.**

## The Context Engine

Conducty's plans, designs, project context, improvements, failure patterns, metrics, and prompt logs all live in an **Obsidian vault** at `$CONDUCTY_VAULT` (default `~/Obsidian/Conducty/`). Every note is wikilinked to its peers — designs link to the plans that consume them, plans link to the context they loaded and the improvements they're testing, failure patterns link to the prompts that surfaced them. Future plans navigate this graph rather than re-grepping a flat history.

**Read [[conducty-obsidian]] before reading or writing any state.** It defines the vault location, naming, frontmatter, link conventions, and indexes.

> [!important] Per plan, not per day
> Conducty runs **per plan**, not per day. Multiple plans per day are normal. Plan notes are timestamped: `Plan YYYY-MM-DD HHmm [Topic].md`. The same applies to designs and improvements. Accumulating notes (`Failure Patterns`, `Metrics`, `Prompt Log`) are singular files appended to over time.

## Skills

Claude Code discovers Conducty skills from `~/.claude/skills/` after running `./install-claude-code.sh`. Each skill is a folder with a `SKILL.md` file that Claude loads on demand based on its `description` trigger.

| Skill | Phase | Trigger |
|-------|-------|---------|
| [[conducty-system]] | foundation | session start, "what is Conducty" |
| [[conducty-obsidian]] | context engine | before any state I/O — vault conventions |
| [[conducty-bootstrap]] | onboarding | first run, "set up Conducty", empty vault |
| [[conducty-shape]] | shape | "shape", "design", "brainstorm" |
| [[conducty-plan]] | plan | "plan", "batch plan", "create a plan" |
| [[conducty-tdd]] | discipline | "TDD", "red-green-refactor" |
| [[conducty-terse]] | discipline | "terse mode", "compress prompts", "tight prompts" |
| [[conducty-execute]] | execute | "execute the plan", "run group A" |
| [[conducty-verify]] | verify | before any "done" / "passed" claim |
| [[conducty-debug]] | debug | "why did this fail", "investigate" |
| [[conducty-checkpoint]] | checkpoint | "checkpoint", group boundary |
| [[conducty-review]] | review | "review this plan" |
| [[conducty-improve]] | improve | "what did we learn", "retrospective" |
| [[conducty-code-review]] | post-plan | "review my changes", "review this PR", whole-branch holistic review |
| [[conducty-ship]] | ship gate | "ship it", "ready to merge", pre-merge battery |
| [[conducty-context]] | support | "load context", "refresh context", "ingest project" |
| [[conducty-worktrees]] | support | parallel prompts, same repo |
| [[conducty-dialectic]] | decision | "analyze this decision", "debate this" |
| [[conducty-vault-graph]] | maintenance | "vault audit", "vault health", weekly hygiene |

## Vault Layout (recap)

Vault is **nested by category** — per-instance notes get a directory, per-project context lives under `Context/{Project}/`, accumulators live under `Accumulators/`. Wikilinks resolve by basename across all subfolders, so directory placement is purely organizational. See [[conducty-obsidian]] for the full layout reference.

Per-instance notes (timestamped) — directory + filename pattern:

- `Plans/Plan YYYY-MM-DD HHmm [Topic].md`
- `Designs/Design YYYY-MM-DD HHmm {Topic}.md`
- `Improvements/Improvement YYYY-MM-DD HHmm.md`
- `Code Reviews/Code Review YYYY-MM-DD HHmm.md`
- `Ship Reports/Ship Report YYYY-MM-DD HHmm.md`

Project context is a **sub-graph** under `Context/{Project}/` — one hub plus several slices (see [[conducty-context]]):

- `Context/{Project}/Context {Project}.md` (hub)
- `Context/{Project}/Context {Project} Architecture.md`, `... Conventions.md`, `... Invariants.md`, `... Hotspots.md`, `... Tests.md`, `... Glossary.md`
- `Context/{Project}/Modules/Context {Project} {Module}.md` (per bounded-context, optional)
- `Context/{Project}/Refreshes/Context Refresh {Project} YYYY-MM-DD HHmm.md` (per refresh)

Indexes:

- `Conducty Index.md` (vault root)
- `Indexes/Plans Index.md`, `Indexes/Designs Index.md`, `Indexes/Context Index.md`, `Indexes/Improvements Index.md`

Accumulating notes (singular files, prepend new entries):

- `Accumulators/Failure Patterns.md`
- `Accumulators/Metrics.md`
- `Accumulators/Prompt Log.md`

## Claude Code Tooling

Conducty skills assume Claude Code's native tools:

- **Task tool** — dispatch implementer/reviewer subagents (used by [[conducty-execute]]). Pass `isolation: "worktree"` for automatic worktree handling, or use [[conducty-worktrees]] for explicit named worktrees.
- **Read / Write / Edit** — file operations (prefer Edit on existing notes; Write only for new ones)
- **Bash** — verification commands, git, test runners, `mkdir`/`ls` against the vault
- **Grep / Glob** — codebase exploration during [[conducty-context]] and [[conducty-debug]], vault navigation in [[conducty-plan]]
- **TaskCreate / TaskUpdate** — in-session progress tracking (separate from vault plan notes)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertbarclayy/conducty](https://github.com/robertbarclayy/conducty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
