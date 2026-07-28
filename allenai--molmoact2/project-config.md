---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo scope

This is the MolmoAct2 release repo. The tracked code under this working directory is:

- `examples/droid/host_server_droid.py` — FastAPI inference server for `allenai/MolmoAct2-DROID` (2 cams, 8-D state, `norm_tag="franka_droid"`, default port 8000).
- `examples/yam/host_server_yam.py` — same shape, but for `allenai/MolmoAct2-BimanualYAM` (3 cams `[top, left, right]`, 14-D state, `norm_tag="yam_dual_molmoact2"`, default port 8202).

Untracked but kept locally:

- `logs/inference_script.py` — DROID Polymetis client bridge that talks to `host_server_droid.py` from the NUC driving a Franka. Not tracked in this branch; treat it as the reference wire-format consumer for DROID.
- `README copy.md` — local snapshot of the pre-cloud-merge README, kept for diff purposes.

The rest of MolmoAct2 (training, fine-tuning, eval) is "coming soon" per the README. The `lerobot/` directory is a submodule (`allenai/lerobot-molmoact2`); evaluation work (e.g. LIBERO replication) happens in there and uses its own README/tooling.

## Environment

- Python is pinned to 3.11 via `.python-version`; `uv` manages everything.
- Torch wheels come from the CUDA-12.1 PyTorch index (configured in `pyproject.toml` under `[tool.uv.sources]`). Don't relax these pins casually — the model loading code path was validated against torch 2.5.1 / transformers 4.57.x.
- After pulling new commits: `uv sync`. After cloning: also `git submodule update --init --recursive`.

## Common commands

```bash
uv sync                                                                    # install/refresh .venv
uv run python examples/droid/host_server_droid.py --host 0.0.0.0           # DROID server, default port 8000 (bf16)
uv run python examples/yam/host_server_yam.py --host 0.0.0.0               # YAM   server, default port 8202 (bf16)
uv run python examples/droid/host_server_droid.py --dtype float32 --cuda-graph  # full precision + CUDA graphs
uv run hf download allenai/MolmoAct2-DROID                                 # pre-cache DROID (~22 GB)
uv run hf download allenai/MolmoAct2-BimanualYAM                           # pre-cache YAM   (~21 GB)
curl http://<host>:8000/act                                                # DROID health
curl http://<host>:8202/act                                                # YAM   health
```

Useful server flags: `--dtype {bfloat16,float16,float32}` (default bf16; fp32 needs ~96 GB VRAM), `--device cuda:0`, `--cuda-graph` (~2× faster action expert, +~2 GB VRAM, not safe under concurrent calls), `--no-warmup`.

There is no test suite or linter wired up in this repo.

## Wire protocol (`/act`)

Both directions are `json_numpy`-encoded (each server calls `json_numpy.patch()` at import, which monkey-patches the stdlib `json` module so ndarrays round-trip). Each server has its own schema — the endpoint path is the same but the payload shape differs by embodiment.

DROID (`examples/droid/host_server_droid.py`):

- Request: `external_cam` (H,W,3 uint8 RGB), `wrist_cam` (H,W,3 uint8 RGB), `instruction` (str), `state` (8,) float32 = `[q1..q7, gripper]`. Optional: `num_steps` (default 10), `enable_cuda_graph`, `timestamp`.
- Response: `actions` (N, 8) float32 absolute joint positions + gripper, `dt_ms` float.

YAM (`examples/yam/host_server_yam.py`):

- Request: `top_cam`, `left_cam`, `right_cam` (each H,W,3 uint8 RGB — order matters and must match training), `instruction` (str), `state` (14,) float32, plus the same optional fields.
- Response: `actions` (N, D) float32, `dt_ms` float. Action shape is driven by `norm_stats.json`; don't hardcode it.

`logs/inference_script.py` is the DROID reference client (untracked). There's no YAM bridge in this repo; if you write one, mirror the YAM schema above. The bridge is hand-rolled HTTP (`requests.post`), not auto-generated, so server-schema changes need matching client edits.

## Server architecture — non-obvious bits

The MolmoAct2 checkpoints were not released with `bfloat16` or local-snapshot loading in mind. Both `examples/droid/host_server_droid.py` and `examples/yam/host_server_yam.py` apply the same set of upstream workarounds; future changes need to preserve these or a server will silently break:

1. **Snapshot-dir loading.** The model's `predict_action` reads `norm_stats.json` from `config._name_or_path`. Loading by repo id leaves that as a non-path string and crashes at inference time. The server always resolves `snapshot_download(repo_id)` and loads from the local directory.

2. **bf16 patches to upstream `modeling_molmoact2.py`.** `_patch_modeling_for_bf16` rewrites the cached `modeling_molmoact2.py` (both in the snapshot dir and in `~/.cache/huggingface/modules/transformers_modules/*/`, which is the copy `trust_remote_code` actually imports) at startup. Two edits, both idempotent and marked with `# patched_bf16_*` comments:
   - Flow-matching trajectory dtype: hardcoded `torch.float32` → `source_tensor.dtype`.
   - `_to_array`: cast to fp32 before `.numpy()` because numpy has no bf16 dtype.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allenai/molmoact2](https://github.com/allenai/molmoact2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
