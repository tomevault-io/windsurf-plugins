---
trigger: always_on
description: Guidance for Claude Code and other AI agents working in this repository. These rules override default behavior — follow them exactly. `agent-docs/` is the detailed reference for how the code behaves; this file is the rulebook and a map into it. `human-docs/` is the concise human guide.
---

# CLAUDE.md

Guidance for Claude Code and other AI agents working in this repository. These rules override default behavior — follow them exactly. `agent-docs/` is the detailed reference for how the code behaves; this file is the rulebook and a map into it. `human-docs/` is the concise human guide.

## Rules

### Environment & tooling
- **Run everything in the Docker image.** The host has **no usable Python** (`import torch` fails). All deps (PyTorch 2.11+cu130, DeepEP, Flash Attention) live only in the images: `halo:blackwell` (B200/B300, SM100/103, FA4+FA2, no FA3) or `halo:hopper` (H100/H200, SM90, FA2+FA3), built credential-free from source (`make build-blackwell` / `make build-hopper` — no token or secret needed) or pulled prebuilt from **Amazon ECR Public** (anonymous, no AWS account): `docker pull public.ecr.aws/whitecircle/halo:{blackwell,hopper}` — moving tags plus immutable `-$(VERSION)` pins, deliberately no `latest` (it would let a Hopper host pull a Blackwell image); the RL servers are `:vllm-0.26.0` and `:sglang-0.5.17` in the same repository (`make push-public-all` publishes). On a host whose docker default runtime rejects `--gpus` (e.g. sysbox-runc), set `DOCKER_RUNTIME=nvidia` on any `make` target. Images are built with **uv** into the system interpreter — **no Poetry, no venv** — so `python`/`torchrun`/`accelerate`/`pytest`/`ruff` are on `PATH`; call them directly, no prefix. Dependency changes go through uv (`uv lock`, `uv pip install`; see `pyproject.toml` `[tool.uv]`). Standard detached launch:
  ```bash
  # Resolve the scratch volume dynamically (see Preflight) — never hardcode a path.
  D=$(findmnt -rbno TARGET,AVAIL,FSTYPE | awk '$3!~/tmpfs|overlay|squashfs|nfs|fuse|autofs/ && $2+0>20e9{print $2,$1}' | sort -rn | head -1 | awk '{print $2}')
  docker run -d --rm --name <job> --gpus all \
    --ipc=host --ulimit memlock=-1 --ulimit stack=67108864 --shm-size=128g \
    --cap-add=SYS_PTRACE --env-file .env \
    -e HF_HOME="$D/hf" -e HF_DATASETS_CACHE="$D/hf/datasets" \
    -e TMPDIR="$D/tmp" -e HALO_DATA_ROOT="$D" \
    -v $(pwd):/workspace -v "$D:$D" -v ~/.aws:/root/.aws -w /workspace \
    halo:blackwell \
    bash -lc "torchrun --nproc_per_node=8 <script> <config> > \"$D/<job>.log\" 2>&1"
  ```
  Provide your own repo-root `.env` (`WANDB_API_KEY`/`HF_TOKEN` as needed, plus any `AWS_*` for S3; start from `.env.example`); it is **not auto-loaded** — pass it with `--env-file`. `HF_HOME`/`HF_DATASETS_CACHE` (HF caches), `TMPDIR` (temp), and `HALO_DATA_ROOT` (toolkit scratch — S3 dataset cache → `$HALO_DATA_ROOT/s3_datasets`, profiler artifacts → `$HALO_DATA_ROOT/profiling`) redirect writes off the small root FS. They are a **convention pointing at a large mounted volume**, not code defaults (each falls back to `~/.cache/...`), and `/mnt` is **not guaranteed large** (on some hosts `/mnt` shares the small root device) — confirm with `findmnt`/`df -h` and point them at the real large volume. Pre-cached S3 datasets (`md5("<bucket>/<key>")`) then load without live AWS. See `agent-docs/infrastructure/docker.md`.
- **Launcher:** `torchrun` for all multi-GPU — EP/CP/TP/ETP **and** plain FSDP2 data-parallel (`torchrun --nproc_per_node=N scripts/training/<script>.py <config>` lands on FSDP2 ZeRO-2); `python` for single-GPU/LoRA. `accelerate launch` + the `launcher-configs/accelerate/*.yaml` configs remain a supported option for plain FSDP data-parallel, but torchrun is the default. The `halo` CLI (`src/cli.py`) picks the launcher — `halo launch <method> <config>`, `halo run <tool>`.
- **Preflight every run — disk, RAM, GPU.** Send large outputs, caches (`HF_DATASETS_CACHE`), and logs to a **verified** large mounted volume; the root FS is small and on some hosts shares a device with `/mnt`. **Check the target before any multi-GB write** (`df -h` / `findmnt` / `readlink -f` — a path's name does not prove its capacity). Before launching also confirm free host RAM and GPU health/free memory (`nvidia-smi`; `scripts/profiling/nvlink_health.py` for NVLink) — never launch onto a full disk, an OOM-prone host, or a degraded or already-occupied GPU. Scratch/temp markdown → `/tmp`.
- **Env vars.** `src/env.py` is the single home for reading toolkit knobs (`HALO_`/`DIST_`/`VLLM_`) — one convention via `env_flag`/`env_int`/`env_float`/`env_str`, plus `HALO_DATA_ROOT` + `data_path()` for scratch. Route a new toolkit var through it, never a raw `os.environ.get`. Launcher (`LOCAL_RANK`/`SLURM_*`/`RANK`), `ACCELERATE_*`, and HF/OS vars are read raw at their owner by design. Full catalogue with defaults (incl. `NVLINK_DOMAIN_SIZE`, the NVL72 locality unit): `agent-docs/reference/configuration-reference.md` (Environment variables).

### Code style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitecircle/halo](https://github.com/whitecircle/halo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
