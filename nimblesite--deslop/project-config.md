---
trigger: always_on
description: <!-- agent-pmo:b636503 -->
---

<!-- agent-pmo:b636503 -->
# Deslop Live — Agent Instructions

## Rule zero — query the Deslop MCP before you write code

Deslop ships the duplicate-detector; its own code must be the cleanest in the
fleet. That only holds if **every agent queries the live Deslop MCP before
authoring code** — prevention, not cleanup.

**LAW — call `find-similar` BEFORE writing any code unit** (function, method,
class, helper, fixture, test setup, parser branch, error type, route handler,
view model — anything past a few lines). Pass a byte range (`path`,
`start_byte`, `end_byte`) or a snippet (`snippet`, `language`). Do NOT skip it
because the code "looks new" — most clones are written by someone certain it
was new.

- `signals.fused ≥ 0.85`, or bucket `identical` / `nearly_identical` → **do not
  write the copy.** Reuse the canonical occurrence; extract a helper if needed.
- `fused < 0.6` (empty or structurally distant) → author it.
- `0.6 ≤ fused < 0.85` → read the canonical occurrence and bias toward reuse.
- `structural_only` → shape-only match, often sibling boilerplate — read first.

**STOP DEAD if the MCP is unreachable or wrong.** If the Deslop MCP is
unavailable, errors, returns stale data, or gives an answer you can tell is
incorrect: **halt. Do not write code, do not guess, do not fall back to memory.**
A duplicate that lands because the gate was down is the exact failure this repo
exists to prevent. Tell the user and wait.

**LOG A GH ISSUE for every Deslop defect.** False positive, wrong bucket, stale
generation, missing cluster, MCP/IPC error — file it immediately with
`gh issue create` (include the cluster id or the triggering snippet). Never
silently work around a defect, widen thresholds, or mark clusters hidden. (`gh`
is the GitHub CLI, not `git` — the one allowed exception to the no-git rule.)

**Other Deslop tools:** dedup of *existing* duplicates → `top-offenders` then
`cluster-by-id`; a whole file → `report-for-file`; a block → `report-for-range`;
the JSON shapes → `schema-doc` once per session.

⚠️ **Never kill a VS Code process — including browser-hosted instances.** The user cannot recover from this; do not do it. ⚠️

⚠️ **Token discipline.** Check file size before reading. Prefer `Grep` over `Read`; use `offset`/`limit`. Make the smallest diff that solves the problem. Delete dead code, unused imports, and stale comments. Call out irrelevant context before proceeding — bloat degrades reasoning. ⚠️

⚠️ **Quality bar.** This codebase is held to an A+ standard: every change must pass review at a top-tier engineering organization (Google / Meta / Microsoft caliber). No exceptions, not even for a single line. Substandard code is fixed immediately, never deferred. ⚠️

⚠️ **"Deslop.live" (reactive) means the whole loop** (watcher → scheduler → session → broadcast → UI). An incremental update drives the entire pipeline, including a reactive UI refresh. ⚠️

⚠️ DO NOT USE GIT, ESPECIALLY NOT STAMPING YOURSELF AS COAUTHOR ⚠️

## Project Overview

**Deslop** (a.k.a. Deslop Live) is a **live duplicate-code analysis server** for AI coding agents and the humans driving them. The shipping surfaces are `deslop-lsp` (LSP server feeding live clone warnings to any LSP-capable editor) and `deslop-mcp` (MCP server letting Claude Code / Cursor / Copilot / Continue / Codex query the running analysis mid-generation, *before* a copy-paste happens). The `deslop` CLI is the cold-cache fallback for CI gates and one-shot audits. All three binaries are thin shells over one `deslop-core` library — the LSP and MCP sit in the agent's inner loop, the CLI re-uses the same engine for batch runs. Ranking is **worst offenders first** (highest weighted duplication impact at the top). Detection and ranking ship today; AI-assisted and mechanical deduplication actions are on the roadmap. Languages today: **C#, Rust, Python, and Dart**; TypeScript/JavaScript and Go are on the roadmap. Parsing is always tree-sitter — regex on source is prohibited.

## Prevention beats cure

The whole point of Deslop is that a duplicate never lands — see **Rule zero** at
the top of this file, which carries the `find-similar` LAW. Post-hoc scrubbing is
what every static analyzer already does; Deslop's edge is being live in the
agent's inner loop. Paste-ready recipe for other repos' `AGENTS.md` / `CLAUDE.md`:
see [docs/snippets/agents-md-recipe.md](docs/snippets/agents-md-recipe.md).

Full spec: [docs/specs/SPEC.md](docs/specs/SPEC.md). Execution plan and live TODO: [docs/plans/PLAN.md](docs/plans/PLAN.md).
- All spec sections have non-numeric, hierarchically structured IDs. All tests refer to spec IDs. All code refers to spec IDs.

**Primary language:** Rust
**Build command:** `make ci`
**Test command:** `make test`
**Lint command:** `make lint`

**There are 7 AgentPMO make targets; repo-specific targets must sit below a horizontal marker.** `make test` runs the test runner with its fail-fast flag, collects coverage, asserts measured ≥ threshold from `coverage-thresholds.json`, and exits non-zero on any failure. To debug a single test, invoke `cargo test <name> -- --nocapture` directly — that is not a Makefile target.

## UI

- The initial UI is a VSIX; IntelliJ and other plugins are in progress.
- Consistency across UI panels is critical.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/Deslop](https://github.com/Nimblesite/Deslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
