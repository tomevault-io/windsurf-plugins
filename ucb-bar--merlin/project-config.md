---
trigger: always_on
description: 1. **Always use `./merlin`** — Never invoke `cmake`, `ninja`, `make`, or raw build commands directly. All build, compile, benchmark, CI, and setup operations go through `./merlin` (the thin wrapper at the repo root that runs `tools/merlin.py` inside the `merlin-dev` conda env). See `docs/reference/cli.md` for the full CLI reference.
---

# Merlin — Agent Instructions

## Golden Rules

1. **Always use `./merlin`** — Never invoke `cmake`, `ninja`, `make`, or raw build commands directly. All build, compile, benchmark, CI, and setup operations go through `./merlin` (the thin wrapper at the repo root that runs `tools/merlin.py` inside the `merlin-dev` conda env). See `docs/reference/cli.md` for the full CLI reference.
2. **Always consult `docs/`** — Before proposing changes or answering questions about architecture, workflows, hardware targets, or conventions, read the relevant documentation under `docs/`. This includes dev-blogs (`docs/dev_blog/`), architecture notes (`docs/architecture/`), how-to guides (`docs/how_to/`), and reference pages (`docs/reference/`).
3. **Never skip the environment** — All commands must run inside the correct environment (see below).
4. **Never commit or push** — Do not run `git commit`, `git push`, or create branches unless the user explicitly asks you to. Stage files (`git add`) only when instructed. The user manages all version control operations.

## Environment

Merlin uses a two-layer environment: **conda** for system/native toolchain deps and **uv** for Python packages.

### Running tools (build, compile, benchmark, etc.)

These three forms are equivalent and all fully supported. The docs use
`./merlin` because it is shortest, but nothing about the longer forms is
deprecated:

```bash
./merlin <subcommand> [args...]                                       # wrapper
conda activate merlin-dev && uv run tools/merlin.py <subcommand> [args...]  # direct
conda run -n merlin-dev uv run tools/merlin.py <subcommand> [args...]       # direct, no activate
```

`./merlin` is a 30-line bash wrapper that delegates to form (3) when conda
isn't active and form (2) when it is — it caches nothing.

### Running any Python script

Always use `uv run` so the correct virtualenv and dependencies are resolved:

```bash
conda run -n merlin-dev uv run python <script.py> [args...]
# or if conda is already active:
uv run python <script.py> [args...]
```

Never use bare `python3` or `pip install` — the project uses `uv` (managed via conda) with `pyproject.toml` and `uv.lock`.

## tools/ — The Developer CLI

`tools/merlin.py` is the single unified entrypoint. Available subcommands:

| Subcommand   | Module               | Purpose                                           |
| ------------ | -------------------- | ------------------------------------------------- |
| `build`      | `tools/build.py`     | Configure and build Merlin and target runtimes    |
| `compile`    | `tools/compile.py`   | Compile MLIR/ONNX models to target artifacts      |
| `setup`      | `tools/setup.py`     | Bootstrap developer environment and toolchains    |
| `ci`         | `tools/ci.py`        | Run repository CI/lint/patch workflows            |
| `patches`    | `tools/patches.py`   | Verify submodule state and manage upstream patches |
| `benchmark`  | `tools/benchmark.py` | Run benchmark helper scripts                      |
| `chipyard`   | `tools/chipyard.py`  | Manage Chipyard hardware backend interactions     |

When you need to understand what flags or options a subcommand accepts, read the corresponding `tools/<module>.py` file or run `./merlin <subcommand> --help`.

### Common build examples

```bash
# Host-only vanilla build (compiler tools)
./merlin build --profile vanilla

# SpacemiT cross-compile with plugin
./merlin build --profile spacemit

# Build a specific cmake target
./merlin build --profile spacemit --cmake-target <target_name>

# Compile a model
./merlin compile models/dronet/dronet.mlir --target spacemit_x60
```

## docs/ — Always Read Before Acting

The `docs/` directory is the authoritative source for how this project works. Key locations:

- **`docs/getting_started.md`** — Quickstart for first-time usage.
- **`docs/repository_guide.md`** — Repo layout and placement conventions for new code.
- **`docs/architecture/`** — Design decisions: plugin/patch model, cmake presets, etc.
- **`docs/how_to/`** — Step-by-step guides for adding dialects, HAL drivers, samples, compile targets.
- **`docs/reference/`** — CLI reference, cmake targets, C++ API, MLIR ops, Python API.
- **`docs/dev_blog/`** — Active engineering logs with debugging context, decisions, and progress. These are invaluable for understanding *why* things are the way they are and what is currently being worked on.

When working on a task, check whether there is a relevant dev-blog entry or how-to guide before starting.

## Repository Layout

```text
merlin/
├── compiler/        — C++/MLIR compiler code (dialects, passes, plugins)
├── tools/           — Python developer entrypoints (build, compile, setup, ci, etc.)
├── models/          — Model definitions, exports, quantization helpers
├── samples/         — C/C++ runtime examples and hardware sample flows
├── benchmarks/      — Benchmark scripts and board-specific profiling
├── docs/            — Documentation source (MkDocs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ucb-bar/merlin](https://github.com/ucb-bar/merlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
