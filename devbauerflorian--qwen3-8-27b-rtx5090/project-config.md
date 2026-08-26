---
trigger: always_on
description: Guidance for AI agents working in this repository. This repo is a
---

# AGENTS.md

Guidance for AI agents working in this repository. This repo is a
**deployment recipe**, not a framework: a single, fully pinned Docker stack
that serves **Qwen3.8-27B (NVFP4, ~18.8 GB)** on **one RTX 5090 (32 GB)** via
vLLM, with an OpenAI-compatible API (reasoning + auto tool choice, 256K
context), a Basic-auth gateway, and a Prometheus/Grafana/DCGM monitoring
sidecar set.

- Human docs: [`README.md`](README.md) (canonical, keep in sync), [`CONTRIBUTING.md`](CONTRIBUTING.md) (commit style).
- Everything that can break a rebuild is **pinned** (see *Pinning discipline* below). Never "modernize" a version casually.

## Commands

| Task | Command |
|---|---|
| Install host prerequisites + download weights (~19 GB, pinned HF revision) | `sudo ./setup.sh` |
| Configure | `cp .env.example .env` then edit (secret: `.env` is gitignored — never commit it) |
| Build + start the stack | `docker compose up -d --build` |
| Watch boot | `docker logs -f vllm` (ready when logs show `Application startup complete`) |
| Smoke test | `curl http://localhost:8020/v1/models` (add `-H "Authorization: Bearer $VLLM_API_KEY"` if the key is set) |
| Smoke, via gateway | `curl -u "$METRICS_USER:$METRICS_PASSWORD" http://localhost:8030/v1/models` |
| Context benchmark | `.venv_download/bin/python benchmarks/context_bench.py` (add `--cached` for prefix-cache deltas, `--quick` for sanity) |
| Throughput benchmark | `.venv_download/bin/python benchmarks/parallel_bench.py` |

There is no test framework or linter. **Verification = the stack comes up,
the smoke curl works, and the benchmark suites pass.** Run benchmarks only on
an idle GPU; results land in `benchmarks/results/` (gitignored, except
`.gitkeep`) and both suites print a paste-ready markdown table for the README.
Bench scripts need `httpx` (+ optional `tokenizers`), which live in
`.venv_download/` (created by `setup.sh`; `httpx` falls back to stdlib
`urllib` if missing).

## Repo map

| Path | What |
|---|---|
| `Dockerfile` | vLLM image: CUDA 13.3.1-devel-ubuntu22.04 base, pinned uv 0.12.5 / Python 3.13.15, full `requirements.lock` (196 pkgs). `ENTRYPOINT ["vllm", "serve"]` |
| `requirements.lock` | Frozen production Python stack — the single source of truth for the image's packages |
| `docker-compose.yml` | All services + vLLM serve flags (light setup: no MTP) — the canonical place serve behavior is configured |
| `docker-compose.mtp.yml` | Override file for heavy usage: full replacement of the vllm `command` list that adds the MTP speculative-decode flag, shipping the dedicated-card ideal-usage tuning (3 seq / 0.965, the shipped full-MTP tuning — bench runs single-stream); run `docker compose -f docker-compose.yml -f docker-compose.mtp.yml up -d` |
| `setup.sh` | Installs pinned `nvidia-container-toolkit==1.20.0-1`, creates `.venv_download/`, downloads weights into `./models/<MODEL_SUBDIR>/` via pinned `huggingface_hub[cli]==1.27.0` `hf download` |
| `.env.example` | Template for `.env`; `docker compose` **and** `setup.sh` both read it |
| `caddy/Caddyfile` | gateway on container port 8030: `/v1/*` (Basic → bearer translation, or the vLLM key passed through as-is for 8020 parity), `/metrics`, `/dcgm/metrics` (Basic only) |
| `prometheus/prometheus.yml` | Scrapes `vllm:8000/metrics` + `dcgm-exporter:9400` every 15 s, 30 d retention |
| `grafana/` | Auto-provisioned datasource + dashboard `vllm-qwen3827b.json` (folder **vllm**) |
| `dcgm-exporter/` | Minimal NVML→Prometheus sidecar (public base image + `nvidia-ml-py` — no NGC login needed). Swap for `image: nvcr.io/nvidia/dcgm-exporter:3.3.9-3.6.0-ubuntu22.04` if you have NGC access |
| `benchmarks/` | `context_bench.py` (TTFT/prefill/decode per context size) and `parallel_bench.py` (aggregate throughput vs concurrency); raw runs in `results/` |
| `models/qwen3.8-27b-nvfp4/` | Weights — **empty in git**, filled by `setup.sh` (gitignored, ~19 GB) |
| `.venv_download/`, `unsloth-nvfp4-env/`, `.venv/` | Local venvs, gitignored build artifacts |

## Services & ports

| Service | Host port | Notes |
|---|---|---|
| `vllm` | `8020` (→ container `8000`) | Direct full API. Auth: `VLLM_API_KEY` bearer on `/v1/*` — **empty key = fully open (LAN only)**. `/metrics`, `/health`, `/docs` stay open by vLLM design |
| `caddy` | `8030` (→ container `8030`) | gateway: Basic auth; the plain vLLM key is also accepted on `/v1/*`; fail-closed: **refuses to start if `METRICS_HASH` is unset** |
| `grafana` | `3000` (fixed, all interfaces) | Login `admin` / `GRAFANA_ADMIN_PASSWORD` (sign-ups disabled) |
| `prometheus` | `127.0.0.1:9090` (fixed, loopback) | Remote access via `ssh -L 9090:localhost:9090` |
| `dcgm-exporter` | none | Docker-network only (`dcgm-exporter:9400`); external reads via `8030/dcgm/metrics` |

All services run `restart: unless-stopped` — they come back after a crash, but stay stopped after a manual `docker stop`.

Port override keys in `.env`: `VLLM_HOST_PORT` (default 8020), `GATEWAY_HOST_PORT` (default 8030). **Container-side ports are fixed** (`vllm:8000`, `caddy:8030`) — only the host mapping changes. If you also want to move a container port, touch the `ports` block in `docker-compose.yml` *and* the site line in `caddy/Caddyfile` together.

## Configuration model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devbauerflorian/qwen3.8-27b-rtx5090](https://github.com/devbauerflorian/qwen3.8-27b-rtx5090) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
