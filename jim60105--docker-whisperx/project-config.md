---
trigger: always_on
description: Authoritative instructions for AI coding agents working in this repository.
---

# AGENTS.md — docker-whisperX

Authoritative instructions for AI coding agents working in this repository.
Read this file fully before taking any action.

> Repository: <https://github.com/jim60105/docker-whisperX>

---

## 1. Communication & Language

| Context | Language |
| --- | --- |
| Replies to the user in chat | **Traditional Chinese (`zh-TW` 正體中文)** |
| Source code, code comments, identifiers | **English** |
| Commit messages, branch names, PR titles | **English** (Conventional Commits) |
| PR descriptions / work reports | **Traditional Chinese** (code blocks stay English) |
| GitHub Issue descriptions | **Traditional Chinese** (code blocks stay English) |

Be precise, consult both this file and the conversation history before acting, and prefer asking over assuming when requirements are ambiguous.

---

## 2. Environment Constraints

- **No `docker` CLI is available.** Use `podman` for any container operations you run locally.
- Builds in CI use **Docker Buildx + `docker buildx bake`** — when reasoning about CI you are reasoning about Buildx semantics, not plain `docker build`.
- The repository is a **Git submodule host**: `whisperX/` is an upstream submodule. Always clone recursively (`git clone --recursive`) and remember that changes inside `whisperX/` belong upstream, not here.
- Target runtime is **GitHub Free runners** (`ubuntu-latest` for amd64, `ubuntu-24.04-arm` for arm64). Build cost and runner disk space (~10 GB free) are first-class concerns.

---

## 3. Project Overview

This repository packages [WhisperX](https://github.com/m-bain/whisperX) (Automatic Speech Recognition with word-level timestamps and speaker diarization) into reproducible, GPU-capable Docker images published to `ghcr.io/jim60105/whisperx`.

The engineering focus is **not** WhisperX itself but the **build pipeline**:

- Around **370+ image variants** (`WHISPER_MODEL` × `LANG`) are produced **weekly** on free runners.
- Each final image is roughly **10 GB**.
- Success depends on disciplined **layer caching, stage reuse, parallel matrix builds, and multi-architecture support** (`linux/amd64` + `linux/arm64`).

Touch the Dockerfile, `docker-bake.hcl`, or workflows with this constraint in mind: any change that breaks cache reuse or balloons image size is a regression.

---

## 4. Repository Layout

```
docker-whisperX/
├── Dockerfile              # Multi-stage build (see §5)
├── docker-bake.hcl         # Buildx bake matrix (models × languages)
├── load_align_model.py     # Preloads wav2vec2 alignment models per LANG
├── whisperX/               # Git submodule — upstream WhisperX source
├── .hadolint.yml           # Hadolint ignore list for the Dockerfile
├── .github/
│   ├── copilot-instructions.md  # Mirror of this guidance for Copilot
│   └── workflows/          # CI pipelines (build base → cache → matrix → publish)
└── README.md
```

---

## 5. Dockerfile Architecture

The Dockerfile is a deliberately ordered multi-stage graph. Preserve the stage names and their roles — CI references them by name.

| Stage | Purpose |
| --- | --- |
| `prepare_base_amd64` / `prepare_base_arm64` | Install arch-specific runtime libs (e.g. `libnppicc12` on amd64, `libgomp1` + `libsndfile1` on arm64). |
| `base` | Selected dynamically via `prepare_base_${TARGETARCH}${TARGETVARIANT}`. |
| `build` | Installs `uv`, fetches `dumb-init`, runs `uv sync` against the `whisperX/` submodule into `/venv`. |
| `no_model` | Runtime image: non-root user (`UID=1001`), ffmpeg, copies `/venv` from `build`, sets `PATH`/`PYTHONPATH`/`LD_LIBRARY_PATH`, smoke-tests `whisperx -h`. Tagged in CI as `latest` / `no_model`. |
| `load_whisper` | Preloads Silero VAD + a specific `faster_whisper` model (`WHISPER_MODEL`). Cached per model in CI. |
| `load_align` | Runs `load_align_model.py` for each entry in `LANG`. |
| `final` | Combines `no_model` runtime with the populated `/.cache` from `load_align`; sets entrypoint to launch `whisperx` with the resolved model + first language. |

Key conventions:

- **Cache mounts** use stable IDs scoped by arch: `--mount=type=cache,id=apt-$TARGETARCH$TARGETVARIANT,sharing=locked,...`. Match this pattern when adding new package installs.
- `LOAD_WHISPER_STAGE` and `NO_MODEL_STAGE` ARGs let CI swap entire stages with pre-built remote images for caching. Locally they default to in-tree stage names — leave the defaults alone.
- The `CACHE_HOME=/.cache` path is **load-bearing**: the diarization model does not honour `TORCH_HOME`, see issue #27. Do not relocate it.
- Run as `USER $UID` (1001) with `chown $UID:0 chmod 775` on writable dirs — this is required for OpenShift compatibility. Preserve it on any new `COPY`/`RUN install -d`.
- **Never use `,z` or `,Z` mount flags** — Buildx does not support them, even though `podman` does.
- Hadolint runs against the Dockerfile. The ignored rule list lives in `.hadolint.yml`; do not silence additional rules without justification.

---

## 6. `docker-bake.hcl` & Build Matrix

- `target "build"` is a matrix over `WHISPER_MODEL` (`tiny`, `base`, `small`, `medium`, `large-v3`, `distil-large-v3`) × `LANG` (~40 languages — see the file for the canonical list).
- `target "no_model"` produces the model-less runtime image tagged `latest` and `no_model`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jim60105/docker-whisperX](https://github.com/jim60105/docker-whisperX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
