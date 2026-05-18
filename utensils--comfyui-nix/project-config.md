---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT WORKFLOW RULES

**ALWAYS run `git add` after making file changes!** The Nix flake requires this for proper operation.
**NEVER commit changes unless explicitly requested by the user.**

## Build/Run Commands

- **Run application**: `nix run` (default)
- **Run with browser**: `nix run -- --open` (automatically opens browser)
- **Run with CUDA**: `nix run .#cuda` (Linux/NVIDIA only, uses pre-built PyTorch CUDA wheels)
- **Run with ROCm**: `nix run .#rocm` (Linux/AMD only, uses pre-built PyTorch ROCm 7.1 wheels)
- **Run with Intel XPU**: `nix run .#xpu` (Linux/Intel only, uses pre-built PyTorch XPU wheels — oneAPI/SYCL, no IPEX)
- **Run with custom port**: `nix run -- --port=8080`
- **Run with network access**: `nix run -- --listen 0.0.0.0`
- **Run with debug logging**: `nix run -- --debug` or `nix run -- --verbose`
- **Build with Nix**: `nix build`
- **Check for updates**: `nix run .#update` (shows latest ComfyUI version and update instructions)
- **Develop with Nix**: `nix develop` (provides ruff, pyright, nixfmt, shellcheck)

### Docker Commands

- **Build Docker image**: `nix run .#buildDocker` (creates `comfy-ui:latest`)
- **Build CUDA Docker**: `nix run .#buildDockerCuda` (creates `comfy-ui:cuda`)
- **Build ROCm Docker**: `nix run .#buildDockerRocm` (creates `comfy-ui:rocm`)
- **Build XPU Docker**: `nix run .#buildDockerXpu` (creates `comfy-ui:xpu`)
- **Cross-build Linux images from macOS**: `nix run .#buildDockerLinux`, `nix run .#buildDockerLinuxCuda`, `nix run .#buildDockerLinuxRocm`, `nix run .#buildDockerLinuxXpu`, `nix run .#buildDockerLinuxArm64`
- **Pull pre-built**: `docker pull ghcr.io/utensils/comfyui-nix:latest` (or `:latest-cuda`, `:latest-rocm`, `:latest-xpu`)
- **Run container**: `docker run -p 8188:8188 -v $PWD/data:/data comfy-ui:latest`
- **Run CUDA container**: `docker run --gpus all -p 8188:8188 -v $PWD/data:/data comfy-ui:cuda`
- **Run ROCm container**: `docker run --device /dev/kfd --device /dev/dri -p 8188:8188 -v $PWD/data:/data comfy-ui:rocm`
- **Run XPU container**: `docker run --device /dev/dri -p 8188:8188 -v $PWD/data:/data comfy-ui:xpu`

## Linting and Code Quality

- **Run all checks**: `nix flake check` — this is the quality gate (there are no unit tests)
- **Individual checks run by `nix flake check`**:
  - `ruff-check`: Lint Python with ruff (`ruff check --no-cache src/`)
  - `pyright-check`: Type-check Python with pyright
  - `nixfmt`: Verify Nix formatting (nixfmt-rfc-style)
  - `shellcheck`: Lint shell scripts in `scripts/`
  - `package`: Full build of the default package
- **Manual linting in dev shell** (`nix develop`):
  - `ruff check src/` — lint Python (or `ruff check src/path/to/file.py` for a single file)
  - `ruff format src/` — format Python (or `ruff format src/path/to/file.py`)
  - `pyright src/` — type-check Python (or `pyright src/path/to/file.py`)
  - `shellcheck scripts/*.sh` — lint shell scripts (or `shellcheck scripts/path/to/script.sh`)
  - `nix fmt` — format Nix files

## Version Management

- Current ComfyUI version: v0.19.3 (pinned in `nix/versions.nix`; released 2026-04-17)
- To update ComfyUI: modify `version`, `rev`, and `hash` in `nix/versions.nix`
- Vendored wheels (spandrel, frontend, docs, etc.) also pinned in `nix/versions.nix`
- Template input files: auto-generated in `nix/template-inputs.nix`
  - Update with: `./scripts/update-template-inputs.sh && git add nix/template-inputs.nix`
- Python version: 3.12
- PyTorch: macOS uses pre-built wheels (2.5.1, pinned to work around MPS bugs on macOS 26); CUDA uses pre-built wheels from pytorch.org (cu128); ROCm uses pre-built wheels from pytorch.org (rocm7.1); Intel XPU uses pre-built wheels from pytorch.org (xpu, oneAPI/SYCL — no IPEX); Linux CPU uses nixpkgs

## Project Architecture

### Directory Structure

- **src/custom_nodes/**: Bundled custom node extensions (model_downloader, etc.)
- **src/patches/**: Runtime Python monkey-patches applied to ComfyUI internals (imported by the launcher's `sitecustomize.py`)
- **nix/**: Flake modules and helpers
  - **versions.nix**: All version pins (ComfyUI, vendored wheels, custom nodes, PyTorch wheels)
  - **packages.nix**: Package definitions and the inline launcher script (`writeShellApplication`)
  - **python-overrides.nix**: Python package overrides (platform-specific PyTorch, dependency fixes)
  - **vendored-packages.nix**: Wheel-based packages (spandrel, frontend, docs, etc.) built from pins in `versions.nix`
  - **template-inputs.nix**: Pre-fetched template input files (auto-generated, do not edit manually)
  - **apps.nix**: Flake app definitions (run, update, Docker build commands)
  - **docker.nix**: Docker image builders
  - **checks.nix**: CI check definitions (ruff, pyright, nixfmt, shellcheck)
  - **custom-nodes.nix**: Bundled custom node package definitions
  - **patches/**: Build-time `.patch` files applied to upstream ComfyUI source (distinct from `src/patches/`, which are runtime Python patches)
  - **modules/comfyui.nix**: NixOS service module with declarative custom nodes
  - **lib/custom-nodes.nix**: Helper functions for custom node management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utensils/comfyui-nix](https://github.com/utensils/comfyui-nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
