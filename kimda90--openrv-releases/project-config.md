---
trigger: always_on
description: This file guides AI assistants (Cursor, Codex, etc.) working in this repository.
---

# Agent guidance for OpenRV-builds

This file guides AI assistants (Cursor, Codex, etc.) working in this repository.

## Scope

This repo contains **only** build and CI configuration and scripts for building [Open RV](https://github.com/AcademySoftwareFoundation/OpenRV) from **upstream**. We do **not** modify or assume ownership of OpenRV source code; it is cloned at build time from `AcademySoftwareFoundation/OpenRV`.

## Key paths

- **`ci/linux/`** – Linux builds: `Dockerfile.rocky8-extended`, `Dockerfile.ubuntu22.04`, `Dockerfile.rocky9-cy2024` (extended images only; vanilla from upstream), `build_in_container.sh`, `detect_qt.sh`, `patches/`, `KNOWN_ISSUES.md`
- **`ci/windows/`** – Windows builds: `build_windows.ps1`, `package_windows.ps1`
- **`.github/workflows/openrv-release.yml`** – Tag-triggered release workflow

## Build architecture

### Linux (Rocky 9)
- **Two-step image build**: (1) Build the **vanilla** image from upstream `dockerfiles/Dockerfile.Linux-Rocky9-CY2024` (tag: `openrv-rocky9-base`). (2) Build the **extended** image from `ci/linux/Dockerfile.rocky9-cy2024`, which `FROM openrv-rocky9-base` and installs missing deps (mesa-libGL-devel, libglvnd-devel, libdrm-devel, pkg-config) to fix GLEW PFNGL* redefinitions and configure checks. The run step uses the extended image (`openrv-build-rocky9`). Upstream Dockerfile is never edited.
- **Build script**: Our `ci/linux/build_in_container.sh` is mounted into the container and handles cloning, patching, building, and packaging. When `OPENRV_BUILD_CACHE_DIR` is set (CI), the workflow mounts the build cache at that path (e.g. `/home/rv/openrv-build-cache`) and the script symlinks `OpenRV/_build` to it after cloning, so Docker never creates `OpenRV` as root. Without `OPENRV_BUILD_CACHE_DIR`, the script preserves an existing `_build` mount during clone (see Caching).
- **DAV1D on Linux**: We apply a build-time patch to the cloned OpenRV's `cmake/dependencies/dav1d.cmake` so dav1d is fetched via **GIT** instead of URL zip. That gives the dependency a `.git` directory so Meson can generate `include/vcs_version.h` without "fatal: not a git repository". The build script tries `dav1d_use_git.patch` (upstream main: CONFIGURE_COMMAND split across two lines) then `dav1d_use_git_oneline.patch` (older tags: CONFIGURE_COMMAND on one line).
- **GLEW on Linux**: We upgrade GLEW to **2.3.0** at build time by editing the cloned `cmake/dependencies/glew.cmake` (version string, URL hash, and library version). GLEW 2.3.0 fixes the duplicate variable definitions (Issue #449); no GLEW patches are applied.
- **GC (bdwgc) on Linux**: bdwgc may install headers flat (`include/gc.h`) while OpenRV expects `include/gc/gc.h`. After `rvcfg`, the script builds the `dependencies` target, then if `OPENRV_FIX_GC_INCLUDE` is not `0` (default `1`), it creates `install/include/gc/` and copies `gc.h` and `gc_allocator.h` there when needed. Set **`OPENRV_FIX_GC_INCLUDE=0`** to skip this fix (e.g. if your bdwgc already installs the nested layout).
- **Caching**: (1) The OpenRV clone is cached by **tag + commit SHA** (`openrv-rocky9-<tag>-<sha>`). (2) Docker layer cache: Buildx `scope=rocky9` for the vanilla image, `scope=rocky9-ext` for the extended image. (3) Build cache: the host directory `openrv-build-cache-rocky9` is mounted at `/home/rv/openrv-build-cache` and `OPENRV_BUILD_CACHE_DIR` is set so the script symlinks `OpenRV/_build` to it (avoids Docker creating `OpenRV` as root). Cache is restored/saved via `actions/cache` with a **tag + SHA prefix** for incremental resumes.

### Linux (Rocky 8)
- **Two-step image build**: (1) Build the **vanilla** image from upstream `dockerfiles/Dockerfile.Linux-Rocky8` in the cloned repo (tag: `openrv-rocky8-base`). (2) Build the **extended** image from `ci/linux/Dockerfile.rocky8-extended`, which `FROM openrv-rocky8-base` and installs mesa-libGL-devel, libglvnd-devel, libdrm-devel, pkg-config. Upstream Dockerfile is never edited.
- **VFX platform**: Rocky 8 job passes `RV_VFX_PLATFORM=CY2023`; `detect_qt.sh` supports CY2023 and finds Qt 5.15 in `~/Qt/5.15.2/gcc_64`. Artifact runs on Rocky 8 (and typically on Rocky 9) due to older glibc.
- **Caching**: Buildx `scope=rocky8` for vanilla; build cache mounted at `/home/rv/openrv-build-cache` with `OPENRV_BUILD_CACHE_DIR` (same pattern as Rocky 9). Build cache key `openrv-rocky8-build-<tag>-<sha>-...`.

### Linux (Ubuntu 22.04, experimental)
- **Custom Dockerfile**: `ci/linux/Dockerfile.ubuntu22.04` is a translation of the upstream Rocky 9 Dockerfile with equivalent Ubuntu packages.
- **Package mapping**: See the Dockerfile for Rocky 9 → Ubuntu package mappings. Critical packages for GLEW/OpenGL include `libgl-dev`, `libglvnd-dev`, `libdrm-dev`.
- **OpenSSL on Ubuntu**: Upstream `make_openssl.py` installs to `install/lib64` for CY2024 on Linux, while `openssl.cmake` expects `install/lib` when `RHEL_VERBOSE` is unset (Ubuntu has no `/etc/redhat-release`). After building the dependencies target, `build_in_container.sh` copies `RV_DEPS_OPENSSL/install/lib64` into `install/lib` when needed so the linker finds `libcrypto.so.3` / `libssl.so.3`.

### Windows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimda90/OpenRV-Releases](https://github.com/kimda90/OpenRV-Releases) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
