---
trigger: always_on
description: Convergence assistant for knowledge corpora.
---

# anneal

Convergence assistant for knowledge corpora.

`anneal` reads a directory of markdown files, computes a typed knowledge graph, checks it for local consistency, and tracks convergence over time. It is built for disconnected intelligences — agents across sessions with no shared memory — to orient in a shared body of knowledge and push it toward settledness.

## Project Context

- anneal's own specs are tracked as markdown in this repository. Inspect and maintain them with `anneal`: `anneal status`, `anneal check`, `anneal handle <file> --impact`.
- Design/spec docs under `.design/` carry the date in the H1 title as `# <Title> — YYYY-MM-DD`, matching the dated filename / frontmatter `date:`.
- The v2.0 master spec is the authoritative Programmable Corpus Runtime reference.
- The v1.x spec is superseded but retained as historical record of the shipped shape.
- The engine-viability artifacts gate v2.0 architecture decisions. Do not casually claim "SP-R1 cleared" — Ascent unsafe is accepted as bounded dependency risk, not eliminated.
- Repo-local anneal state is optional and should only be used when explicitly configured.
- `.planning/ROADMAP.md` and `.planning/STATE.md` own the roadmap spine and current state. Treat `bd` as the source of truth for in-progress work; `.planning/STATE.md` is updated at phase boundaries.
- `install.sh` ships with releases — treat installer correctness as release-critical.
- `.beads/config.yaml` is tracked repo config only; keep machine-specific federation settings local to your shell environment.
- Store state in specs sparingly. Use `bd` for state.
- On the maintainer's machine, `main` runs as `anneal-dev` beside release `anneal`, sharing `~/.config/anneal/config.toml` + history state. Keep config/history formats backward-compatible or migrate, else the release binary breaks.

For orientation in the v2.0 master spec (`2026-05-13-corpus-runtime.md`):
- Part I (`§1-§3`) for framing and the cold-agent test
- Part II (`§4-§8`) for architecture (substrate / adapters / surfaces, Source trait, engine-replaceability)
- Part III (`§9-§16`) for substrate primitives (identity, stored relations, engine-derived predicates, trails, capabilities)
- Part IV (`§17-§20`) for the language (grammar, types, aggregation, stratification)
- Part VII (`§29-§37`) for CLI + MCP surfaces
- Part X (`§47-§48`) for files and layout
- Part XV for `CR-*` label conventions (CR-D decisions, CR-R rules, CR-Q questions, CR-Fw forward-looking)

CR-* labels are referenced from bd issues and commit messages — search the spec for the exact label to find the governing definition.

For low-context corpus orientation, prefer:
- `anneal context "<goal>"`
- `anneal status`
- `anneal search "<text>" --limit 25`
- `anneal read <handle> --budget 4000`
- `anneal handle <handle> --impact`

Use `anneal context "X"` for "find the section that defines X" work,
`grep -rn "X"` for exhaustive literal occurrences with line numbers, and
`anneal -e '? ...'` for structural graph predicates.

Avoid broad default dumps like raw `check --json`, empty search queries, or full graph queries unless you are intentionally expanding with `schema`, `describe`, and narrow `anneal -e` predicates.

## Rust Toolchain

- `rust-toolchain.toml` pins Rust 1.94.0 with `rustfmt` and `clippy`.
- Use `just`. `just check` is the default gate (fmt + `install.sh` syntax + clippy + test, each step timed). Inspect `justfile` or run `just --list` for the full command surface.
- `just build` for a release binary; `just release-verify` for release-readiness gates.
- `just audit` = architecture fitness functions: `cargo-machete` (unused deps) + `cargo-deny` (advisories/bans/licenses/sources, configured in `deny.toml`). `just check` runs the offline subset (machete + deny bans/licenses/sources), guarded to skip if the tools aren't installed.
- Do NOT run `just check`/the test suite inside a git worktree (bug anneal-re9h): a git-fixture test writes `core.bare=true` into the shared `.git/config` and bricks git repo-wide. Recover with `git config core.bare false`.
- Add dependencies with `cargo add`; never hand-write version strings.
- `ast-grep run -p 'PATTERN' -l rust` for AST-aware code search (no config needed). Useful patterns: `$X.unwrap()`, `todo!($$$)`, `#[allow($$$)]`. Add `-r 'REPLACEMENT'` for structural refactoring; `--json` for machine-readable output.

## Module Boundaries

**v2.0 target shape (Phase 1 in flight — `bd anneal-xu2`):** a cargo workspace.

- `crates/anneal-core` — substrate: Datalog runtime, dynamic IR, engine-derived primitives (Ascent-backed), convergence stdlib, provenance, trail capture. No source-specific code.
- `crates/anneal-md` — markdown adapter; implements `Source`.
- `crates/anneal-cli` — the binary; links core + md.
- `crates/anneal-mcp` — MCP server; links core + md.
- Adapters beyond markdown (`anneal-mdx`, `anneal-code`, `anneal-host`) are sibling crates added v2.1+.

`anneal-core` is the only crate other anneal crates depend on. Engine choice is internal to `anneal-core` — see master spec §8 final paragraph. Do not expose Ascent as a general runtime `.dl` loader; the dynamic IR owns prelude/project/inline rules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flowerornament/anneal](https://github.com/flowerornament/anneal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
