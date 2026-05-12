---
trigger: always_on
description: **Important**: Do NOT commit or push changes without explicit user permission.
---

# TXL (Triton Xtra Language) - Project Workflow

**Important**: Do NOT commit or push changes without explicit user permission.

## Initial Build

First time building TXL wheel for Linux:

```bash
# 1. Initialize git submodules
git submodule update --init --recursive

# 2. Download LLVM (if not already present)
# Place llvm-7d5de303-ubuntu-x64/ in project root

# 3. Build Docker image and create wheel
./tools/build-wheel-docker.sh -n

# Wheel will be in: output/txl-3.5.1-cp312-cp312-manylinux_2_35_x86_64.whl
```

Build flags:
- `-n` - Apply TXL patches to Triton (run once, first time only)
- `-j N` - Number of parallel jobs (default: 8)
- `-c` - Clean build directories before build
- `--no-cache` - Rebuild Docker image without cache

## Rebuild

After making code changes, use incremental rebuild (much faster):

```bash
# Fast incremental rebuild (uses ninja, only rebuilds changed files)
./tools/build-wheel-docker.sh -r

# Rebuild with clean (full rebuild)
./tools/build-wheel-docker.sh -r -c
```

Notes:
- Conda environment is persisted in `txl-conda/` directory
- Build artifacts are persisted in `thirdparty/triton/build/`
- Uses clang by default (less memory than gcc)

## Test

Test TXL wheel on Modal's cloud H100 GPUs:

```bash
# Run test with default volume (txl-dump)
./tools/modal_tests.sh flash_attention.py
./tools/modal_tests.sh mla_decoding.py
./tools/modal_tests.sh nsa_prefill.py

# Run with custom test name
./tools/modal_tests.sh flash_attention.py my-test

# Run with custom volume name
./tools/modal_tests.sh flash_attention.py my-test txl-dump

# Output files:
# - docker/dumps/{test_name}_{timestamp}.log - Console output
# - docker/dumps/{test_name}_{timestamp}/ - Dump files (kernel caches)
```

Available Modal test scripts:
- `docker/flash_attention.py` - Flash attention benchmark
- `docker/mla_decoding.py` - MLA decoding benchmark
- `docker/nsa_prefill.py` - NSA prefill benchmark (1800s timeout)

### Debug Environment Variables

Pass debug environment variable to Modal container:

```bash
# Run with TXLGPU pipeliner debug
TRITON_LLVM_DEBUG_ONLY=txlgpu-pipeliner ./tools/modal_tests.sh nsa_prefill.py debug-test txl-dump
```

The debug output will be in `docker/dumps/{test_name}_{timestamp}.log`.

Notes:
- All tests save dump files to Modal volume `txl-dump`
- Dump files are automatically downloaded after test completes
- Use `--force` flag in volume get to overwrite existing local directories

## Debug Tips

### Local Debug (if you have GPU)
```bash
TRITON_LLVM_DEBUG_ONLY="triton-gpu-taskid-propagate" \
TRITON_KERNEL_DUMP=1 \
TRITON_DUMP_DIR=dump \
TRITON_ALWAYS_COMPILE=1 \
python python/txl/tests/fused-attention.py
```

### CUDA Debug
```bash
CUDA_COREDUMP_SHOW_PROGRESS=1 \
CUDA_ENABLE_COREDUMP_ON_EXCEPTION=1 \
CUDA_LAUNCH_BLOCKING=1 \
cuda-gdb
```

### Memory Issues
If build runs out of memory:
```bash
# Reduce parallel jobs
./tools/build-wheel-docker.sh -j 4
```

### GLIBCXX Error
If you get 'GLIBCXX_3.4.30' not found:
```bash
conda install -c conda-forge gcc=12.1.0
```

## Other Notes

- **Don't push for every change** - Only push when user explicitly asks
- Use `./tools/build-wheel-docker.sh -r` for incremental rebuilds after code changes
- Use `./tools/build-wheel-docker.sh -r -c` for clean rebuilds when build issues occur
- The `-n` flag should only be used once when setting up the project for the first time
- LLVM and conda directories are excluded from git (too large)

## Code Development

### Important: Never modify patch/triton directly

This repo has two copies of Triton code:
- `thirdparty/triton/` - Git submodule (main Triton codebase) - **MODIFY HERE**
- `patch/triton/` - TXL patches applied to Triton - **DO NOT MODIFY**

**Correct workflow**:
1. Edit code in `thirdparty/triton/` (the submodule)
2. Build and test with `./tools/build-wheel-docker.sh -r`
3. Repeat steps 1-2 until fix is verified
4. Only when explicitly requested by user, copy changes to `patch/triton`:
   ```bash
   bash tools/cp_from_triton.sh
   ```
5. Commit the patch changes

### Workflow: Making Changes

1. **Edit in submodule** (thirdparty/triton):
   ```bash
   # Make changes to code in thirdparty/triton/
   ```

2. **Build and test**:
   ```bash
   ./tools/build-wheel-docker.sh -r
   # Run tests
   ```

3. **Repeat** until fix is verified

4. **Copy to patch/triton** (only when user requests):
   ```bash
   bash tools/cp_from_triton.sh
   ```

5. **Commit** (only when user requests)

### Scripts

- `tools/cp_to_triton.sh` - Copy patch/triton → thirdparty/triton
- `tools/cp_from_triton.sh` - Copy thirdparty/triton → patch/triton
- `tools/diff_triton.py` - Compare patch/triton vs thirdparty/triton

### Git Tips

- Remove trailing slashes from `cp -r` commands to avoid issues
- Submodule changes are tracked separately from main repo
- Use `.gitignore` patterns like `llvm-*`, `txl-conda/` for large build artifacts
- **Never modify patch/triton directly** - only copy from thirdparty/triton

## Debugging Pass Failures

When encountering `RuntimeError: PassManager::run failed`, follow this workflow:

### Step 1: Identify the Failing Pass

Run test with TXLGPU pipeliner debug flag to see which stage fails:

```bash
# Set the debug env var BEFORE running the test script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deciding/txl](https://github.com/deciding/txl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
