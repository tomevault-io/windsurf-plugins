---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`dora-rs/dora-hub` is the **collection of reusable [dora](https://github.com/dora-rs/dora) nodes** — ~60 ready-to-use sensors, models, transforms, and sinks you can drop into a dataflow. Nodes are independently packaged and published (Python to PyPI, Rust to crates.io); a dataflow pulls one in via `pip install dora-<name>` / a `git:` source / or a `hub:` reference.

This repo also hosts the **Dora Hub catalog** (`node-index/`) — the git-backed index that the `dora hub` CLI resolves `hub:` references against (spec: [`docs/plan-node-hub.md`](https://github.com/dora-rs/dora/blob/main/docs/plan-node-hub.md) in `dora-rs/dora`).

> **Note:** the `node-index/` catalog, the `index-ci/` Rust gate, and the `node-index CI` workflow are the Dora Hub catalog enforcement. `make index-ci` is a no-op in a tree without them.

Two languages, one repo:
- **Python nodes** (~55) — `pyproject.toml` + a package dir, built/tested with **uv**, linted with **ruff**, tested with **pytest**.
- **Rust nodes** (~14) — members of the root **Cargo workspace**; some are Rust+Python (maturin) hybrids.

## Repository Layout

| Path | What |
|------|------|
| `node-hub/<name>/` | One node per directory — the active, maintained collection |
| `node-archive/` | Deprecated nodes, not built or tested in CI |
| `node-index/` | The Dora Hub catalog (`<ns>/<name>/<version>.yml`); see `node-index/README.md` |
| `examples/` | End-to-end dataflows exercised by CI (`examples/*/dataflow.yml`) |
| `tests/` | Workspace-level Rust integration tests |
| `benches/` | Benchmarks |
| `index-ci/` | Rust crate (`dora-index-ci`) — the node-index validate / append-only / namespace / auto-merge gate |
| `scripts/` | Repo tooling (`test-node.sh`) |
| `.github/workflows/` | CI (see below) |

### Anatomy of a node

**Python node** (`node-hub/dora-echo/`):
```
dora-echo/
  pyproject.toml          # name = "dora-echo"; [project.scripts] dora-echo = "dora_echo.main:main"
  dora_echo/__init__.py   # package dir (underscores)
  dora_echo/main.py       # entrypoint
  tests/test_dora_echo.py # pytest
  README.md
```
**Packaging convention:** the PyPI dist name, the `[project.scripts]` console-script, and the value a dataflow uses as `path:` are **all the same string** (`dora-echo`). New nodes must keep this 1:1:1 mapping. Lint config lives under `[tool.ruff.lint]` — see the shared baseline in the root `ruff.toml`.

**Rust node** (`node-hub/dora-rerun/`): a `Cargo.toml` workspace member (add it to the root `Cargo.toml` `members`). Rust+Python hybrids also carry a `pyproject.toml` and build a wheel with maturin. **Hub-spawnability:** a Rust node must take its node id from the daemon — use `DoraNode::init_flexible(NodeId::from(...))` (or `init_from_env()`), never a hard-coded `init_from_node_id(...)`, or `hub:` only works when the dataflow names the node exactly that id. (Python `Node()` reads `DORA_NODE_CONFIG` already, so it's fine.)

### Node README checklist

Every node must ship a `README.md`. `dora-node.yml` is the machine contract; the README is the human one — keep them consistent (same description, same example). Use this structure (omit a section only if truly N/A; see `node-hub/terminal-print/README.md` for the reference):

- [ ] **Title + one-line description** — matches `dora-node.yml`'s `description`.
- [ ] **Behavior** — what the node actually does (the logic), not just what it is.
- [ ] **Inputs** — each input id + type. **Declare every input a dataflow will wire**: a `hub:` build fails on any wired input not in the manifest (an empty map is *not* a wildcard). A generic sink that prints "any input" must still declare a concrete input (e.g. `data`) — don't leave `inputs` empty.
- [ ] **Outputs** — each output id + type, or "None" for sinks.
- [ ] **Environment variables** — name, type, default, meaning (mirror `dora-node.yml`'s `env`), or "None". Document only vars that **actually affect behavior**: a var the code reads with `os.getenv` but never uses is *not* part of the contract — leave it out of the manifest/README (don't imply it does something). A manifest `default:` is **documentation, not injected** — give one only when the code itself has an `os.getenv("X", DEFAULT)`; a *required* var (code errors if unset) gets no `default:` and must appear in the example `env:`.
- [ ] **Naming constraints** (manifest validation): port ids can't contain `/`; env names can't be the reserved `PATH`/`PYTHONPATH`/`LD_*`/`DYLD_*`. If a node uses one (e.g. a topic `v1/chat/completions` or an env `PATH`), rename it in the node code (keeping any unrelated HTTP route/path) so the manifest is valid.
- [ ] **Usage** — a copy-pasteable dataflow YAML snippet wiring the node. Prefer the `hub:` form. If you show a from-source `path:`, it is the built **executable** (the manifest `entrypoint`, e.g. `target/release/<bin>`), paired with `build:` — never the package directory.
- [ ] **Build** — for workspace-member Rust nodes, note `cargo build --release --target-dir target` (package-local binary, matches `entrypoint`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dora-rs/dora-hub](https://github.com/dora-rs/dora-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
