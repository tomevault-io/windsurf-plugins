---
trigger: always_on
description: This repository is a GUI-first, multi-service gRPC platform for Android development workflows.
---

# APK Workbench - Agent Overview

## Purpose
This repository is a GUI-first, multi-service gRPC platform for Android development workflows.
It is intentionally minimal but complete enough to extend. The GTK4 UI and CLI are thin clients;
the service crates contain the real workflows. The project is designed around a JobService that
streams events to clients while long-running jobs execute in other services.
- Canonical upstream repository: `https://github.com/Denuo-Web/APK-Workbench`

## Supported host
- Linux ARM64 (aarch64) is the only supported host for running the full stack (services/UI/Cuttlefish).
- Debian 13 on Linux ARM64 is the primary validated distro for full-stack support, release smoke
  tests, and default Cuttlefish host-tool automation; Raspberry Pi OS 64-bit is included.
- Non-Debian Linux ARM64 hosts are experimental for the full stack and generally require explicit
  overrides such as APKW_CUTTLEFISH_INSTALL_CMD.
- x86_64 is intentionally out of scope because Android Studio already covers it.
- Toolchain catalog includes Linux ARM64 SDK/NDK artifacts plus Windows ARM64 NDK artifacts (r29/r28c/r27d);
  no darwin SDK/NDK artifacts are published in the custom catalogs.
- Cuttlefish install uses APKW_CUTTLEFISH_INSTALL_CMD when set; Debian-like hosts fall back to the
  android-cuttlefish apt repo install command.
- GitHub Releases is the canonical binary distribution channel (`linux-aarch64.tar.gz` plus checksums);
  the Debian `.deb` is an additional convenience artifact, and GitHub Packages is not used for native binaries.
- Release packaging scripts default `VERSION` from workspace metadata, share a single binary list via
  `scripts/release/common.sh`, share Java runtime policy via `scripts/release/apkw-env.sh`, and enforce Linux ARM64 host checks unless
  `APKW_ALLOW_UNSUPPORTED_RELEASE_HOST=1` is set for explicit experimental packaging.
- Debian packages install the launcher and binaries under `/usr/lib/apkw`, expose `/usr/bin/{apkw,apkw-ui,apkw-cli}` symlinks,
  ship minimal manpages for those commands, depend on `libgtk-4-1` plus `libwebkitgtk-6.0-4` for the embedded Cuttlefish pane,
  validate `PKGNAME` before packaging, and strip staged binaries during packaging.

## Maintenance
Keep this file and the per-service AGENTS.md files in sync with code changes. When Codex changes
files, commits, or pushes, update the relevant AGENTS.md entries and adjust TODO lists to remove
completed items or move them into the implementation notes.

## Repository map
- crates/apkw-core: JobService (event streaming and job registry)
- crates/apkw-workflow: WorkflowService (multi-step pipeline orchestration)
- crates/apkw-toolchain: ToolchainService (SDK/NDK provider, install, verify)
- crates/apkw-project: ProjectService (templates, create/open, recent list)
- crates/apkw-build: BuildService (Gradle builds and artifact listing)
- crates/apkw-targets: TargetService (ADB + Cuttlefish target management)
- crates/apkw-observe: ObserveService (run history and bundle export)
- crates/apkw-ui: GTK4 GUI client
- crates/apkw-cli: CLI sanity tool
- crates/apkw-util: Shared helpers (paths, time, service bootstrap, job history)
- crates/apkw-telemetry: Opt-in telemetry spooler (usage events + crash reports)
- crates/apkw-proto: Rust gRPC codegen for proto/apkw/v1
- proto/apkw/v1/*.proto: gRPC contracts
- CHANGELOG.md: release notes and post-tag history used for version increment prep
- scripts/dev/run-all.sh: local dev runner for all services (uses the shared launcher env helper to auto-export ANDROID_SDK_ROOT/ANDROID_HOME and APKW_ADB_PATH when an SDK is detected)
- scripts/dev/apkw-gradle.sh: wrapper for building external Android projects with APKW-managed ARM64 SDK/NDK + `aapt2` override; prefer this over plain `./gradlew` when working outside this repo, and note that it prefers APKW-managed toolchains over inherited shell SDK env unless `APKW_GRADLE_RESPECT_EXISTING_ENV=1`
- scripts/release/common.sh: shared release metadata/helpers (workspace version, supported-host guards, binary list)
- scripts/release/apkw-env.sh: shared Android/Java environment detection for the dev runner and installed launcher; it exports host OS + 4K/16K page-size profile and allows explicit override via `APKW_HOST_PAGE_SIZE`
- scripts/release/build.sh: release build + GitHub Releases tarball packaging helper
- scripts/release/build-deb.sh: Debian (.deb) convenience package builder (templates package metadata/docs and installs the launcher helper)
- scripts/release/apkw-start.sh: installed launcher (services + UI, logs to ~/.local/share/apkw/logs)
- packaging/deb/*: Debian packaging metadata (control, desktop entry, postinst/postrm, manpages)
- assets/apkw.svg: GTK app icon used by the Debian package
- docs/release.md: release build steps
- SampleConsole: Minimal Compose sample app (Sample Console) bundled with APKW
- CS492_Assignment1_RosenauJ/CS492A1RosenauJ: Course assignment sample app

## Runtime topology
Default addresses (override with env vars):
- Job/Core:     127.0.0.1:50051 (APKW_JOB_ADDR)
- Toolchain:    127.0.0.1:50052 (APKW_TOOLCHAIN_ADDR)
- Project:      127.0.0.1:50053 (APKW_PROJECT_ADDR)
- Build:        127.0.0.1:50054 (APKW_BUILD_ADDR)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Denuo-Web/APK-Workbench](https://github.com/Denuo-Web/APK-Workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
