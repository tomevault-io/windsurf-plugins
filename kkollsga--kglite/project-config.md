---
trigger: always_on
description: uv venv .venv                # one-time environment creation
---

# KGLite — Claude Code Conventions

## Build & test

```bash
uv venv .venv                # one-time environment creation
uv run --no-sync maturin develop  # fast dev install when Python tests need current Rust code
make gate                    # fast format/static + docs-facts checkpoint
make lint                    # fast format/static lint (no build, metadata walk, or imports)
make test-mcp                # package-scoped tests; also test-core / test-cli
```

**The repo `.venv` is owned by `uv`.** For direct Python or maturin commands,
use `uv run --no-sync …`; do not activate the environment manually and do not
use bare `uv run`, which may sync dependencies and rebuild the editable project
before running the requested command. This repository does not track `uv.lock`:
provision with `uv venv` and install dependencies explicitly with `uv pip
install --python .venv/bin/python …`. Make targets already select `.venv`
themselves.

**Build the smallest touched surface.** This is a virtual workspace, so bare
`cargo build --lib` builds every library member—including `kglite-py`—and then
`maturin` recompiles a different `python-extension` feature/crate-type variant.
Do not use that pair as a generic gate. Select one path:

- Rust engine → `make test-core` or a narrower `cargo test -p kglite <filter>`.
- MCP server → `make test-mcp` or a narrower package test filter.
- CLI → `make test-cli` plus only matching interface tests.
- Python wrapper/core test that does not exercise bundled commands → run
  `uv run --no-sync maturin develop --no-default-features --features
  abi3,python-extension` directly; do not pre-build the workspace.
- MCP/CLI bridge or final packaged-contract gate → run the full default debug
  extension once with `uv run --no-sync maturin develop` (or `make dev`).

The default extension intentionally links the engine, CLI, and MCP server; its
MCP feature adds roughly 100 resolved packages. Do not pay that cost for a
Rust-only or narrow Python check. Build caches live on the internal disk by
standing setup (2026-07): `target` is a **symlink** to
`/Users/Shared/cargo-targets/KGLite` (repo-relative `target/...` paths keep
working), and `SCCACHE_DIR=/Users/Shared/sccache` is pinned in
`~/.cargo/config.toml [env]` because `$HOME` sits on the external USB volume.
Do not override `CARGO_TARGET_DIR`/`SCCACHE_DIR` per-plan or switch
target/profile paths mid-plan merely because a build is slow; if the symlink
is missing (fresh clone), recreate it before the first build. Cargo never
garbage-collects the target dir — `make prune-target` (size-gated
`cargo clean`, wired into the release skill) keeps it bounded. macOS
Gatekeeper adds a ~30 s first-run assessment to every freshly linked local
binary unless the invoking terminal is in Privacy & Security → Developer
Tools; a warm `cargo test` that stalls at ~0 % CPU on first execution is that
assessment, not a hung test.

`make test`, `make test-full`, and bare workspace `cargo test` are broad
diagnostics, not routine local gates. Run them only to investigate a failure
that crosses package boundaries; otherwise let GitHub CI parallelize them.

**Testing discipline (locked 2026-07-22).**

- **The installed extension must be a debug build during correctness work.** A
  release-built `kglite.abi3.so` silently disables the debug-only assertions
  (parser stack-depth behavior, the disk arena-guard protocol) that are the
  suite's real detectors — the 2026-07-21 audit found latent bugs those
  assertions catch immediately. After any release-profile build (benchmarks,
  release constants), rebuild debug (`uv run --no-sync maturin develop`)
  before the next test run.
- **Binary-backed suites resolve binaries through
  `tests/conftest.py::workspace_binary`** — newest of release/debug, and a
  skip-with-rebuild-command when the binary predates the root `Cargo.toml`.
  Never hard-code a profile path in a test; a stale release binary shadowing
  fresh code produces contract failures that reveal nothing.
- **Every Python test carries a 120 s hang ceiling** (`pytest-timeout`,
  configured in `pyproject.toml`; opt-in heavy markers are exempted in
  `tests/conftest.py::pytest_collection_modifyitems`). A test that hits the
  ceiling is a FAILED test — fix the hang; never raise the default, never
  wait out a stuck run. The default suite's slowest test is ~2 s, so the
  ceiling is pure hang detection.

**Dev-environment cleanliness — every file accumulation needs a gate.** Any
path the tooling writes outside git must have a bound and an owner: `target/`
→ `make prune-target` (40 GB size gate); regenerable artifacts and tool caches
(`.bench-current.json`, `docs/_build`, `.mypy_cache`, `.ruff_cache`,
`.pytest_cache`, `.uv-cache`, stale ABI-variant extensions, `.DS_Store`) →
`make prune-dev` (wired into the release skill); sccache → its 30 GiB config
cap; `dev-docs/` and `inbox/` → their skills. Never add a new file-writing
step (bench capture, fixture dump, scratch graph) without pointing it at the
session scratchpad / `tmp_path`, or adding it to `prune-dev` in the same
change. `.hypothesis/` is deliberately exempt — it is the found-counterexample
regression corpus, not a cache.

**Local correctness testing stays in the default/debug profile.** Never run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkollsga/kglite](https://github.com/kkollsga/kglite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
