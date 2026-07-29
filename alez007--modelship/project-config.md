---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

`AGENTS.md` is the canonical operational guide — read it first for toolchain, commands, gotchas, release flow, and plugin authoring. This file summarizes the points most often needed mid-task.

## Toolchain essentials

- Python is pinned exactly to `3.12.10` (not `>=`). Dependency manager is **uv** with a workspace; `plugins/*` are workspace members. Never use `pip install`.
- `cuda` and `cpu` extras are **mutually exclusive** (declared in `[tool.uv] conflicts`) — `torch`/`torchvision` come from different indexes per extra. A third extra, `thin`, is empty — no torch/vllm.
- Line length is **120**, not 88. Ruff owns formatting (`E501` disabled); don't hand-sort imports (isort via `I` rule handles it). `plugins/*` are third-party to isort; `modelship` is first-party.
- Pyright runs in `basic` mode, scoped to `modelship`, `plugins`, `mship_deploy.py`. Pre-commit only runs ruff — don't rely on it to catch type errors.

## Common commands

```bash
# Install (choose cuda XOR cpu, plus dev, plus optional plugin extras)
uv sync --extra dev --extra cuda                       # what CI uses
uv sync --extra dev --extra cpu --extra kokoroonnx     # CPU + a plugin

make lint        # ruff check + ruff format --check + pyright — all three MUST pass
make lint-fix    # auto-fix ruff issues
make test        # uv run pytest tests/ -v

# Single test
uv run pytest tests/test_config.py::TestLlamaServerConfig::test_defaults -v
```

CI mirrors `make lint` + `pytest tests/ -v`. Match it locally before pushing.

`make lint` requires the `cuda` extra — pyright fails with `reportMissingImports` for `gguf`, `diffusers`, and `psutil` under the cpu sync. (`vllm` is now importable under both extras — Stage E0 wired a CPU wheel index — but that alone doesn't unblock a cpu-only lint.) Tests pass on either extra.

When running tests on your own initiative, skip the slow integration suite: `uv run pytest tests/ -v -m "not integration"`. Only run full `make test` when explicitly requested.

## Running the server

`mship_deploy.py` is the entry point (not a console script, not `python -m`). It:

1. Reads `config/models.yaml` (gitignored — copy from `config/examples/`). An explicit `--config <path>` that doesn't exist is still a hard error. Absent both `--config` and the default file, `mship_deploy.py` no longer errors — it bootstraps an empty coordinator (gateway up, no models) that waits for a later `--config`/`--reconcile` redeploy or a joining node to bring capacity; this is the mode a bare `docker run modelship:thin` with no mounted config exercises.
2. Starts its **own** Ray head by default and tears it down on exit. With `--use-existing-ray-cluster` it instead connects to a cluster you manage via `ray.init(address="auto")` and deploys-and-exits without teardown — the driver must run **on** a cluster node (it can't attach from off-cluster; k8s does this via a KubeRay RayJob).
3. Deploys models **additively** by default (each gets a random suffix like `qwen-a3f9k`). Use `--reconcile` to instead make the cluster match the config exactly (add/remove/replace) — it never tears the cluster down.
4. Starts a FastAPI Ray Serve app named `modelship api` on port 8000. Override via `--gateway-name` (multiple gateways can coexist on one cluster).

Docker's `CMD` is `uv run --no-sync mship_deploy.py` (auto-detecting CPUs/GPUs unless `MSHIP_NODE_NUM_CPUS`/`MSHIP_NODE_NUM_GPUS` set), against the prebuilt venv (extras chosen by `--build-arg MSHIP_VARIANT=thin|cpu|cuda`). Plugin wheels in `MSHIP_PLUGIN_WHEEL_DIR` ship to Ray workers per-deployment via `runtime_env`, resolved from `models.yaml`. The Dev Container overrides this — inside it you must `uv sync` and run `mship_deploy.py` manually. Right after connecting, the driver logs the cluster's observed node/GPU/CPU totals (total vs. currently schedulable) — the quickest way to tell a legitimately-waiting head apart from a misconfigured one.

## Architecture map

- `mship_deploy.py` — Ray init + deploy loop. `build_deployment_options` (in `modelship/deploy/actor_options.py`) handles GPU allocation: multi-slot vLLM deploys (`tp*pp > 1`) always build a Ray Serve placement group (one whole-GPU bundle per slot, STRICT_PACK) that vLLM's ray executor inherits via `get_current_placement_group()`. Single-slot deploys use a scalar `num_gpus` on the outer actor (fractional sharing supported). Fractional `num_gpus` with `tp*pp > 1` is rejected at config time — Ray packs fractional PG bundles onto the same physical GPU. `llama_server` loader supports whole-GPU offload (`num_gpus` must be `0` or an integer — fractional is rejected, llama.cpp has no VRAM-fraction knob); `stable_diffusion_cpp` remains CPU-only, forced to `num_gpus: 0`.
- `modelship/openai/api.py` — FastAPI gateway. Uses `RequestWatcher` + a single shared `DisconnectRegistry` Ray actor (keyed by request id) to propagate client disconnects across process boundaries and cancel in-flight inference.
- `modelship/infer/model_deployment.py` — the single `@serve.deployment` actor class; lazily imports the right backend from `config.loader`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alez007/modelship](https://github.com/alez007/modelship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
