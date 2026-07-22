---
trigger: always_on
description: This repository builds prebuilt LLVM/Clang, Halide, cctools, MSVC, and Xcode support bundles for release artifacts. Most meaningful behavior lives in GitHub Actions workflows, CMake toolchain/cache files, and version-specific patches applied to external upstream checkouts.
---

# AGENTS.md

## Project Scope

This repository builds prebuilt LLVM/Clang, Halide, cctools, MSVC, and Xcode support bundles for release artifacts. Most meaningful behavior lives in GitHub Actions workflows, CMake toolchain/cache files, and version-specific patches applied to external upstream checkouts.

Treat this as a packaging and cross-compilation repository. Keep changes small, reproducible in CI, and aligned with the existing matrix of:

- Architectures: `x86_64`, `aarch64`
- Platforms: `windows`, `macos`, `ubuntu-22.04`, `ubuntu-24.04`
- LLVM versions currently built by CI: `20.1.8`, `22.1.4`
- Halide versions currently built by CI: `19.0.0`, `21.0.0`

## Repository Layout

- `.github/workflows/llvm-prebuilt.yml` builds and packages LLVM/Clang distributions.
- `.github/workflows/halide-prebuilt.yml` builds Halide packages using LLVM artifacts from a previous LLVM workflow run.
- `.github/workflows/cctools-prebuilt.yml` builds Linux cctools artifacts.
- `.github/workflows/msvc-bundle.yml` and `.github/workflows/xcode-bundle.yml` package platform SDK/toolchain support files.
- `.github/workflows/github-release.yml` downloads workflow artifacts, creates checksums, and publishes a GitHub release.
- `cmake/*.cmake` contains target toolchain files plus `llvm-distribution.cmake`, the LLVM initial cache that defines the installable distribution surface.
- `patches/*.patch` contains version-specific patches applied to upstream LLVM or Halide sources after checkout.
- `scripts/Get-UbuntuPackage.ps1` resolves, downloads, and extracts Ubuntu packages for Linux aarch64 cross-compilation.

## Build Model

LLVM builds are two-stage:

1. Build native host tools from the upstream LLVM checkout with Ninja.
2. Configure the target LLVM build with the matching `cmake/<arch>-<os>.cmake` toolchain file and `-C cmake/llvm-distribution.cmake`.

Important variables exported by CI and consumed by `llvm-distribution.cmake` include:

- `LLVM_NATIVE_TOOL_DIR`
- `LLVM_TABLEGEN`
- `CLANG_TABLEGEN`
- `LLVM_CONFIG_PATH`
- `LLVM_VERSION`
- `CMAKE_INSTALL_PREFIX`

Do not remove the host-tools phase when changing cross-compilation behavior. Cross builds depend on native tablegen/config/helper tools.

## Local Verification Commands

Prefer PowerShell-compatible commands because the workflows use `pwsh` heavily across platforms.

Patch application smoke test:

```powershell
# LLVM 20 checkout
git -C llvm-project apply ../llvm-prebuilt/patches/llvm-20-add-lld-install-targets.patch
git -C llvm-project apply ../llvm-prebuilt/patches/llvm-20-add-llvm-name-prefix-to-llc-lli-opt-tools.patch

# LLVM 22 checkout
git -C llvm-project apply ../llvm-prebuilt/patches/llvm-22-add-llvm-name-prefix-to-llc-lli-opt-tools.patch
```

LLVM host tools:

```powershell
cmake -G Ninja -S llvm-project/llvm -B llvm-host `
  -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" `
  -DCMAKE_BUILD_TYPE=Release -Wno-dev
cmake --build llvm-host --target llvm-tblgen clang-tblgen llvm-config
```

LLVM target configuration:

```powershell
cmake -G Ninja -S llvm-project/llvm -B llvm-build `
  -DCMAKE_INSTALL_PREFIX=llvm-install `
  -DCMAKE_TOOLCHAIN_FILE="$PWD/llvm-prebuilt/cmake/x86_64-ubuntu-24.04.cmake" `
  -C "$PWD/llvm-prebuilt/cmake/llvm-distribution.cmake" -Wno-dev
```

LLVM package build/install targets:

```powershell
cmake --build llvm-build
cmake --build llvm-build --target install-distribution
```

When validating a broad change, test Ubuntu x86_64 first because it is the simplest build path, then test aarch64 cross-compilation and both Ubuntu releases separately.

## Toolchain File Rules

- Keep platform leaf files small. Ubuntu files should include the matching base Linux file.
- Base Linux files set `CMAKE_SYSTEM_NAME`, `CMAKE_SYSTEM_PROCESSOR`, cross-compilation state, compilers, target architecture, and target triples.
- Windows toolchains set `CMAKE_SYSTEM_NAME` and processor only; CI supplies MSVC target environments with `ilammy/msvc-dev-cmd`.
- macOS toolchains set `CMAKE_SYSTEM_NAME Darwin` and `CMAKE_OSX_ARCHITECTURES`; CI sets `MACOSX_DEPLOYMENT_TARGET`.
- Preserve the `aarch64`/`arm64` distinction: artifact and matrix names use `aarch64`; Apple CMake architecture values use `arm64`; Windows MSVC target architecture values use `arm64`.

## LLVM Distribution Rules

`cmake/llvm-distribution.cmake` is the distribution contract. Be careful when changing:

- `LLVM_ENABLE_PROJECTS`
- `LLVM_TARGETS_TO_BUILD`
- `LLVM_DISTRIBUTION_COMPONENTS`
- `LLVM_DEVELOPMENT_COMPONENTS`
- tool utility/component lists

The lld development components depend on the `llvm-*-add-lld-install-targets.patch` patch. If lld install components fail, verify the matching lld patch still applies and still creates the expected install targets before changing the component list.

The `llc`, `lli`, and `opt` tools are patched to install as `llvm-llc`, `llvm-lli`, and `llvm-opt` to avoid system tool conflicts. Account for that naming in tests, package inspections, and docs.

## Patch Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awakecoding/llvm-prebuilt](https://github.com/awakecoding/llvm-prebuilt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
