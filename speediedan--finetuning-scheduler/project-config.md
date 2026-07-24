---
trigger: always_on
description: **Fine-Tuning Scheduler (FTS)** is a PyTorch Lightning callback that enables flexible, multi-phase fine-tuning schedules. It allows users to define custom parameter unfreezing and optimizer configuration schedules through YAML configuration files.
---

# Copilot Instructions for Fine-Tuning Scheduler

## Repository Overview

**Fine-Tuning Scheduler (FTS)** is a PyTorch Lightning callback that enables flexible, multi-phase fine-tuning schedules. It allows users to define custom parameter unfreezing and optimizer configuration schedules through YAML configuration files.

**Key Technologies:**

- Python 3.9+ (CI tests on 3.10 and 3.13)
- PyTorch 2.7.0+ with PyTorch Lightning ecosystem
- Core deps: pytorch-lightning (standalone) or lightning (unified package), transformers

**Repository Size:** ~100 files, primarily Python, with YAML configs and shell scripts

## Lightning Package Support

FTS supports both standalone and unified Lightning packages:

- **Unified (default):** `lightning` package with `lightning.pytorch`
- **Standalone:** `pytorch_lightning` package

**USE_CI_COMMIT_PIN Environment Variable:**

- When set, installs Lightning from a git commit (specified in `requirements/ci/overrides.txt`)
- Default in dev/CI builds for consistent testing against latest Lightning changes
- Can be disabled with `--no-commit-pin` flag in build scripts

## Code Standards

### Required Before Each Commit

- Run tests in your local environment and ensure all tests are passing:

```bash
cd /home/speediedan/repos/finetuning-scheduler && python -m pytest src/finetuning_scheduler tests -v
```

- Ensure all pre-commit hooks pass.
- If the copilot session is still failing despite trying to get tests and pre-commit hooks passing for some time, it's okay to commit your intermediate work with a comment about the present challenge to be dealt with in a subsequent session.

### Requirement for Each Pull Request

- All pull requests must pass the CI checks.
- Ensure that the code is well-documented, with docstrings for all public functions and classes.
- Write unit tests for new functionality and ensure existing tests pass.
- Ensure the cpu coverage reported by our `ci_test-full.yml` workflow is >= the existing coverage.

## Build and Validation Commands

### Environment Setup

Development environment uses `uv` for fast, reliable dependency management:

Set environment context variables (developer-specific paths):

```bash
export FTS_VENV_BASE=/mnt/cache/${USER}/.venvs
export FTS_TARGET_VENV=fts_latest
export FTS_REPO_DIR=${HOME}/repos/finetuning-scheduler  # Example: adjust to your local repo path
```

```bash
# Install uv (one-time setup)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create development environment (creates traditional venv)
# The build script handles Lightning commit pinning and optional PyTorch nightly automatically
./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest

# Activate the environment
cd ${FTS_REPO_DIR} && \
source ${FTS_VENV_BASE}/${FTS_TARGET_VENV}/bin/activate

# Run commands directly (no need for 'uv run')
python --version
python -m pytest tests/
```

**Manual installation (without build script):**

```bash
cd ${FTS_REPO_DIR}

# For manual installs, set UV_OVERRIDE to use the pinned Lightning commit
export UV_OVERRIDE=${PWD}/requirements/ci/overrides.txt
uv pip install -e ".[all]"
```

**Manual installation with a PyTorch prerelease (nightly/test):**

```bash
cd ${FTS_REPO_DIR}

# Install a PyTorch prerelease first (adjust version and CUDA target as needed; see configuration in requirements/ci/torch-pre.txt)
# Example (nightly):
uv pip install --prerelease=allow torch==2.11.0.dev20260121 --index-url https://download.pytorch.org/whl/nightly/cu130

# Then install FTS with Lightning commit pin
export UV_OVERRIDE=${PWD}/requirements/ci/overrides.txt
uv pip install -e ".[all]"
```

# To configure PyTorch prerelease used by the build scripts and azure pipelines, edit `requirements/ci/torch-pre.txt`:

# Line 1: torch version (e.g., 2.11.0 for test/RC or 2.11.0.dev20260121 for nightly)

# Line 2: CUDA target for local builds (e.g., cu130) — CI uses cpu

# Line 3: channel type: "test" or "nightly"

### Development Environment Scripts

Use the provided build script for automated setup:

```bash
# Standard development build (uses FTS_VENV_BASE or default ~/.venvs)
./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest

# Build with explicit venv directory (recommended for hardlink performance)
./scripts/build_fts_env.sh --repo-home=${HOME}/repos/finetuning-scheduler --target-env-name=fts_latest --venv-dir=/mnt/cache/${USER}/.venvs

# Build without Lightning commit pinning (use PyPI release)
./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest --no-commit-pin

# Build with specific PyTorch nightly version
./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest --torch_dev_ver=dev20240201

# Build with PyTorch test channel
./scripts/build_fts_env.sh --repo-home=${PWD} --target-env-name=fts_latest --torch_test_channel

# Build from Lightning source
./scripts/build_fts_env.sh --repo-home=${HOME}/repos/finetuning-scheduler --target-env-name=fts_latest --from-source="lightning:${HOME}/repos/lightning"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speediedan/finetuning-scheduler](https://github.com/speediedan/finetuning-scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
