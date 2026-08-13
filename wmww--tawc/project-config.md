---
trigger: always_on
description: Tess's Android Wayland Compositor (tawc) is an Android app plus rootfs/build scripts for running desktop Linux programs on Android.
---

Tess's Android Wayland Compositor (tawc) is an Android app plus rootfs/build scripts for running desktop Linux programs on Android.

## Quick Reference
- Build APK: `scripts/build-app.sh`
- Build/install/launch: `scripts/app-build-install.sh` (`--no-build`, `--no-launch` supported)
- Compositor Rust check: use the Android build path (`scripts/build-app.sh` or Gradle app tasks). Do **not** run host `cargo check --manifest-path compositor/Cargo.toml`; `ndk-sys` only compiles for Android.
- Run in rootfs: `scripts/rootfs-run.sh '<command>'` or interactive with no command
- Run Firefox: `scripts/rootfs-run.sh 'firefox --no-remote'`
- Run lxterminal: `scripts/rootfs-run.sh 'lxterminal'`
- Integration tests: `scripts/run-integration-tests.sh [filter]` (builds/installs test deps as needed; `--no-build` reuses existing deploy)
- tawcroot tests: `tawcroot/test.sh [--host|--device] [--no-build] [FILTER...]`
- App unit tests: `./gradlew :app:testDebugUnitTest`
- Logs: `adb logcat -s tawc-native` for Rust, `adb logcat -s tawc` for Kotlin
- Screenshot: use `/data/local/tmp/tawc-dev/`; analyze screenshots with a sub-agent, then delete device and host copies.

## Current Project Shape
- Install methods: `tawcroot` is default and the only release-supported method. `proot` and `chroot` are debug-only dev-loop options.
- Supported distros are **Arch Linux ARM** and **Debian sid** (Arch x86_64 stands in for ALARM on the emulator). Manjaro ARM and Void still ship in every build but are dev-only, behind the install form's "Other distros" expander — see [notes/distro-options.md](notes/distro-options.md).
- Graphics backends: `libhybris`, `libhybris-zink`, `gfxstream`, and `cpu` ship by default. `libhybris` works on all tested physical devices and is the production/default path. `gfxstream` is experimental/partial; it is the x86_64 emulator default only because libhybris is unsupported there. See [notes/gpu-strategy.md](notes/gpu-strategy.md), [notes/libhybris-zink.md](notes/libhybris-zink.md), and [notes/gfxstream-bridge.md](notes/gfxstream-bridge.md).
- The debug exec broker is the normal host-to-app command path. Host helper binary: `tests/integration/src/bin/tawc-exec.rs`; wrapper: `scripts/tawc-exec.sh`; protocol notes: [notes/exec-broker.md](notes/exec-broker.md).
- SHM buffers are intentionally tinted magenta by default to expose fallback paths. Do not remove this unless explicitly asked.

## Operating Rules
- Keep docs compact here. Put durable design/build details in `notes/`; start with [notes/README.md](notes/README.md).
- This is an agent-written project. Existing code and notes may be wrong; verify against source/scripts before trusting old prose.
- When `.tawctarget` names a usable device target, verify implemented app/device-facing changes on that target before calling them done.
- If you add or change a build dependency, host package, vendored repo, env var, or toolchain version, update [notes/building.md](notes/building.md) in the same change.
- Use existing scripts instead of one-off adb/chroot commands when possible, if the scripts are broken fix them (or at least open an issue).
- Only commit, amend, tag, or push when explicitly asked. Git push may hang without user approval.
- Do not run formatting tools (`cargo fmt`, `rustfmt`, etc.) unless explicitly asked.
- Do not edit `app/icon.svg` unless explicitly asked — it is the hand-drawn source of truth for the app icon. Every other form of the mark is generated from it by `scripts/gen-icon.sh`; after an asked-for icon change, run that script and commit the SVG plus all four generated files together. See [notes/building.md](notes/building.md) ("App icon").
- Keep prose, comments, errors, and commit messages short unless extra detail is genuinely useful.
- Keep production logging sparse. Do not log per-frame work, per-input events, test-only milestones, or high-volume protocol chatter; prefer explicit query/debug surfaces for tests.

## Releases
- Full process: [notes/release.md](notes/release.md). Versions are a plain counter (`1`, `2`, …); `versionName` in `app/build.gradle.kts` is the single source, `versionCode` derives from it.
- When asked to prep a release: bump the version, draft release notes, write the F-Droid changelog and run `scripts/check-version-sync.sh` (nothing else in the repo should name the version statically), commit `release: vN`, tag `vN` (no push), then hand off — signing runs as the key-owning user where Claude is unavailable, so end by telling the user to run `scripts/build-release-apk.sh` and the remaining publish steps from the notes.

## Issues
- Issues live in `issues/`. Do not solve them unless asked or the fix falls out of current work.
- Create/update issues for nontrivial problems discovered during other work.
- Delete confirmed-solved issues; move still-useful context into notes first.

## Plans
- Future plans live in `plans/`. Do not execute them unless asked.
- Do not write new plans unless asked.
- Like issues, delete them and integrate their contents into your notes when they are complete.

## Device Safety
- The physical phone is precious: do not reboot it unless explicitly asked.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wmww/tawc](https://github.com/wmww/tawc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
