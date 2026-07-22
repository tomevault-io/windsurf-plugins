---
trigger: always_on
description: - `crates/`: the Rust workspace (~27 crates). Load-bearing ones:
---

# Repository Guidelines

## Project Structure & Module Organization
- `crates/`: the Rust workspace (~27 crates). Load-bearing ones:
  - `uni`: public API facade (`Uni` / `Session` / `Transaction`, forks, plugins host wiring) — also hosts the bulk of the integration tests under `crates/uni/tests/`.
  - `uni-store`: storage engine — Lance (L1) backend, L0 buffers, WAL, OCC/SSI commit machinery, snapshots, compaction.
  - `uni-query`: Cypher planner + DataFusion-based executor, plus the Locy fixpoint runtime (`df_graph/locy_*`).
  - `uni-cypher`: pest grammar, parser, AST (shared by Cypher and Locy front-ends).
  - `uni-locy`: Locy compiler front-end (typecheck, stratification, probabilistic semantics).
  - `uni-plugin*`: plugin framework — `uni-plugin` (kernel/traits), `-host`, `-builtin`, and the WASM/Extism/Rhai/PyO3 loaders.
  - `uni-algo`, `uni-crdt`, `uni-fork`, `uni-bulk`, `uni-cli`: graph algorithms, CRDT types, fork engine, bulk ingest, CLI.
  - `uni-tck` / `uni-locy-tck`: openCypher and Locy conformance suites (cucumber).
- `bindings/`: PyO3 Python wheels (`uni-db` + 5 feature variants sharing one source tree) and `uni-pydantic` (OGM).
- `website/docs/`: user-facing documentation (mkdocs).
- `docs/UNI_BLACK_BOOK.md`: the internals reference (architecture source of truth).
- `examples/`, `demos/`: runnable examples, plugin samples, demo walkthroughs.

## Build, Test, and Development Commands
- `cargo build`: compile the workspace.
- `cargo run -p uni-cli`: run the CLI binary.
- `cargo nextest run`: run all tests (parallel, preferred).
- `cargo nextest run -E 'test(<name>)'`: run a specific test by name.
- `cargo nextest run --run-ignored all`: include ignored/slow perf tests.
- `cargo bench`: run Criterion benchmarks.
- `cargo fmt`: format code.
- `cargo clippy`: lint for common issues.
- Use `cargo nextest` instead of `cargo test` for regular test runs.

## Coding Style & Naming Conventions
- Follow standard Rust style and format with `rustfmt`.
- `snake_case` for modules/functions, `CamelCase` for types/traits, `SCREAMING_SNAKE_CASE` for constants.
- Keep file names aligned with module paths (e.g., `crates/uni-store/src/runtime/wal.rs`).
- Add brief comments only when logic is not obvious.

## Testing Guidelines
- Unit tests live next to code in `crates/*/src/**` with `#[cfg(test)] mod tests`.
- Integration tests live in `crates/*/tests/` (mostly `crates/uni/tests/common/`) and often use `#[tokio::test]`.
- Benchmarks go in `crates/uni/benches/` using Criterion.
- If a test is flaky or sensitive to parallelism, document the constraint in a comment at the test site.
- For TCK compliance runs, use `scripts/run_tck_with_report.sh`.
- For filtered TCK subsets, use `scripts/run_tck_with_report.sh "~Match1"` (replace filter as needed).
- TCK run artifacts are written under `target/cucumber/` (results/report) and synced into `compliance_reports/` by mode.

## Commit & Pull Request Guidelines
- Use Conventional Commits as in history: `feat: ...`, `fix: ...`, `docs: ...`, `chore: ...`.
- PRs should include a short rationale, tests run, and links to related issues.
- Update `docs/UNI_BLACK_BOOK.md` when architecture or internals change, and `website/docs/` when user-facing behavior changes.

## Agent Git Safety Rule
- Do not perform any git action unless the user has explicitly instructed it in the current conversation turn.
- This includes all git commands and git-related workflows (for example: `status`, `diff`, `add`, `commit`, `push`, `pull`, `checkout`, `reset`, `merge`, `rebase`, `tag`, `stash`, `cherry-pick`).

## Fork system — current invariants (post Phase 7)

The fork track shipped phases 0 through 7. This block captures the
*current* contract an agent needs to honor when touching fork code.
The phase-specific blocks below preserve the rationale and gotchas
from each phase for deeper context.

- **Registry single-writer rule.** Every mutation to
  `catalog/fork_registry.json`, `catalog/fork_schemas/*`,
  `catalog/fork_tombstones/*`, and `catalog/forks/{id}/id_allocator.json`
  goes through `ForkRegistryHandle`. The 2PC state machine in
  `crates/uni-store/src/fork/registry.rs` is the only writer; never
  patch the JSON files directly.
- **Reads are routed through `BranchedBackend`.** A forked session's
  `Session::query` and `Session::tx` go through the fork's branched
  backend, which chains Lance `base_paths` from the fork's branch
  up to main. Never bypass it to read fork data — the branch
  resolution is what gives the fork its isolated view.
- **`UidIndex` is NOT branch-isolated.** The shared
  `indexes/uni_id_to_vid/{label}/index.lance` dataset accumulates
  entries from both primary and fork branches. Anything that
  resolves UID → primary VID must verify the candidate via
  `MATCH (n:Label) WHERE id(n) = $vid` on a primary session. The
  `resolve_primary_vid` helper in `crates/uni/src/api/fork_diff.rs`
  is the canonical implementation.
- **Diff identity is content UID, not VID.** Vertex pairing is by
  `compute_vertex_uid(label, ext_id, properties)`; edge pairing is
  by `(src_uid, dst_uid)` scoped to edge type. VIDs on `DiffVertex`
  are informational. Two unrelated forks with overlapping VIDs
  still pair correctly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/uni-db](https://github.com/rustic-ai/uni-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
