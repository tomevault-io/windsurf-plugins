---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Open Data Hub OGX Distribution — a containerized distribution of [OGX](https://github.com/opendatahub-io/ogx) (the opendatahub-io fork of Llama Stack) for AI/ML workflows. The project generates and maintains multi-arch (amd64/arm64) container images with pre-configured providers for inference, vector storage, file processing, and other ML APIs.

The container image is published to `quay.io/opendatahub/odh-ogx-core`.

## Common Commands

```bash
# Regenerate all auto-generated files + run linting
pre-commit run --all-files

# Build container image locally
# The build pulls model/data files from a "modelcar" base image on the stage
# registry (username: agentic-api), so log in first, then build:
#   printf '%s' "$STAGE_REGISTRY_TOKEN" | podman login registry.stage.redhat.io --username agentic-api --password-stdin
#   podman build -t ogx-core .
podman build -t ogx-core .

# Run container (requires PostgreSQL and at least one inference endpoint)
podman run -p 8321:8321 -e VLLM_URL=http://host:8000/v1 ogx-core

# Run smoke tests (requires running container, vLLM, and PostgreSQL)
./tests/smoke.sh

# Run integration tests (clones upstream OGX repo, runs pytest against live server)
./tests/run_integration_tests.sh
```

Linting is handled entirely via pre-commit: Ruff (Python), Shellcheck (shell), Actionlint (GitHub Actions workflows).

## Architecture

### Build Pipeline

`pre-commit run --all-files` triggers local hooks that regenerate distribution artifacts:

1. **`build/gen_config.py`** (hook: `gen-config`, always runs) — strips dependency-only providers from `build/build.yaml` and writes `distribution/config.yaml`.
2. **`build/gen_containerfile.py`** (hook: `gen-containerfile`, always runs) — generates `Containerfile` from `Containerfile.in`, embedding `config.yaml` as base64-encoded OCI labels.
3. **`build/verify_secrets.py`** (hook: `verify-secrets`, always runs) — verifies that secret env vars in `build.yaml` have matching `_FILE` support in `distribution/entrypoint.sh`.
4. **`build/gen_distro_docs.py`** (hook: `doc-gen`, runs when `build/build.yaml`, `build/build.env`, or `distribution/config.yaml` change) — generates `distribution/README.md` with an API/provider table.

**`build/gen_lockfile.py`** (no pre-commit hook — runs in CI and locally via `./build/run_gen_lockfile.sh`) — creates temp venvs, runs `ogx stack list-deps` and `opentelemetry-bootstrap` to discover dependencies, then compiles pinned lock files with `uv pip compile`. Requires Linux; on macOS use `./build/run_gen_lockfile.sh` which runs it inside a container.

Shared configuration (`BuildConfig`) lives in **`build/common.py`**.

### Model/Data Artifacts

At image build time, the model and data files (docling models, HF embedding model, tiktoken
encodings, etc.) come from a pre-published, ProdSec-scanned **"modelcar" container image**
(`registry.stage.redhat.io/rhai/modelcar-redhatai-ogx-distribution:3.0`), rather than being
downloaded from Hugging Face/ModelScope. The modelcar stores every file under `/models` in the
`.cache`-relative layout OGX expects, so a multi-stage `COPY --from` lands them under
`${APP_ROOT}/.cache` (it also ships the HF hub `refs/main` marker directly). It is a UBI-micro
image with no Python, so it is used only as a build stage — `odh-midstream-python-base-3-12`
remains the runtime base.

The `OGX_ARTIFACT_SOURCE` build arg selects how the files are obtained, with **no fallback
between modes** (`Containerfile.in`):

- **`pull` (default; non-fork/local/Konflux):** uses the modelcar image directly as a build
  stage (`OGX_MODELCAR_IMAGE`). The builder must be logged in to the stage registry (username
  `agentic-api`) so it can pull the image; the build **fails** if the pull fails. In GitHub
  Actions a `docker/login-action` step authenticates with the `REDHAT_STAGE_REGISTRY_TOKEN` repo
  secret; for a local build,
  `printf '%s' "$TOKEN" | podman login registry.stage.redhat.io --username agentic-api --password-stdin`.
- **`cache` (fork/Dependabot CI builds):** copies from `distribution/artifact-cache/models`
  staged in the build context, and **fails** (empty/missing `.cache` trees) if no cache is
  staged. `.github/actions/prefetch-artifact` restores that tree via `actions/cache` (keyed on
  the modelcar ref); non-fork runs extract the modelcar's `/models` and save it to keep the
  cache warm, and fork PRs can restore (but not save) the base branch's cache without registry
  access. The `models/` contents are gitignored (only `.gitkeep` is tracked).

The fork vs non-fork decision is made in the workflow (`IS_FORK`) and passed as the build arg.

The midstream **Konflux** build (`.tekton/`) always uses `pull`: the PipelineRuns set
`build-args: OGX_ARTIFACT_SOURCE=pull`. Because the modelcar is now pulled as a build stage
(not via a build secret), the stage-registry **pull secret** must be linked to the build
pipeline service account in the `open-data-hub-tenant` namespace so buildah can pull it.
Downstream (product) consumption is handled separately.

### Auto-Generated Files (do not edit manually)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/ogx-distribution](https://github.com/opendatahub-io/ogx-distribution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
