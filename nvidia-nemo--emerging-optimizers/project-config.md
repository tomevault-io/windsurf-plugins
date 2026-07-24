---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`emerging_optimizers` is an NVIDIA research package implementing matrix-preconditioned and orthogonalization-based optimizers (Muon, SOAP, PSGD, Lion, Scion, etc.) for LLM training, plus supporting Triton kernels and EVD/Sinkhorn utilities. Python 3.12+, PyTorch 2.0+. Dependency management is **uv** (`uv.lock` is committed).

## Environment setup

The canonical environment is the NGC PyTorch container (`docker/Dockerfile.ci` builds from `nvcr.io/nvidia/pytorch:26.02-py3`); torch, triton, and the CUDA libs come from the base image, **not** uv. The Dockerfile is the source of truth for how to bring up an env that matches CI:

```
# Inside the NGC pytorch container:
export PIP_CONSTRAINT=""                  # NGC sets a pip constraint that breaks uv resolution
export UV_PROJECT_ENVIRONMENT=/opt/venv
uv venv --system-site-packages "$UV_PROJECT_ENVIRONMENT"
uv sync --link-mode symlink --locked --all-groups \
    --no-install-package absl-py --no-install-package torch \
    --no-install-package triton \
    --no-install-package nvidia-cublas-cu12 --no-install-package nvidia-cuda-cupti-cu12 \
    --no-install-package nvidia-cuda-nvrtc-cu12 --no-install-package nvidia-cuda-runtime-cu12 \
    --no-install-package nvidia-cudnn-cu12 --no-install-package nvidia-cufft-cu12 \
    --no-install-package nvidia-cufile-cu12 --no-install-package nvidia-curand-cu12 \
    --no-install-package nvidia-cusolver-cu12 --no-install-package nvidia-cusparse-cu12 \
    --no-install-package nvidia-cusparselt-cu12 --no-install-package nvidia-nccl-cu12 \
    --no-install-package nvidia-nvjitlink-cu12 --no-install-package nvidia-nvtx-cu12
```

Key points:

- `--system-site-packages` is required so the venv inherits the container's torch/CUDA — without it `uv sync` will try to install incompatible CUDA wheels.
- The `--no-install-package` allowlist (torch, triton, all `nvidia-*-cu12` wheels, plus absl-py which the container also pins) prevents uv from clobbering the container's stack. If you add a dep that pulls in a new CUDA wheel, extend this list.
- `--all-groups` installs the `dev`, `test`, and `docs` groups together; use this rather than `uv sync` alone.
- `--link-mode symlink` is used because the bind-mount layout makes hardlinks fail.
- Outside the NGC container (e.g. local laptop), drop `--system-site-packages` and the `--no-install-package` flags; uv will install vanilla torch from PyPI.

## Common commands

Use `uv run <cmd>` so commands resolve against the locked environment.

- **Lint / format**: `uv run ruff check --fix .` and `uv run ruff format .` — CI does **not** auto-fix; pre-commit runs both.
- **Type check**: mypy is configured strictly for the `emerging_optimizers/` tree (`disallow_untyped_defs = True`); tests, benchmarks, docker, and docs are excluded. Pre-commit runs mypy with the same scope.
- **Pre-commit**: `uv run pre-commit run --all-files`. Note the `no-underscore-md` local hook — Markdown filenames must use hyphens, not underscores (except under `.github/`).
- **Add a dependency**: `uv add <pkg>` for required, `uv add --optional --extra <extra> <pkg>` for optional, `uv add --group <group> <pkg>` for dev/docs/test groups. Commit `uv.lock` and `pyproject.toml` together.

## Tests

Tests use `absl.testing` (not pytest) and accept absl flags. Standard form:

```
python tests/test_<name>.py --device={cpu|cuda} [--seed=42] -v -2 --xml_output_file=<path>.xml
```

Distributed CPU tests are launched via `torchrun`:

```
torchrun --nproc_per_node=<n> --no-python python tests/test_distributed_muon_utils_cpu.py --device=cpu -v -2
```

CI test suites are encoded as shell scripts in `tests/ci/`:

- `L0_Tests_CPU.sh` — all `tests/test_distributed_*_cpu.py` at 8 and 4 ranks (rekls also at 1 rank) + a couple of CPU-only tests.
- `L0_Tests_GPU.sh` — every `tests/test_*.py` (excluding `*_cpu.py`) on `--device=cuda`, run twice (random seed, then `--seed=42`), plus the `tests/convergence/*_test.py` set.
- `L1_Tests_GPU.sh` — convergence tests at `--seed=77` plus the `moe_c4_convergence.py` runs against muon/soap with a loss target.

Coverage is collected with `coverage run -p --source=emerging_optimizers ...` and aggregated; configuration in `pyproject.toml` (`[tool.coverage.*]`) excludes Triton-decorated code, `if TYPE_CHECKING`, and abstract methods.

Run a single test class/method using absl conventions, e.g. `python tests/test_orthogonalized_optimizer.py OrthogonalizedOptimizerTest.test_smoke --device=cpu -v -2`.

### Authoring tests

- **Every test file must define `--device` and `--seed` flags.** CI invokes every test under both `--device=cpu` and `--device=cuda`, and runs each GPU test twice (random seed, then `--seed=42` / `--seed=77`). Tests without these flags will be invoked with `--device=...` and fail at flag parsing. Match the existing pattern: `flags.DEFINE_enum("device", "cpu", ["cpu", "cuda"], ...)`, `flags.DEFINE_integer("seed", None, ...)`, and a `setUpModule` that seeds when `FLAGS.seed is not None`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/Emerging-Optimizers](https://github.com/NVIDIA-NeMo/Emerging-Optimizers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
