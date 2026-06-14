---
trigger: always_on
description: AGH is an agent operating system — a Go single-binary daemon that manages AI agent sessions via ACP (Agent Client Protocol). It spawns ACP-compatible agents (Claude Code, OpenClaw, Hermes, etc.) as subprocesses, communicates via JSON-RPC over stdio, persists events in SQLite, and exposes interfaces via HTTP/SSE (web UI) and UDS (CLI). A Fumadocs site at `agh.network` documents the runtime and the AGH Network protocol.
---

## Project Overview

AGH is an agent operating system — a Go single-binary daemon that manages AI agent sessions via ACP (Agent Client Protocol). It spawns ACP-compatible agents (Claude Code, OpenClaw, Hermes, etc.) as subprocesses, communicates via JSON-RPC over stdio, persists events in SQLite, and exposes interfaces via HTTP/SSE (web UI) and UDS (CLI). A Fumadocs site at `agh.network` documents the runtime and the AGH Network protocol.

**Goals**: daemon single-binary in background, strong observability, agent-first system (agents manipulate via CLI + REST), highly extensible, highly configurable.

**Core product premise**: every capability must be both extensible by the runtime and manageable by agents. Features are incomplete if they only work through internal Go calls or the web UI.

## Greenfield Alpha — Zero Legacy Tolerance

- **No production users exist.**
- Never sacrifice code quality for backward compatibility.
- Never write migration, compatibility, or defensive code for old state — delete obsolete code instead of working around it.
- **Hard cuts, not bridges:**
  - Renames must update code, storage, APIs, CLI, extensions, specs, RFCs, and `.compozy/tasks/*` artifacts all in a single change.
  - Do not create aliases, dual fields, or schema fallback paths.
- Every breaking-change techspec **MUST** explicitly list its delete targets.

## Critical Rules

- **`make verify` MUST pass** before completing ANY task (runs `codegen-check → bun-lint → bun-typecheck → bun-test → web-build → fmt → lint → test → build → boundaries` across the entire monorepo, not just `web/`). Zero warnings, zero errors. Exceptions are just if you just update documentation that don't affect test, lint or typecheck.
- **`make lint` (Go golangci-lint) and `make bun-lint` (oxfmt + oxlint over every workspace) both have zero tolerance** — any warning or lint issue is a blocking failure.
- **Check dependent package APIs** before writing integration code or tests.
- **Never add dependencies by hand in `go.mod`** — always use `go get`.
- **Never run destructive git commands** (`git restore`, `git checkout`, `git reset`, `git clean`, `git rm`) **without explicit user permission**. If the worktree contains unexpected edits, read and work around them.
- <critical>NEVER ignore errors with `_` in production code or in tests — every error must be handled or have a written justification.</critical>
- **Test placement is mandatory before test creation.** Before adding, moving, or expanding any test, name the invariant, owning layer, and canonical suite. Default to editing an existing canonical suite; do not create standalone or duplicate regression tests unless no existing suite can own that invariant. Static/prose/CSS/generated/snapshot/config tests require explicit product-contract rationale.
- **Static-artifact tests are exceptional.** Forbidden by default: Storybook setup/config/globs/decorators/bootstrap, CI workflow/action YAML, Mage/Make/package-script plumbing, generic config/file existence, generated-output drift, snapshots/goldens, prose/CSS literals, and coverage-padding. Prefer stronger gates (`make verify`, `make codegen-check`, builds, link checks, Storybook visual capture, or real command smoke). Coverage floors never justify filler tests.
- <critical>NEVER COMMITS `ai-docs/` or `.tmp/` TO THE REPO. They are local tracking artifacts.</critical>
- **Always use subagents for exploration** to avoid bloating your own context. Route by shape:
  - **Single-file lookup or one targeted question** ("where is X defined?", "which files reference Y?") → `Explore` subagent (read-only, returns excerpts inline).
  - **Multi-area research that needs written analysis artifacts** (3+ distinct slices, cross-cutting question, output must persist for a TechSpec / ADR / task) → activate the `agent-exploration` skill, which scouts the territory, dispatches N `explorer` subagents in parallel under the scoped-write contract, and synthesizes `summary.md`.
  - **Competitor / reference-repo research** against `.resources/<repo>/` → activate `cy-research-competitors` (the specialized variant) instead of `agent-exploration`.
- **Subagents default to read-only.** Use them for analysis, exploration, and parallel research. The author of every code change is the agent paired with the user, and subagent output is treated as evidence. A subagent may write, edit, or commit only when the parent's prompt explicitly delegates that action (e.g. "write the analysis file at X", "apply the fix in Y"); otherwise it must return its output for the parent to write.
- **ALWAYS CHECK** the `internal/CLAUDE.md` when doing Go-related stuff
- **ALWAYS CHECK** the `web/CLAUDE.md` when doing things related to the web package

## Workflow Rules

These govern how features move from idea to ship. Internalize them before opening a TechSpec or running a task.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/agh](https://github.com/compozy/agh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
