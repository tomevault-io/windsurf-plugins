---
trigger: always_on
description: > **Scoped instructions** for C++, build, CI, release, security, and testing live in
---

# ExplorerLens — Copilot Instructions

> **Scoped instructions** for C++, build, CI, release, security, and testing live in
> `.github/instructions/*.instructions.md`. Read those files before making changes in their domain.

## Project Overview

ExplorerLens is a **Windows Shell Extension** (IThumbnailProvider COM DLL) that generates
GPU-accelerated thumbnails for 200+ file formats across 25 specialized decoders.

- **Version:** 40.0.1 (Codename: Procyon)
- **Language:** C++20 (MSVC v145 toolset, Visual Studio 18 2026)
- **Build System:** CMake 3.25+ with presets (Engine) + MSBuild (Shell/Manager)
- **Preferred Compiler:** MSVC cl.exe 19.50 (v145 toolset) — **never use Clang for production builds**
- **GPU:** CPU decode with GDI+ fallback · DirectX 11/12/Vulkan GPU planned (Phase 2+)
- **Platforms:** Windows (IThumbnailProvider), macOS Quick Look (stub), Linux Nautilus (stub)
- **COM CLSID:** `9E6ECB90-5A61-42BD-B851-D3297D9C7F39`
- **Build Status:** 0 errors, 0 warnings

## Architecture

```text
LENSShell.dll (2940 KB) — COM Shell Extension (IThumbnailProvider)
LENSManager.exe (400 KB) — GUI Configuration Utility
ExplorerLensEngine.lib — Core decode + render pipeline
```

## AI Tooling Surface

ExplorerLens is configured for the current 2026 GitHub Copilot and VS Code agent workflow surface.

| Asset | Location | Role |
| ------- | ---------- | ------ |
| Repository rules | `.github/copilot-instructions.md` | Primary project contract for agents and Copilot |
| Scoped instructions | `.github/instructions/*.instructions.md` | Pattern-specific rules for CI, tests, versions, size policy, and workspace behavior |
| Custom agents | `.github/agents/*.agent.md` | 5 repo-specialized agents (ExplorerLens, Docs, Release, TestCorpus, CI-Ops) + Explore |
| Prompt templates | `.github/prompts/*.prompt.md` | 14 reusable prompts for review, tests, scaffolding, release, debug, CI, and diagrams |
| Repository skills | `.github/skills/*/SKILL.md` | 7 focused task playbooks for build, docs, decoders, corpus, perf, workflows, ci-ops |
| Capability reference | `.github/standards/ai-tooling-capabilities.md` | Canonical inventory for instructions, agents, prompts, skills, MCP servers, and workflow coverage |
| MCP configuration | `.vscode/mcp.json` | Workspace MCP servers for GitHub, filesystem, and docs-scoped editing |

### MCP Servers (Workspace)

- `github` — GitHub API operations through `@modelcontextprotocol/server-github`
- `filesystem` — full workspace filesystem scope through `@modelcontextprotocol/server-filesystem`
- `project-docs` — docs-only filesystem scope for `.github/` and `docs/`

When changing AI-facing repo files, keep `.github/standards/ai-tooling-capabilities.md` synchronized with the actual assets and workflow inventory.

### Key Directories

| Directory | Purpose |
| ------------------------------ | --------------------------------------------------------------------------------- |
| `LENSShell/` | Shell extension DLL (COM registration, thumbnail provider) |
| `LENSManager/` | WTL-based admin GUI for registration/settings |
| `Engine/` | Core library — decoders, GPU pipeline, caching, observability |
| `Engine/Core/` | Decode pipeline, GPU renderer, resource management |
| `Engine/Decoders/` | Format-specific decoders (25+ total, incl. CAD/glTF/Scientific) |
| `Engine/Plugin/` | Plugin ecosystem (trust chain, sandbox, compat kit, ref pack) |
| `Engine/Memory/` | Memory management (compactor, hot-mode, pressure controller, footprint optimizer) |
| `Engine/Pipeline/` | Pipeline stages (fallback engine, zero-copy upload, parallel I/O) |
| `Engine/Cache/` | Cache management (adaptive budget, PSO cache, sub-ms cache, multi-tenant) |
| `Engine/Utils/` | Utilities (matrix validation, installer lifecycle) |
| `Engine/Tests/` | Unit tests + Google Benchmark |
| `Engine/AI/` | AI/ML modules (scene understanding, smart crop, IQA, search) |
| `Engine/GPU/` | GPU decode acceleration (NVDEC/QuickSync/AMF vendor routing) |
| `Engine/Media/` | Live preview scrubber (video frame extraction, timeline, cache) |
| `Engine/Platform/` | Platform Abstraction Layer — Win32, macOS QL, Linux Nautilus |
| `build-scripts/` | PowerShell build automation |
| `build-scripts/core/` | Build-Library-Core.ps1 — unified build module |
| `build-scripts/external-libs/` | Per-library build scripts (zlib, LZ4, zstd, etc.) |
| `cmake/` | CMake toolchain files |
| `packaging/` | MSI (WiX), Inno Setup, MSIX manifests |
| `SDK/` | Plugin SDK (C ABI, plugin_api.h) |
| `docs/` | All documentation |
| `.github/workflows/` | CI/CD pipelines |

## Build Quick Reference

```powershell
.\build-scripts\Build-MSVC.ps1          # build (sources vcvars64 automatically)
.\build-scripts\Build-MSVC.ps1 -Clean -Test   # clean build + tests
ctest --test-dir build -C Release --output-on-failure
```

> Full toolchain tables, presets, and external library inventory → `.github/instructions/build.instructions.md`

## Key Types

- `LENSArchive` — Main archive handler, routes formats to decoders
- `LENSTYPE` — Enum of supported archive/format types (LENSArchive.h)
- `IThumbnailProvider` — Windows COM interface implemented by LENSShell
- `ExplorerLensEngine` — Core engine library linking all decoders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RajwanYair/ExplorerLens.io](https://github.com/RajwanYair/ExplorerLens.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
