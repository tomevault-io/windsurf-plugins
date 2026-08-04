---
trigger: always_on
description: Pure-Rust port of Lua that runs **5.1, 5.2, 5.3, 5.4, and 5.5** from one core,
---

# omniLua — agent guide

Pure-Rust port of Lua that runs **5.1, 5.2, 5.3, 5.4, and 5.5** from one core,
selected per instance. No C dependency. Ships to crates.io + npm, runs the stock
LuaRocks client, embeds in Rust programs and in the browser (`wasm32`). This file
is the operational entry point — read it first, then the subsystem guide for
wherever you're working (`crates/<x>/CLAUDE.md`, `harness/CLAUDE.md`).

The public artifact is **omniLua**: embedding crate `omnilua` (dir
`crates/lua-rs-runtime/` unchanged), CLI crate `omnilua-cli` producing the
`omnilua` binary, npm package `omnilua`. The version env var is
`OMNILUA_VERSION` (canonical), with `LUA_RS_VERSION` still read as a fallback.
The local directory `lua-rs-port/` and internal crate names (`lua-vm`, `lua-gc`,
…) are unchanged — they are implementation paths, not public surfaces.

This repo is one of three siblings under `../`; see **`../CLAUDE.md`** for the
tree-level story (the AI-agent harness is the real product). Cross-repo
"who's-doing-what" lives in **`../AGENT_COORDINATION_BOARD.md`**.

## Orientation — find current truth, don't trust prose

- **Backlog → GitHub issues** (`gh issue list`). No doc enumerates "what's left."
- **Test status → run the harness** (`harness/run_official_all.sh`). Never trust a
  hardcoded pass-count written in a doc — they rot.
- **Version → `CHANGELOG.md` / `git tag`.** Don't hardcode it anywhere.
- Active frontier: generational GC (issues #104, #113) and extracting the harness
  to `../port-harness` + `../redis-rs-port`.

## The one rule: the oracle is the only truth-teller

A change that builds but no oracle has spoken on is **unverified**. Build success
is not signal. The oracle is the unmodified upstream reference binary
(`reference/lua-5.4.7/`, `reference/lua-5.3.6/`, and `/tmp/lua-refs/bin/lua5.x`).
The full methodology for version work is **`specs/MULTIVERSION_PLAYBOOK.md`** —
read it before touching version-specific behavior.

## How to work with this repo

### Build & run
```bash
cargo build -p omnilua-cli -q         # debug CLI → target/debug/omnilua
target/debug/omnilua script.lua       # run a file
target/debug/omnilua -e 'print(1+2)'  # one-liner
target/debug/omnilua                  # REPL (no args)
OMNILUA_VERSION=5.1 target/debug/omnilua script.lua  # pick a version (5.1–5.5; default 5.4)
```

### The iteration ladder — climb only as far as the question forces
| Tier | Command | Answers |
|---|---|---|
| 1 | `cargo build -p <crate> -q` | does it compile? |
| 2 | `cargo test -p lua-rs-runtime --test multiversion_oracle` | behavior vs the baked oracle constants — **the inner loop** |
| 3 | `specs/oracle/diff_one.sh <ver> "<snippet>"` | one specific divergence vs the live reference binary |
| 4 | `harness/canaries/gc/run_canaries.sh` | a GC / metamethod / table change didn't break the collector |
| 5 | `harness/run_official_test.sh reference/lua-c/testes/<t>.lua` | one real program (5.4) |
| 6 | `harness/run_official_all.sh` + `cargo test --workspace` + `specs/oracle/check.sh` ×5 | the PR gate |

A GC-lifecycle, heap-guard, VM-construction, or embedding-entry-point change
also runs **`harness/strict_guard_check.sh`**. The GC's three no-active-heap
guard checks (detached allocation, sweep-blind weak handle, dropped pacer
charge) now panic **unconditionally in every build** — the never-freed dual of
`LUA_RS_GC_QUARANTINE`'s freed-too-early — so any guard-coverage violation
self-reports with a backtrace under the normal test suites; the script is just
the convenience runner for the whole workspace. The embedding leak canaries
(`crates/lua-rs-runtime/tests/leak_canaries.rs`, a counting global allocator
asserting net-zero live bytes across VM/chunk/coroutine/callback churn) run
with the normal workspace tests.

Start one rung lower than feels right; if the cheap rung is silent, that's your
answer. Per-version detail: `specs/MULTIVERSION_PLAYBOOK.md §3`.

### Multi-version
One core, version chosen at runtime (`Lua::new_versioned` / `LUA_RS_VERSION`). The
hot bytecode dispatch loop is **version-free** — resolve the version **once** in a
cold path (e.g. `lua-vm`'s `legacy_for` flag) and never branch per-opcode.
**Version-gated compat code is load-bearing; do not "simplify" it away.** Seam map
and per-version cheat-sheet: `specs/MULTIVERSION_PLAYBOOK.md §5–6`. To debug a
non-5.4 divergence, diff the snippet against that version's reference:
`specs/oracle/diff_one.sh 5.3 "<snippet>"`.

**Reference binaries.** `reference/lua-5.4.7/` (port baseline) and
`reference/lua-5.3.6/` are vendored in-repo — source committed, binaries
gitignored, build with `make macosx -C reference/lua-5.x`. The multi-version
oracle scripts use `/tmp/lua-refs/bin/lua5.x` (all five versions), pinned in
`specs/oracle/CONTRACT.md`; rebuild them from there if `/tmp` was cleared.

### Benchmarks
`harness/bench/` measures the **omniLua / reference-C ratio** (wall + RSS), not
absolute throughput — the ratio is the only fair number. Any perf **claim**
must follow **`docs/MEASUREMENT_PROTOCOL.md`** (frozen-baseline interleaved
A/B, Ir/branch-sim arbiters, drop-if-neutral) — wall time alone does not
attribute on this rig.
```bash
bash harness/bench/compare.sh                              # all workloads, best-of-5

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianm199/omnilua](https://github.com/ianm199/omnilua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
