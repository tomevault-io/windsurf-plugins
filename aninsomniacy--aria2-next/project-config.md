---
trigger: always_on
description: This file defines repository rules for AI coding agents. Human contributors should start with [README.md](README.md), [packaging/README.md](packaging/README.md), [tools/README.md](tools/README.md), [third_party/README.md](third_party/README.md), and [docs/maintenance/README.md](docs/maintenance/README.md).
---

# AGENTS.md - aria2-next

This file defines repository rules for AI coding agents. Human contributors should start with [README.md](README.md), [packaging/README.md](packaging/README.md), [tools/README.md](tools/README.md), [third_party/README.md](third_party/README.md), and [docs/maintenance/README.md](docs/maintenance/README.md).

> [!IMPORTANT]
> All changes must meet industrial-grade quality. Find the root cause before changing code, keep behavior compatible unless the task explicitly changes it, avoid unrelated churn, and verify the exact build or release path affected by the change.

## Architecture

| Area | Ownership |
| --- | --- |
| Core | C99 and C++11 aria2 command-line client and core implementation |
| Build | CMake 3.25+ with Ninja as the default generator |
| Tests | CTest plus the CppUnit test suite |
| Packaging | Cross-platform release automation under `packaging/` and `.github/workflows/release.yml` |
| Third-party source | Vendored `third_party/wslay` with local ownership rules |

## Key Paths

| Path | Purpose |
| --- | --- |
| `CMakeLists.txt` | Project declaration and CMake module entry point |
| `CMakePresets.json` | Standard configure, build, and test presets |
| `cmake/modules/` | Build options, probes, dependencies, targets, tests, and summary output |
| `cmake/Sources.cmake` | Core source inventory |
| `cmake/TestSources.cmake` | Test source inventory |
| `src/` | aria2 core, CLI, protocol, disk, RPC, and platform code |
| `src/includes/aria2/` | Public libaria2 headers |
| `tests/` | CppUnit tests and fixtures |
| `docs/` | Manual sources, completion tooling, and maintenance records |
| `packaging/` | Release dependencies, Dockerfiles, cross-build scripts, and package assets |
| `packaging/dependencies.env` | Release dependency version source |
| `third_party/` | Bundled third-party source |
| `tools/` | Local developer helpers |

## Build Rules

CMake is the only supported build system for aria2-next. Do not restore Autotools files, add another maintained build system, or route release packaging through removed upstream build scripts.

Keep the top-level `CMakeLists.txt` small. New build logic belongs in the focused modules under `cmake/modules/`. Keep generated config headers, package metadata, CMake files, Ninja files, test output, binaries, and archives out of the source tree.

Feature probes must match the source they guard. Prefer `check_symbol_exists`, `check_cxx_symbol_exists`, or small `check_*_source_compiles` snippets with the exact headers needed by the compiled branch. Windows and MinGW probes must include the correct Win32 headers and include order.

## Version Management

`CMakeLists.txt` is the single source of truth for the project version:

```cmake
project(
  aria2
  VERSION 2.0.2
  ...
)
```

`PROJECT_VERSION` feeds generated package metadata and release artifact naming. Scripts that need the version must read it from `CMakeLists.txt`; they must not carry independent version constants.

Use `./scripts/bump-version.sh <major.minor.patch>` to change the CMake project version. The script only accepts plain numeric versions.

Release tags use `v{PROJECT_VERSION}`. The tag version and CMake project version must match exactly after removing the leading `v`. Pre-release, beta, RC, channel, build-metadata, or date-based release suffixes are not supported.

Treat published release tags as immutable. If a failed release has not been consumed, delete the failed GitHub Release and tag, fix the commit, then recreate the same release deliberately. If a release has been publicly consumed, stop and report the situation. Do not choose, bump, or publish a replacement version unless the maintainer explicitly specifies the new version.

## Dependency Management

`packaging/dependencies.env` is the authoritative dependency baseline for maintained release packaging. It owns dependency versions, source archive names, download URLs, and SHA-256 hashes. Update it before changing dependency versions in scripts, Dockerfiles, workflow files, package notes, or README tables.

Do not add automated dependency PR systems, scheduled dependency update workflows, or Dependabot configuration unless explicitly requested. Dependency updates should be intentional and verified through the affected release path.

## Release Process

The release workflow is `.github/workflows/release.yml`. It runs on `release: published` and on manual `workflow_dispatch` validation. Tag pushes do not publish release builds directly.

Maintained release artifacts are standalone executable assets named `aria2-next-<version>-linux-x86_64`, `aria2-next-<version>-linux-aarch64`, `aria2-next-<version>-macos-arm64`, `aria2-next-<version>-macos-x86_64`, `aria2-next-<version>-windows-x86_64.exe`, `aria2-next-<version>-windows-arm64.exe`, and `aria2-next-<version>-checksums.sha256`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnInsomniacy/aria2-next](https://github.com/AnInsomniacy/aria2-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
