---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ShadowLM Trainer is a fine-tuning SDK: load any open model, train it with any of
13 methods, on any hardware, then own the weights. The headline use case is
"shadowing" — moving one task off a rented frontier model onto a small model you
own, by capturing real agent traffic (`slm.capture()`), judging episodes, and
training on them — without modifying the agent (the model API is the only
boundary). The repo is the **engine**; the orchestration tier is ShadowLM Studio.

The whole product reads like the task in `shadowlm/models.py`:
`slm.load(...)` → `model.finetune(ds, method=...)` → `model.generate(...)` →
`model.save(...)`. Keep that surface tiny — the machinery lives in the backends.

## Commands

```bash
make install           # editable install with CLI + mlx backend (Apple Silicon dev loop)
make install-torch     # editable install for CUDA / CPU boxes
make frontend          # npm install + build the React studio into shadowlm/_static
make serve             # studio UI + API on one port (PORT=8329)
make dev               # serve with Vite hot-reload UI alongside the backend
make demo              # end-to-end smoke: a tiny CLI finetune (mlx, 0.5B, ~seconds)
make check             # compileall the package + `tsc -b` the frontend
make build             # build the frontend, then the wheel+sdist, then twine check
make release           # bump patch (or BUMP=minor/major, V=x.y.z), build, tag, push

pytest                                       # the CPU test suite (tests/, excludes gpu/)
pytest tests/test_more_plus_router.py        # a single test file
pytest tests/test_more_plus_router.py::test_name -x   # a single test, stop on first fail
make gpu-test          # the CUDA verification suite — run on a GPU box only
```

`tests/gpu/` (CUDA, real GPU) is **not** part of the default `pytest` run — it's
invoked explicitly via `make gpu-test` or `python tests/gpu/test_cuda.py`.

Releases publish to PyPI via `.github/workflows/publish.yml` on a `v*` tag. The
CI gate requires the version to match in **three** places: the git tag,
`pyproject.toml`, and `shadowlm/__init__.py`. `make bump`/`make release` keep
the latter two in sync — never edit only one.

## Architecture

Two orthogonal registries — **backends** (where training runs) × **methods**
(what training does) — meet in the SDK surface. Adding a backend or a method
touches one file and no others.

### The two axes

- **`shadowlm/backends/`** — a `Backend` (see `backends/base.py`) holds a loaded
  model and knows how to `load` / `finetune` / `generate` / `chat` / `save`.
  Implementations: `mlx.py` (Apple-Silicon dev loop), `torch.py` (the production
  CUDA/CPU path, on HF `Trainer` + `accelerate` + `trl` + `peft`), `remote.py`
  (speaks the JSON protocol to a server), `verl.py` (multi-GPU GRPO). Selection
  lives in `backends/__init__.py::select_backend` — `auto` = CUDA→torch,
  else Apple→mlx, else torch-on-CPU. **Everything user-facing is
  backend-agnostic**; mlx and torch must stay swappable without changing the SDK.

- **`shadowlm/methods/`** — each method is a declarative `TrainingMethod` spec
  (`methods/base.py`): an adapter kind (`ADAPTER_LORA`, `ADAPTER_MORE`, …), a
  base-model requirement (`quantized_base`: True=needs 4-bit, False=needs
  unquantized, None=either), a `trainer` ("sft"/"dpo"/"grpo"), and a default LR.
  **Backends dispatch on the spec's fields, never on the method name** — that
  invariant is what makes `method="lora"` → `"qlora"` a one-word change.
  Registering a method is a new module with one `register(...)` call, imported in
  `methods/__init__.py`; users can `methods.register(...)` at runtime too.

### The SDK surface (`models.py`, `training.py`, `data.py`)

- `models.py` — `load()` returns a `Model`; `Model.finetune/generate/chat/save`.
  This is the whole library in one object; resist growing it. Tool-call parsing
  for `chat()` (small models emit slightly mangled tool JSON) lives here too.
- `training.py` — `TrainConfig` (every hyperparameter, with which backend honors
  it noted inline), `Metric`, and `TrainingRun` (the live+final handle:
  metrics history, sparkline/plot, checkpoints, persistence). `TrainConfig` is
  the single source of truth — the CLI's `--set`/`--config` validates against the
  dataclass so it can't drift from the SDK.
- `data.py` — `Dataset` is rows + a detected format (chat / sharegpt /
  preference / instruction / text / raw). Backends turn a formatted dataset into
  training text. Local loading is pure-stdlib; `from_hf` lazy-imports `datasets`.

### The shadowing / agent-tuning loop

- `capture.py` — `slm.capture(model)` is a drop-in OpenAI-compatible proxy that
  records an unmodified agent's traffic, reconstructing message-level
  trajectories (calls that extend a prior call's message prefix merge into one
  episode; use an `x-session-id` header to disambiguate interleaved conversations).
- `rl.py` — `Trajectory` / `TrajectoryGroup` / `judge_group` (LLM-judge scoring),
  fed into `method="grpo"`.
- `apo.py` — `optimize_prompt()`: optimize the prompt instead of weights, same
  capture/judge front end, no GPU.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-gitagent/shadowLM](https://github.com/open-gitagent/shadowLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
