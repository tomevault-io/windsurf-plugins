---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

torchcomms is an experimental collective communications API for PyTorch, providing pluggable backends (NCCL, NCCLX, Gloo, RCCL, RCCLX, XCCL) for distributed GPU/CPU computing. It includes a compatibility layer (`distwrap`) that serves as a drop-in replacement for `torch.distributed`.

## Environment Setup

Prerequisites: Python 3.10+, CMake 3.22+, Ninja 1.10+, CUDA toolkit (for GPU backends).

### Using uv (preferred)

```bash
uv venv --python 3.10
source .venv/bin/activate

# Install PyTorch nightly with CUDA 12.6
uv pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cu126

# Other CUDA versions available:
# uv pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cu130
# uv pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cu132

# Install build and dev dependencies (setuptools/packaging needed for --no-build-isolation)
uv pip install setuptools packaging pyyaml
uv pip install pytest numpy psutil lintrunner parameterized pydot

# Install torchcomms from source (default: NCCL + NCCLX + Gloo + Transport)
uv pip install --no-build-isolation -v .
```

### Using conda

```bash
conda create -n torchcomms python=3.10
conda activate torchcomms

# Install PyTorch nightly with CUDA
pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cu126

# Install required native deps (glog, gflags, fmt are needed for all builds; nccl for NCCL backend)
conda install conda-forge::glog=0.4.0 conda-forge::gflags conda-forge::fmt conda-forge::nccl -y

# Install torchcomms from source (USE_SYSTEM_LIBS=1 required with conda to link shared libs)
pip install -r requirements.txt
USE_NCCLX=OFF USE_SYSTEM_LIBS=1 pip install --no-build-isolation -v .
```

### Installing from PyPI (nightly wheels, no source build needed)

```bash
pip install --pre torch torchcomms --index-url https://download.pytorch.org/whl/nightly/cu126
```

## Building with Different Backends

Backend selection is controlled by environment variables (ON/OFF or 1/0) set before `pip install`:

| Variable | Default | Description |
|---|---|---|
| `USE_NCCL` | ON | Standard NCCL (uses PyTorch's bundled library, no extra build needed) |
| `USE_NCCLX` | ON | Meta's extended NCCL fork (built from vendored source in `comms/ncclx/`) |
| `USE_GLOO` | ON | CPU backend |
| `USE_RCCL` | OFF | AMD ROCm |
| `USE_RCCLX` | OFF | Meta's extended RCCL |
| `USE_XCCL` | OFF | Intel XPU |
| `USE_TRANSPORT` | ON (OFF on ROCm) | RDMA transport layer |
| `USE_SYSTEM_LIBS` | unset | When set, uses conda/system libs instead of building from source |

### NCCL-only (fastest build — skips NCCLX third-party dep compilation)

```bash
USE_NCCLX=OFF pip install --no-build-isolation -v .
```

### NCCLX build

`build_ncclx.sh` is the main build script. It builds ~20 third-party dependencies from source (fmt, zlib, boost, openssl, glog, gflags, folly, fbthrift, etc.), generates nccl_cvars files, builds the comms tracing service, and then compiles the vendored NCCL fork via `make`.

```bash
# Build everything from source (recommended for first time):
./build_ncclx.sh

# Or use conda/system libs for third-party deps:
USE_SYSTEM_LIBS=1 ./build_ncclx.sh

# Clean rebuild:
CLEAN_BUILD=1 ./build_ncclx.sh

# Skip third-party dep rebuild (if already built):
NCCL_BUILD_SKIP_DEPS=1 ./build_ncclx.sh
```

Key env vars for `build_ncclx.sh`:
- `CUDA_HOME` — CUDA installation (default: `/usr/local/cuda`)
- `NVCC_ARCH` — GPU architectures (default: `a100,h100`; auto-adds `b200` if CUDA 12.8+)
- `BUILDDIR` — build output directory (default: `build/ncclx`)
- `NCCL_HOME` — NCCLX source directory (default: `comms/ncclx/stable`)

### RCCL build (AMD ROCm)

```bash
export ROCM_HOME=/opt/rocm
export RCCL_INCLUDE=$ROCM_HOME/include/rccl
./build_rccl.sh
```

### RCCLX build (AMD ROCm, Meta extended)

```bash
conda install conda-forge::glog=0.4.0 conda-forge::gflags conda-forge::fmt -y
export BUILD_DIR=${PWD}/comms/rcclx/develop/build/release/build
export ROCM_HOME=/opt/rocm
export RCCLX_INCLUDE=${BUILD_DIR}/include/rccl
export RCCLX_LIB=${BUILD_DIR}/lib

# Narrow to your GPU arch to speed up build:
./build_rcclx.sh --amdgpu_targets gfx942  # MI300X/MI325X
./build_rcclx.sh --amdgpu_targets gfx950  # MI350X/MI355X
```

### XCCL build (Intel XPU)

```bash
source $INTEL_ONEAPI/compiler/latest/env/vars.sh
source $INTEL_ONEAPI/ccl/latest/env/vars.sh
USE_XCCL=ON USE_NCCL=OFF USE_NCCLX=OFF USE_TRANSPORT=OFF pip install --no-build-isolation -v .
```

### Install after backend build

For non-default backend combinations, disable the backends you don't need:

```bash
# Example: RCCLX-only
USE_NCCL=OFF USE_NCCLX=OFF USE_GLOO=OFF USE_RCCL=OFF USE_RCCLX=ON USE_TRANSPORT=OFF pip install --no-build-isolation -v .
```

### CMake direct build (C++ only, no Python package)

```bash
cmake -G Ninja -B build -DBUILD_TESTS=ON -DUSE_NCCL=ON -DUSE_NCCLX=OFF
cmake --build build
```

## Linting

Linting uses `lintrunner` configured in `.lintrunner.toml` with three linters:

- **CLANGFORMAT** — C++ formatting (`comms/torchcomms/**/*.{hpp,cpp}`) via `clang-format==21.1.2`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meta-pytorch/torchcomms](https://github.com/meta-pytorch/torchcomms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
