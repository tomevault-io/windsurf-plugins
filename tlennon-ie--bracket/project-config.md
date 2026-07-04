---
trigger: always_on
description: This file is read by Claude Code, Cursor, Cline, Aider, and other coding
---

# Bracket — agent + contributor guide

This file is read by Claude Code, Cursor, Cline, Aider, and other coding
agents that operate on this repository. It is also the fastest read for a
human contributor who just landed.

> Tagline: *Train the same diffusion model eight ways. Pick the one that
> looks best. With a p-value.*

---

## What this is

Bracket is a hyperparameter-search and ranking tool for diffusion-model
fine-tunes. The user points it at a dataset, picks a base model and training
type, sets a budget, and Bracket runs many short training trials in parallel
configurations, scores each by training-loss curve **and** sample-image
quality (via a local VLM), and emits a Markdown report with statistical
confidence on which config wins.

Bracket **drives** existing trainers — it does not re-implement training.
The trainers it drives today are
[`musubi-tuner`](https://github.com/kohya-ss/musubi-tuner) (for Z-Image and
Flux-2-Klein) and [`sd-scripts`](https://github.com/kohya-ss/sd-scripts)
(for SDXL). The installer clones both into `~/.cache/bracket/trainers/` so
you never depend on any specific developer's machine.

## How the code is organised

| Concern | Single source of truth |
|---|---|
| Trainer adapters (one per model+mode) | [`bracket/trainer/`](./bracket/trainer/) |
| Hyperparameter search controllers | [`bracket/search/`](./bracket/search/) |
| Search-space overrides (`apply_search_overrides`, `clamp_config_to_overrides`) | [`bracket/search/space.py`](./bracket/search/space.py) |
| Run launcher (subprocess + tfevents) | [`bracket/orchestrator/runner.py`](./bracket/orchestrator/runner.py) |
| Scoring (loss + VLM) | [`bracket/orchestrator/scorer.py`](./bracket/orchestrator/scorer.py) |
| Orchestration loop (baseline → curated → search → finals) | [`bracket/orchestrator/loop.py`](./bracket/orchestrator/loop.py) |
| Dataset preflight validation + CLI | [`bracket/dataset/validator.py`](./bracket/dataset/validator.py), [`bracket/dataset/validator_cli.py`](./bracket/dataset/validator_cli.py) (run as `python -m bracket.dataset.validator <toml>`) |
| Latent-cache detection (drives sd-scripts `--skip_cache_check`) | [`bracket/dataset/latent_cache.py`](./bracket/dataset/latent_cache.py) |
| VLM judge protocol + LMStudio impl | [`bracket/judge/`](./bracket/judge/) |
| Markdown report (auto-regens on stale ledger) | [`bracket/proof/report.py`](./bracket/proof/report.py) |
| Model + training-type registry (UI dropdowns) | [`bracket/registry.py`](./bracket/registry.py) |
| FastAPI server (HTTP + WebSocket + static frontend) | [`bracket/api/`](./bracket/api/) |
| React frontend (Vite + shadcn/ui) | [`frontend/`](./frontend/) |
| Results-page loss-curve overlay | [`frontend/src/components/results/RunComparisonChart.tsx`](./frontend/src/components/results/RunComparisonChart.tsx) |
| Legacy Gradio UI (deprecated, removed in v0.2) | [`bracket/ui/`](./bracket/ui/) |
| Tests | [`tests/`](./tests/) — orchestrator unit tests stub `validate_dataset_toml` via [`tests/conftest.py`](./tests/conftest.py) |

Every concern has exactly one canonical module. Adding a new trainer is
~150 lines: implement the [`Trainer`](./bracket/trainer/base.py) protocol
and register a preset in [`registry.py`](./bracket/registry.py).

## Conventions agents must follow

1. **Import path is `bracket`**, not `omnisteer_diffusion` (renamed for v0.1).
2. **No hardcoded user-machine paths** in source code. All defaults come from
   env vars (`BRACKET_*` prefix) or the installer-managed
   `~/.cache/bracket/trainers/` location. See [`registry.py`](./bracket/registry.py).
3. **Tests are unit-only and must stay fast.** Anything touching real GPUs,
   real subprocess training, or real LMStudio belongs in a smoke test outside
   the repo (see *Smoke tests* below).
4. **All changes need passing tests.** Run `pytest -q` before claiming a
   change is done. Target: <30 s for the full suite.
5. **Type-annotate everything** in `bracket/api/`. The Pydantic schemas there
   are the contract for the React frontend; `Any` is banned.
6. **Logger names are dotted** — `logging.getLogger("bracket.<subpackage>")`,
   never bare `logging.info`.
7. **No `print()` statements** in library code. Use `logger.info` etc.
8. **Subprocess output never piped through `subprocess.PIPE`** — always
   redirect to a log file directly. Tqdm `\r` updates fill OS pipe buffers
   and freeze the trainer; this is documented at length in
   [`bracket/trainer/base.py`](./bracket/trainer/base.py)'s comments.

## Common operations

### Run the dev stack
```bash
# Terminal 1
./launch.sh                       # FastAPI on :8000
# Terminal 2
cd frontend && npm run dev        # Vite on :5173 with proxy to :8000
```

### Run tests
```bash
source .venv/bin/activate         # or .\.venv\Scripts\Activate.ps1
pytest -q                         # full suite
pytest -q tests/test_api.py       # one file
pytest -q -k "scorer"             # by name
```

### Add a new trainer adapter
1. Create `bracket/trainer/<model>_<mode>.py` mirroring
   [`zimage_full.py`](./bracket/trainer/zimage_full.py) — implement
   `declare_search_space()`, `baseline_config()`, `curated_configs()`,
   `prepare_run()`, `config_from_dict()`.
2. Register a preset in [`registry.py`](./bracket/registry.py) — add a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tlennon-ie/bracket](https://github.com/tlennon-ie/bracket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
