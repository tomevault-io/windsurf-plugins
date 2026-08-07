---
trigger: always_on
description: vvm (vLLM Version Manager) is a Rust CLI tool that manages multiple vLLM installations in isolated Python virtual environments. Inspired by nvm/fnm for Node.js. Each version gets its own venv under `~/.vvm/versions/`.
---

# AGENTS.md

## Project Overview

vvm (vLLM Version Manager) is a Rust CLI tool that manages multiple vLLM installations in isolated Python virtual environments. Inspired by nvm/fnm for Node.js. Each version gets its own venv under `~/.vvm/versions/`.

## Build & Test

```bash
# Build release binary
cargo build --release

# Run all tests
cargo test

# Lint (CI runs this with -D warnings)
cargo clippy --all-targets --all-features -- -D warnings

# Format + manifest sort (CI enforces both)
cargo fmt --all
cargo sort -w

# Build + install to ~/.local/bin + configure shell integration
./install.sh
```

Install the pre-commit hook once per clone so unformatted code never reaches CI:

```bash
git config core.hooksPath .githooks
```

## Architecture

~7,600 lines of Rust across 24 files. Single binary, no runtime dependencies beyond `uv` and `python3`.

### Source Layout

- **`src/main.rs`** — CLI entry point. Clap derive-based command dispatch (14 subcommands + 1 hidden). `Cli` struct must be `pub` for clap_complete in init.rs. `Uninstall` takes `Vec<String>` for multi-version support. `exec_with_version` resolves `pip`/`pip3` to `uv pip` and falls back to `python -m {cmd}` for commands absent from `venv/bin`.
- **`src/config.rs`** — `Config` struct managing `$VVM_DIR` paths (versions/, aliases/, repos/, cache/, current symlink).
- **`src/errors.rs`** — `VvmError` enum via thiserror. All commands return `Result<(), VvmError>`.
- **`src/cuda.rs`** — CUDA variant detection and `CudaVariant` enum (cu128/cu129/cu130/cpu). Detection order: `$VVM_CUDA_VARIANT` → nvidia-smi → `/usr/local/cuda/version.json` (or `$CUDA_HOME`) → `nvcc --version` → **cpu** when none of them report CUDA. A detected-but-unmapped major (CUDA 11 and older, or a future 14+) maps to the newest variant, cu130. `has_gpu_access()` checks nvidia-smi for login-node detection. `detect_gpu_arch_list()` queries nvidia-smi for compute capabilities (used by `--compile` to set `TORCH_CUDA_ARCH_LIST`).
- **`src/venv.rs`** — Python venv creation via `uv venv` and command execution. `run_in_venv_with_env` for passing extra env vars (used by `--path` installs).
- **`src/fsutil.rs`** — NFS-tolerant directory removal. `std::fs::remove_dir_all` can fail with ENOTEMPTY on NFS even after every entry is unlinked (stale attribute caches, `.nfs*` silly-rename files from another client). Renames the tree to a hidden sibling first — one atomic rename frees the caller's path immediately — then removes with retries.
- **`src/metadata.rs`** — `VersionMetadata` (serde JSON) stored as `metadata.json` per installed version. Fields: type, name, version, commit, pr, branch, source_path, cuda, installed_at, arch_list (TORCH_CUDA_ARCH_LIST of compiled kernels; local-compiled installs only).
- **`src/github.rs`** — GitHub API client (reqwest blocking). PR→commit, branch→commit, releases list, recent commits (by branch or with `until` timestamp), commit SHA resolution. Token resolved from `$GITHUB_TOKEN` → `$GH_TOKEN` → `gh auth token`, cached per process.
- **`src/resolve/specifier.rs`** — `VersionSpecifier` parser: release, commit:, branch:, pr:, nightly. `dir_name()` sanitizes `/` → `-` in branch names. Rejects refs that would escape the versions directory when interpolated into a path.
- **`src/commands/install.rs`** — Core install logic. Install paths: `install_from_wheel` (release), `install_nightly_with_fallback` (nightly), `install_branch_with_fallback` (branch), `install_from_local` (--path), `install_from_local_compiled` (--path --compile), `install_from_source` (--source), `install_from_pr` (pr). `find_wheel_url` fetches metadata.json for direct wheel URL. `verify_installation` skipped on login nodes. `install_from_local_compiled` auto-detects GPU arch, clears stale CMake cache, and saves build log. Also holds the cross-node install lock and `--repo` clone/fetch caching.
- **`src/commands/install_extra.rs`** — Bundled recipes for auxiliary libraries that can't install via plain pip: `deepgemm`, `flashinfer` (source build + matching cubin/jit-cache companions, or prebuilt nightly wheels), `deepep` (NVSHMEM download + build). All three default their ref to the active vllm's pin (see below). Owns the wheel cache under `~/.vvm/cache/wheels/`.
- **`src/commands/use_cmd.rs`** — Updates `current` symlink. Tries raw dir name first (for local-* names), then parses as specifier. Reads `.vvmrc` with upward directory walk. Hints user to `eval "$(vvm env)"` if shell wrapper not active.
- **`src/commands/env.rs`** — Outputs shell-evaluable `export` statements for PATH/VIRTUAL_ENV/VVM_DIR. Strips previous vvm bin from PATH before prepending new one.
- **`src/commands/init.rs`** — Generates shell wrapper function + tab completions (via clap_complete) for bash/zsh/fish. Dynamic completion for `use`/`uninstall`/`exec` via hidden `list-installed` subcommand.
- **`src/commands/alias.rs`** — Create/list/show/remove aliases (symlinks in aliases/ dir).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vllm-project/vvm](https://github.com/vllm-project/vvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
