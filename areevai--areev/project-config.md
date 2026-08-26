---
trigger: always_on
description: Embedded memory engine for AI agents — reference implementation of OMS (Open
---

# Areev

Embedded memory engine for AI agents — reference implementation of OMS (Open
Memory Spec). Rust workspace of 16 crates (plus `areev-js`, a standalone napi
package built outside the workspace). Memories
are immutable content-addressed grains in per-file Turso databases, queried
with CAL, and rendered into model-ready context in-process (no server in the
recall path).

The framework adapters (`areev-langgraph`, `areev-crewai`) **left this repo**
on 2026-08-23 for `AreevAI/areev-adapters` — they depend only on the
published PyPI `areev`, so they version against their upstream frameworks,
not against the core. That repo is currently **private and parked** (1.0.0
of both stays on PyPI and works); un-parking is its README's checklist. Do
not re-add adapter code here — a change that breaks them surfaces in that
repo's CI, and re-adding a gate for it is part of un-parking.

**Status**: published — the library crates + the `areev` binary on crates.io,
`areev` on PyPI, and the Node binding on npm as **`@areev/areev`** (+ scoped
per-platform addons). Unscoped `areev` on npm is still blocked: npm's
similarity filter 403s it against `argv` (`Package name too similar to
existing package argv`), unresolved as of the 1.5.2 release attempt
(2026-08-22) despite an earlier belief the exception had been granted —
`areev-win32-x64-msvc`'s separate *security hold* was released that day, but
the *main package's* filter block was not, and the two got conflated here
once already. A support ticket for the unscoped name is open; until it
lands, npm releases publish under the pre-1.5.1 `@areev/*` scope (never the
bare `areev-<platform>` names, which 403 the same way). crates.io and PyPI
are unaffected — different registries, no naming block there.
`areev-py`, `areev-bench`, and `areev-conformance` stay `publish = false`;
`areev-js` ships to npm, not crates.io.
The version lives in `[workspace.package]` in the root `Cargo.toml` (all crates
inherit it) and `CHANGELOG.md` records each release — don't restate the number
here (it goes stale). `ARCHITECTURE.md` is the design source of truth — the
architecture and the numbered design decisions; `CHANGELOG.md` summarizes what
exists; `crates/areev-bench/RESULTS.md` has the benchmark numbers.

## Commands

```bash
cargo test --workspace            # full suite (2,100+ tests, ~1 min)
cargo test -p areev-cal          # per-crate
cargo run --release -p areev-store --example bench       # latency gates
cargo run --release -p areev-store --example voice_loop  # 50ms-cadence gate
cargo run -p areev -- recall --db demo.db --ns caller --subject john

# does the loop actually make an agent better? (crates/areev-bench/SELFIMPROVE.md)
cargo run -p areev-bench --bin selfimprove_aba -- --workdir /tmp/aba --mock --assert-shape
#   ^ keyless CI floor: proves the A/B/A/B plumbing, never a learning claim.
#     Live numbers need --agent-cmd/--llm-cmd (see SELFIMPROVE.md "Reproduce").

python3 scripts/check_versions.py   # all five version sites agree
python3 scripts/repo_stats.py       # regenerate the README quality metrics
python3 scripts/repo_stats.py --check   # what CI asserts (2% drift tolerance)
cargo llvm-cov --workspace --lcov --output-path lcov.info   # then:
python3 scripts/coverage.py --lcov lcov.info             # regenerate docs/coverage.json
python3 scripts/coverage.py --lcov lcov.info --check      # what CI asserts (2-point tolerance)

scripts/build_demo.sh               # rebuild data/demo.db (the README's demo memory)
node scripts/shoot_console.mjs http://127.0.0.1:7461 demo/screens   # re-shoot its screenshots

docker build -t areev .             # the container image (postgres+tls features on) — docs/docker.md
```

- **Do not run blanket `cargo fmt`** — the tree is not uniformly rustfmt-clean
  (~177 files differ). Match surrounding style; format only
  the lines you touch.
- If CLI/MCP smoke tests fail with "spawn areev: No such file or directory":
  the cached test binary has a stale absolute path baked in via
  `CARGO_BIN_EXE_areev` (happens after the repo folder moves/renames).
  Fix: `touch crates/areev-cli/tests/*.rs` and re-run.
- CI (`.github/workflows/ci.yml`): test on ubuntu/macos/windows, clippy
  (`-D warnings`), MSRV build, `cargo doc`, coverage, Python (maturin + pytest),
  and Node (napi build + `node --test`), plus `versions` (every version site
  agrees) and `stats` (the README quality figures still match the tree).
  `security.yml` runs `cargo deny`. Still run tests locally before pushing.
- **The README quotes generated numbers.** `scripts/repo_stats.py` emits
  `docs/repo-stats.{json,md,html}` and four SVGs the README embeds (the
  quality chart + the per-crate coverage chart, light/dark each); the
  `stats` job fails the build if they drift >2% from the tree. Line coverage is
  a separate input: `scripts/coverage.py` turns the `coverage` job's LCOV trace
  into `docs/coverage.json`, which `repo_stats.py` renders into the same charts.
  The README's latency chart is separate: `scripts/bench_chart.py` renders
  `docs/assets/bench-latency-{light,dark}.svg` from numbers quoted out of
  `crates/areev-bench/RESULTS.md` — re-run it by hand when RESULTS.md changes
  (measurements are not tree-derivable, so it is not CI-checked).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AreevAI/areev](https://github.com/AreevAI/areev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
