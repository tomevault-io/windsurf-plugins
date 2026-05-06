---
trigger: always_on
description: Provides common functionality for collecting release files and building OCI images. The
---

# ocibuild Development Guide

This document provides a comprehensive overview of the `ocibuild` project for continuing development.

## Project Overview

**ocibuild** is a pure Erlang library for building OCI-compliant container images programmatically, without requiring Docker or any container runtime. It's inspired by:

- .NET's `Microsoft.NET.Build.Containers`
- Google's `ko` (for Go)
- Java's `jib`

### Feature Comparison with Similar Tools

| Feature                   | ocibuild           | ko (Go)     | jib (Java)        | .NET Containers |
|---------------------------|--------------------|-------------|-------------------|-----------------|
| No Docker required        | ✅                 | ✅          | ✅                | ✅              |
| Push to registries        | ✅                 | ✅          | ✅                | ✅              |
| Layer caching             | ✅                 | ✅          | ✅                | ✅              |
| Tarball export            | ✅                 | ✅          | ✅                | ✅              |
| OCI annotations           | ✅                 | ✅          | ✅                | ✅              |
| Auto OCI annotations      | ✅ (VCS)           | ✅          | ✅                | ✅              |
| Build system integration  | ✅ (rebar3/Mix)    | ✅          | ✅ (Maven/Gradle) | ✅ (MSBuild)    |
| Multi-platform images     | ✅                 | ✅          | ✅                | ✅              |
| Reproducible builds       | ✅                 | ✅          | ✅                | ✅              |
| Smart dependency layering | ✅                 | N/A         | ✅                | ✅              |
| Non-root by default       | ✅                 | ✅          | ❌                | ✅              |
| SBOM generation           | ✅ (SPDX)          | ✅ (SPDX)   | ❌                | ✅ (SPDX)       |
| Image signing             | ✅ (cosign)        | ✅ (cosign) | ❌                | ❌              |
| Zstd compression          | ✅ (OTP 28+)       | ✅          | ❌                | ❌              |

Legend: ✅ Implemented | ⏳ Planned (P# = Priority) | ❌ Not implemented

**References:**
- [ko: Easy Go Containers](https://ko.build/)
- [jib - Build container images for Java](https://github.com/GoogleContainerTools/jib)
- [.NET SDK container creation](https://learn.microsoft.com/en-us/dotnet/core/containers/overview)

### Goals

1. **Zero dependencies** — Only OTP stdlib modules (crypto, zlib, inets, ssl, json)
2. **BEAM-universal** — Works from Erlang, Elixir, Gleam, LFE via hex.pm
3. **OCI compliant** — Produces standard OCI image layouts
4. **No Docker required** — Builds and pushes images directly to registries

### Target OTP Version

- Primary target: OTP 27+ (has built-in `json` module)

### Code style

- Always prefer using `maybe` instead of deeply nested `case...of`
- Prefer Markdown style comments instead of EDoc (deprecated).

---

## Architecture

### Module Structure

```
src/
├── ocibuild.erl              # Public API - the main interface users interact with
├── ocibuild.app.src          # OTP application spec
│
├── http/                     # HTTP/Registry operations
│   ├── ocibuild_http.erl         # Public facade for HTTP operations
│   ├── ocibuild_http_sup.erl     # OTP supervisor for HTTP workers
│   ├── ocibuild_http_pool.erl    # Coordinates parallel HTTP operations
│   ├── ocibuild_http_worker.erl  # Single-use worker (owns its httpc profile)
│   ├── ocibuild_registry.erl     # Registry client (pull/push via HTTP with retry)
│   └── ocibuild_cache.erl        # Layer caching for base images
│
├── oci/                      # OCI image building
│   ├── ocibuild_layer.erl        # OCI layer creation (tar + compression + digests)
│   ├── ocibuild_manifest.erl     # OCI manifest generation (with annotations)
│   ├── ocibuild_index.erl        # OCI image index for multi-platform images
│   ├── ocibuild_layout.erl       # OCI image layout export (directory/tarball)
│   ├── ocibuild_tar.erl          # In-memory TAR archive builder (POSIX ustar)
│   └── ocibuild_sbom.erl         # SPDX 2.2 SBOM generation
│
├── adapters/                 # Build system adapters
│   ├── ocibuild_adapter.erl      # Behaviour for adapters (rebar3, Mix, etc.)
│   ├── ocibuild_release.erl      # Shared release handling (layers, env, ports, etc.)
│   ├── ocibuild_rebar3.erl       # Rebar3 provider (implements ocibuild_adapter)
│   └── ocibuild_mix.erl          # Mix adapter (implements ocibuild_adapter)
│
├── vcs/                      # Version control
│   ├── ocibuild_vcs.erl          # VCS behaviour and detection
│   └── ocibuild_vcs_git.erl      # Git adapter (CI env vars + git commands)
│
└── util/                     # Utilities
    ├── ocibuild_compress.erl     # Compression abstraction (zstd on OTP 28+, gzip fallback)
    ├── ocibuild_digest.erl       # SHA256 digest utilities
    ├── ocibuild_json.erl         # JSON encode/decode (OTP 27 native + fallback)
    ├── ocibuild_time.erl         # Timestamp utilities (SOURCE_DATE_EPOCH)
    ├── ocibuild_validate.erl     # Shared validation (null bytes, path traversal)
    └── ocibuild_progress.erl     # Progress reporting

lib/
├── mix/tasks/ocibuild.ex         # Mix task (mix ocibuild command)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intility/erlang-oci-builder](https://github.com/intility/erlang-oci-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
