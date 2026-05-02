---
trigger: always_on
description: This is a CLI tool for tuning vLLM's Triton kernel configs on NVIDIA GPUs.
---

# vLLM-Tune — Agent Guide

This is a CLI tool for tuning vLLM's Triton kernel configs on NVIDIA GPUs.
It benchmarks MoE and FP8 GEMM kernels inside a running Docker container,
saves optimal configs as JSON, and deploys them via `docker cp`.

---

## Architecture

```
vllm-tune.sh                ← Main entry point (orchestrator)
├── tune-moe.sh             ← MoE fused expert dispatch tuning
├── tune-fp8.sh             ← FP8 block-scaled GEMM tuning
├── lib/common.sh           ← Shared utilities (sourced by tune-*.sh)
├── mod/run.sh              ← Container mod for persistent deployment
└── configs/                ← Config store (pre-shipped + user-generated)
    └── <model-slug>/tp<N>/
        ├── moe/*.json      ← E=...,N=... kernel configs
        ├── fp8/*.json      ← N=...,K=... kernel configs
        └── metadata.json   ← Provenance (model, TP, vLLM version, timestamp)
```

### Data flow

```
1. vllm-tune.sh parses args → delegates to tune-moe.sh / tune-fp8.sh
2. tune-*.sh runs benchmarks inside Docker container (via docker exec)
3. After each batch/shape: docker cp results out → jq merge into configs/
4. On completion: optional docker cp deploy into container's vLLM site-packages
```

### Key design decisions

- **Crash-safe**: After every single batch size (MoE) or shape (FP8), results
  are copied from the container and merged into host-side backup. A power loss
  mid-tuning loses only the current item.
- **Incremental**: `jq -s '.[0] * .[1]'` merging means re-running with a
  subset of batch sizes extends (not overwrites) the config.
- **No container modifications**: All tuning commands use `docker exec`. The
  container image is never modified.

---

## File responsibilities

| File | Role | Modifiable in isolation? |
|------|------|-------------------------|
| `vllm-tune.sh` | CLI parsing, tmux, deploy, mod sync, metadata | Yes |
| `tune-moe.sh` | MoE benchmark loop (calls `benchmark_moe.py` via git sparse-checkout) | Yes |
| `tune-fp8.sh` | FP8 benchmark loop (inline Python Triton benchmark) | Yes |
| `lib/common.sh` | Shared functions (MUST be updated if changing retry/merge/report behavior) | Careful — sourced by both tune scripts |
| `mod/run.sh` | Runs inside container at startup — copies JSON configs into vLLM paths | Yes |
| `configs/` | JSON kernel configs — tracked in git, never auto-deleted | Append-only |

---

## Conventions

### Script conventions

- All scripts use `set -euo pipefail`
- Error output goes to stderr via `die()` / `echo ... >&2`
- Functions are documented with comments specifying arguments and globals
- ANSI colors used for terminal output (bold, cyan, red for errors)
- Environment variables override defaults (never hardcode user-specific values)

### Config conventions

- Config filenames are set by vLLM's benchmarking tools — do NOT rename them
- MoE format: `E=<experts>,N=<dim>,device_name=<GPU>,dtype=<dtype>,block_shape=[128,128].json`
- FP8 format: `N=<out>,K=<in>,device_name=<GPU>,dtype=fp8_w8a8,block_shape=[128,128].json`
- Model slug: lowercase, `/` → `--`, special chars → `-` (e.g. `qwen--qwen3.6-35b-a3b-fp8`)
- Config store path: `configs/<model-slug>/tp<N>/{moe,fp8}/`

### Naming

- The project is **vLLM-Tune** (capital L, capital T, hyphen)
- The CLI command is **`vllm-tune.sh`** (lowercase, hyphen)
- The mod is **`vllm-tune`** (lowercase, hyphen)

---

## Common tasks

### Adding support for a new tuning mode

If vLLM adds a new kernel type that benefits from tuning:

1. Create `tune-<type>.sh` following `tune-moe.sh` as template
2. Source `lib/common.sh` for shared utilities
3. Add a new `--mode <type>` option to `vllm-tune.sh`
4. Add a `CONFIGS_<TYPE>` path and deploy target in `deploy_all()`
5. Update `mod/run.sh` to install the new config type

### Adding pre-shipped configs for a new model

1. Run tuning: `./vllm-tune.sh <model> --tp <N>`
2. Verify results with a benchmark
3. Commit the `configs/<model-slug>/tp<N>/` directory
4. Add a row to the "Pre-shipped Configs" table in `README.md`

### Modifying shared behavior (retry, merge, reports)

All shared logic lives in `lib/common.sh`. Both `tune-moe.sh` and `tune-fp8.sh`
source it. Key functions:

| Function | What it does |
|----------|-------------|
| `preflight()` | Container check + tini detection + jq check |
| `run_with_retry()` | Execute command with MAX_RETRIES retries |
| `merge_results()` | Crash-safe jq merge from container to host |
| `post_round()` | Clear caches + check for zombies |
| `generate_report()` | Write Markdown tuning report |
| `print_summary()` | Print styled terminal summary |
| `cleanup_zombies()` | Restart container if zombie processes accumulated |
| `clear_caches()` | Drop OS page caches (configurable via env vars) |

**Rule**: If you change a function signature in `common.sh`, you must update
both `tune-moe.sh` and `tune-fp8.sh`.

### Modifying the container deployment paths

vLLM kernel config paths are auto-detected at runtime:

```bash
# In vllm-tune.sh:
detect_vllm_paths()  # runs: python3 -c "import vllm; ..."

# In mod/run.sh:
VLLM_ROOT=$(python3 -c "import vllm, pathlib; ...")
```

Both fall back to `/usr/local/lib/python3.12/dist-packages/vllm` if detection
fails. If vLLM changes its internal config directory structure, update:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeraphimSerapis/vllm-tune](https://github.com/SeraphimSerapis/vllm-tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
