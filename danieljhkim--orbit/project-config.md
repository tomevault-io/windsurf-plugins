---
trigger: always_on
description: Project instructions for agents working on Orbit.
---

# CLAUDE.md

Project instructions for agents working on Orbit.

## Rules

- **Don't commit** until the Orbit task has been explicitly approved by the human.
- **Don't invent task IDs** — get them from `orbit.task.add`. Don't edit task files directly — use `orbit.task.update`.
- **Don't add cross-crate dependencies** without checking [`ARCHITECTURE.md`](ARCHITECTURE.md). If a new edge is genuinely needed, file a task and an ADR before adding it.

## Branching

- **`main`** is the release / production branch — only release merges and hotfixes land here. Default base for external install URLs, npm/Homebrew consumers, and the GitHub default-branch view.
- **`agent-main`** is the dev integration branch — every task PR targets `agent-main`.
- **Promotion**: each release tags on `agent-main`, then merges `agent-main → main` via a merge commit. See [`RELEASING.md`](RELEASING.md) §10b.
- **Hotfixes** branch from `main`, merge to `main`, tag a patch release on `main`, then back-merge `main → agent-main` in the same session. See [`RELEASING.md`](RELEASING.md) §Hotfix flow.

## Build / Lint

`make ci-fast` (fmt-check + guardrail scripts; no compile) must pass before a task moves to `review`. The full `make ci` is the canonical merge gate via [`.github/workflows/ci.yml`](.github/workflows/ci.yml) on every PR — don't run it per task locally.

## Agent Read Exclusions

Team-wide `Read()` exclusions (build artifacts, generated graph data, runtime state) live in [`.claude/settings.json`](.claude/settings.json) under `permissions.deny`. If you work on the excluded code itself (e.g. the graph builder under `.codegraph/`, or benchmark harness output), override locally in `.claude/settings.local.json` with a matching `allow` rule — don't relax the committed list.

## Architecture

Crate layering, per-crate responsibilities, and scoping rules live in [`ARCHITECTURE.md`](ARCHITECTURE.md). Read it before adding a new crate, a new dependency edge, or a new persisted artifact.

Reusable codebase-specific patterns (Command, RAII guard, newtype, crate-boundary error translation) live in [`docs/design-patterns/`](docs/design-patterns/). When you reach for one of those shapes, copy from the documented reference instead of inventing a new one.

## Code Navigation

This repo has two semantic graphs available (no live LSP). Use them in this order:

- **Definition / signature lookup** → `codegraph_search` then `codegraph_node`. Returns file:line, signature, and leading doc comment without a `Read`. **Avoid `orbit_graph_search` for plain symbol lookups** — slow on large repos, and you can build orbit-graph selectors by template from the codegraph result: `symbol:<file>#<name>:<kind>`. Reach for `orbit_graph_search` only when you need a method-on-impl selector, a `source_regex` search, or `include_non_code` doc/config matches.
- **Outbound calls (what does X call?)** → `codegraph_callees`. Expect duplicate edges per call site — dedupe mentally.
- **Find references / callers (who uses X?)** → **`orbit_graph_refs`** with `include: "all"`, *not* `*_callers`. Both graphs' `callers` indexes miss cross-crate calls that go through `pub use` re-exports (e.g. a symbol defined in `orbit-common`, re-exported from `orbit-core`, called in another crate), so they routinely return empty for real public functions. `orbit_graph_refs` surfaces the actual call sites plus re-export points.
- **Blast radius before edits** → `codegraph_impact`.
- **Ground-truth fallback** → `rg --type rust 'symbol_name'`. Use when `refs` looks incomplete or you need to see exact textual context (macro call sites, doc references, etc.).
- **From a plain shell (no MCP/codegraph)** → the same orbit-graph queries are bundled in the main `orbit` binary as `orbit graph <sub>` (`search`/`show`/`refs`/`callees`/`impact`/`deps`/`trace`/`overview`/`implementors`, plus `sync`). In-process the tools above are faster — reach for the CLI only when the graph tools aren't available.

## Design Docs

- **Layout.** Feature design docs live under `docs/design/<feature>/`. Folder layout, required sections, ADR format, and glossary shape are documented in [`docs/design/CONVENTIONS.md`](docs/design/CONVENTIONS.md). Use the `orbit-docs` skill / `orbit docs` surface to retrieve indexed docs.
- **Same-PR updates.** Change the doc in the same PR as the code: flip affected ADR statuses (`Proposed → Accepted` with task ID), bump `**Last updated:**`, add a new ADR for any non-obvious decision the change embodies. Stale docs are a review blocker.

## Rust Practices

Lint-enforced rules (full set in `[workspace.lints]`; key implications below):

- **No `unwrap()` / `expect()` at crate boundaries.** Propagate `OrbitError`; use `expect("<invariant>")` only when the invariant is local and documented. See [`docs/design-patterns/error_translation.md`](docs/design-patterns/error_translation.md).
- **No `print!` / `eprint!`.** Use `tracing` with structured fields (`tracing::info!(run_id, ...)`), not string interpolation. Allowlisted only for genuine CLI/example user output.
- **No lock guards across `.await`.** Scope `std::sync::Mutex` / `RwLock` to a block, or use `tokio::sync` for cross-task state.

Conventions (not lint-enforced):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danieljhkim/orbit](https://github.com/danieljhkim/orbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
