---
trigger: always_on
description: ry is a language tool for R, built as a language server plus CLI. It aims to be world class at three things: code analysis on the level of rust-analyzer — with a static type checker at its core — plus code formatting and linting.
---

# Overview

ry is a language tool for R, built as a language server plus CLI. It aims to be world class at three things: code analysis on the level of rust-analyzer — with a static type checker at its core — plus code formatting and linting.

The type checker is central: no static type checker exists for R, so ry defines its own typing semantics (the contract lives in the typing reference at `docs/src/content/docs/reference/type-system.md`). Because R itself has no type-annotation syntax, annotations are written in `#:` comments using a JSDoc-like notation, which keeps annotated code fully compatible with ordinary R tooling.

Crates:

- `crates/` — the shipping product: `syntax` (lexer/parser, lossless rowan trees), `semantics` (the salsa-based analysis core and type checker), `format` (the formatter, syntax-only), `ide` (editor features as pure reads), `ry` (LSP server + CLI), and `repl` (the R console behind `ry repl` and `ry run` — runtime-loaded R, so the rest of the workspace stays R-less)
- `legacy/` — the frozen previous implementation (`analysis-legacy`, `engine-legacy`, `ry-legacy`, its `fixtures` harness) and `differential`, now ONLY the cross-stack benchmark harness (the identity-parity program is complete and retired by user decision — the new stack's fixtures are the contract; no change needs oracle agreement); everything lives here because every dependency edge points at the oracle, so the eventual legacy deletion sweep is one directory removal (its new-stack-only perf witnesses migrate out first)

The project is built by AI agents driving development, with light human steering. Agents keep two written homes current: the docs site (`docs/`) holds the authoritative, user- and contributor-facing specs (they are contracts — mandatory to keep accurate), and `.agents/memory/MEMORY.md` is the agent knowledge base (engineering state, priorities, debt, and non-obvious design rationale, so they are not rediscovered). Update both in the same session as the work that changes them.

# Goals

- Deliver high-quality diagnostics for R in the style of Rust and Elm: clear, precise, actionable wording; avoid overly internal or theory-heavy language when user-facing wording would be clearer; prefer precise source ranges over coarse fallback ranges.
- Provide full editor tooling — hover, completion, goto-definition, references, rename, inlay hints — and preserve the semantic information those features need whenever practical.
- Provide first-class formatting and linting alongside analysis.
- Scale to very large code bases, including more than 300,000 LoC; performance matters.

# Ownership mandate

The user has delegated full technical ownership to the agents: empty the backlog and bring the project to the best possible state — rust-analyzer quality. That explicitly covers code structure, crate boundaries, naming, performance, pipeline architecture, semantic correctness, and judged deduplication. Do not optimize for "safe, risk-free" minimal diffs; bring code to its intended shape, including large refactors, and take responsibility for the outcome. Design decisions that previously required a user check-in are now the agent's to make: decide, implement, and record the decision and rationale in `.agents/memory/decisions.md` (or the docs page it belongs to) in the same session. Two standing constraints: work directly on `main` (user directive), and do not open new pull requests.

# Do not think like a human (user directive)

Human engineering instincts — de-risking, staging, keeping diffs small and reviewable, avoiding "scary" rewrites — exist because a human's time is scarce and starting over is expensive for them. Neither is true for an agent, so those instincts pick the wrong strategy here:

- **Go directly to the intended end shape in one change**, however large and invasive. Break the whole codebase mid-change if the target design calls for it, then fix everything — compiler errors, warnings, tests — afterwards in one sweep. Do not sequence a redesign into small incremental steps to "manage risk"; that trades the right design for ceremony.
- **Never propose or choose a watered-down variant of a design because the full version is a big change.** If the full version is right, implement the full version. Starting over after a failed attempt is cheap; shipping the wrong shape is not.
- **File size is not a problem.** Do not split, reorganize, or flag a file merely for being large (the LSP server module is fine as one file). Split only when a genuine new logical component exists.
- Correctness gates are unchanged: the fixture suites, witnesses, clippy, and fmt must be green before a change lands — the point is to reach green in one big pass, not to shrink the change.

# Incremental analysis


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felix-andreas/ry](https://github.com/felix-andreas/ry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
