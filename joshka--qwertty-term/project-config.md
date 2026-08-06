---
trigger: always_on
description: qwertty-term is a full Rust rewrite of [Ghostty](https://ghostty.org): a native macOS
---

# Agent Guidance

qwertty-term is a full Rust rewrite of [Ghostty](https://ghostty.org): a native macOS
terminal emulator plus a family of reusable, embeddable crates. It is a byte-faithful port
of Ghostty's Zig (pinned at commit `77190bd02`, source at `~/local/ghostty`), verified by
differential testing against the original. (The pin was bumped from `2da015cd6` →
`77190bd02` on 2026-07-15 to adopt the scroll-region optimizations; the 14-commit delta is
mostly perf commits already mirrored as new work, plus the scroll-region `no_scrollback`
semantics. Per-file provenance comments that cite `2da015cd6` record where that code was
*originally* ported from and are left as historical notes; the differential **oracle** —
the authority — is now built at `77190bd02`. The three font/sprite cursor-height commits in
the delta are tracked for separate verification in `docs/threads/status/issues.md`.)

**Start here:** `docs/rewrite-prompt.md` (the constitution), then `docs/threads/README.md`
(the parallel-thread model + PR/gate/status protocol), then `docs/handoff.md` /
`docs/port-status.md` / `docs/feature-coverage.md` for current state. Per-subsystem analysis
lives in `docs/analysis/` (commit-stamped) and decisions in `docs/adr/`.

## Repo layout and version control

- Version control is **jj** (colocated git). The repo root (`~/local/ghostty-rs`) is a bare
  store — it holds `.git`, `.jj`, `work/`, and an `AGENTS.md`, but **no checkout** and is NOT
  a jj workspace. Never run jj/git/cargo at the root (it re-creates a phantom workspace that
  snapshots everything — see the root `AGENTS.md`).
- All work happens in per-workspace checkouts under `work/<id>`. Create one from an existing
  checkout: `cd work/josh && jj workspace add ../<id> --name <id> --revision main`. One
  writer per checkout; stay in your own, never touch a sibling's.
- **jj discipline** (full text in `docs/threads/README.md`): run `jj st` after every edit
  burst to snapshot (the unsafe window is between "files edited" and "next jj command";
  `cargo`/`npx`/`git` do NOT snapshot). If the working copy goes stale, just `jj workspace
  update-stale` — it snapshots first, nothing is lost; recover a "vanished" edit via `jj op
  log` → `jj restore`. Never fall back to git plumbing or scratchpad copies. Verify commits
  are non-empty after `jj describe`.
- **Ship via the PR pipeline** (`docs/threads/README.md`): `jj new 'trunk()'` to start on
  current main, `jj describe` → push a bookmark (`jj git push --bookmark <id>/<feature>`) →
  `gh pr create` → merge (which advances `main`). Small doc-only changes may land direct to
  main. `trunk()` (== `main`) is the integration point; keep it green.

## Commands

```bash
# Build / run the app — macOS 13+ (Metal); the crate/binary is `qwertty-term`
cargo run -p qwertty-term --release
cargo run -p frame-capture -- --help        # headless VT-bytes → PNG (the embeddability demo)

# Test
cargo test --workspace
cargo test -p qwertty-term-vt <name>         # a single test by name substring
cargo test -p qwertty-term-vt --test <file>  # a single integration-test file (crates/*/tests/)
cargo test -p qwertty-term-vt --release --all-targets   # RELEASE LANE — never skip (see below)
cargo test -p qwertty-term-vt --release --lib --features slow_runtime_safety  # paranoid lane (ADR 0001)

# Differential parity vs the Ghostty reference oracle
cargo test -p vt-diff                        # curated corpus (no Zig artifact needed)
cargo test -p vt-diff --features reference   # vs libghostty-vt (build the ref lib first — bottom)

# Lint / format (repo markdownlint config: 100 cols, aligned tables, code-fence languages)
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
npx markdownlint-cli2 "**/*.md"              # scripts/align_md_tables.py fixes aligned tables

# App smokes — macOS, drive real Metal/windows; NOT in CI (run locally)
cargo run -p qwertty-term --release -- --offscreen-smoke
QWERTTY_TERM_SMOKE_SPLITS=1 cargo run -p qwertty-term --release   # also GEOMETRY/SEARCH/KEYBIND/FOCUS/MOUSE/BELL/…

# Fuzz (nightly, standalone workspace at crates/qwertty-term-vt/fuzz)
cargo +nightly fuzz run parser -- -max_total_time=180
cargo +nightly fuzz run resize -- -max_total_time=180   # resize-interleaved — catches the class CI misses

# Codegen (checked-in generated tables)
cargo xtask gen-unicode            # UCD tables — exact parity with Ghostty's generated table
cargo xtask gen-nerd-constraints   # nerd-font per-icon sizing table

# Benchmarks
scripts/bench-vtebench.sh [--terminal ghostty --app-path <bundle>]   # + docs/benchmarks/
scripts/bench-doomfire.sh

# Build the Ghostty reference lib (for `vt-diff --features reference`)
cd ~/local/ghostty && mise exec zig@0.15.2 -- zig build -Demit-lib-vt=true -Doptimize=ReleaseFast
```

**The gate** (must pass before `jj describe`/PR): `cargo check --workspace --all-targets`
(zero warnings) · `cargo test --workspace` · the **release lane** · `fmt` · `clippy -D
warnings`. Additionally: `vt-diff --features reference` + a new corpus case when engine
**semantics** change; the offscreen + relevant `QWERTTY_TERM_SMOKE_*` smokes when the app
changes; `markdownlint` when docs change; before/after numbers when perf changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshka/qwertty-term](https://github.com/joshka/qwertty-term) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
