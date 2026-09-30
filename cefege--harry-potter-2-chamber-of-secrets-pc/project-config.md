---
trigger: always_on
description: Rules of engagement for autonomous agents working in this repository. The
---

# AGENTS.md — Operating Handbook for AI Agents

Rules of engagement for autonomous agents working in this repository. The
canonical operational reference is `Docs/OPERATIONS.md`; this handbook defines
behavior and boundaries, not procedure detail.

This is a single-era C++ codebase (legacy UE1-derived engine, modernized).
There is no Rust code in this tree — an earlier effort to rewrite the engine
in Rust (`crates/`, `cargo`) was removed prior to public release. A handful of
scripts and doc sections still reference that removed code (see "Known Stale
References" below); treat those as dead, not as a second live subsystem.

## Ownership / Subsystem Map

|Subsystem|Path|Notes|
|---|---|---|
|Core|`HarryPotter2/Unreal/Core/`|Object model, serialization (`FName`/`FString`, package79 archives), memory, names, portable SHA-256|
|Engine|`HarryPotter2/Unreal/Engine/`|Game loop, `UEngine::InputEvent` dispatch, level flow|
|Render|`HarryPotter2/Unreal/Render/`|Scene rendering shared by driver backends|
|XOpenGLDrv|`ThirdParty/XOpenGLDrv/`|GL driver + text seam (`FCanvasTextRequest`, native text backends: CoreText on macOS, FreeType on Linux). Pinned upstream; see ThirdParty rules below|
|SDLDrv|`HarryPotter2/Unreal/SDLDrv/`|SDL2 window/input (`USDLViewport` → `CauseInputEvent`)|
|SDLLaunch|`HarryPotter2/Unreal/SDLLaunch/`|Launch policy/store, desktop shell integration (`HP2ShellLauncher`), `HP2MacLauncher.mm`|
|Launcher|`HarryPotter2/Unreal/Launch/`|Bootstrap paths, static packages, editor runtime entry, platform path resolution (`HP2Paths.cpp`)|
|ALAudio / codecs|`HarryPotter2/Unreal/ALAudio/`, `EAAudioCodec/`, `Vorbis/`, `OpenAL/`|Sound pipeline|
|Launcher shell UI|`Launcher/Quickshell/`|Quickshell-based desktop launcher scaffold|
|Build scripts|`Build/*.py`|`game_test.py`, `smoke_maps.py`, `check_bundle.py`, `prepare_retail_data.py`, `repair_save.py`, `abi_inventory.py`, `retail_flow_smoke.py`; CMake modules in `Build/CMake/`|
|Tests|`Tests/`|C++ `*Tests.cpp` (`main()`-style, minimal TU), Python `*Tests.py` (`unittest`, no app launch), fixtures in `Tests/Fixtures/`. Registered via `hp2_add_behavior_test` in `Build/CMake/HP2Targets.cmake`|
|Data roots|`HarryPotter2/Unreal/` (prototype), retail overlay via `Build/prepare_retail_data.py`|Prototype root is `HP2_UNREAL_ROOT` in `Build/CMake/HP2Sources.cmake`|
|Dist bundle (macOS)|`dist/macos-arm64/HarryPotter2.app`|Verify with `Build/check_bundle.py`|
|Dist bundle (Linux)|`dist/linux-arm64/` (`bin/`, `lib/`, `share/`)|Installed by the `linux-arm64*` presets via `Build/CMake/HP2Install.cmake`|
| Release DMG (macOS)|`dist/macos-arm64/HarryPotter2-<version>-macos-arm64.dmg`|Built by the `hp2_macos_dmg` target (macOS branch of `HP2Install.cmake`); the artifact Homebrew and GitHub Releases serve|
| Homebrew cask|`Casks/harry-potter-2.rb`|This repo is its own tap. Tap name must equal the repo name, and the URL must be passed explicitly, or Homebrew looks for a separate `homebrew-hp2` repo. `version`/`sha256` are owned by `.github/workflows/release.yml` via `Build/stamp_cask.py` — never hand-edit|
| In-bundle importer|`Build/hp2-import-game-data.sh`|Wraps `prepare_retail_data.py`; shipped as `Contents/Resources/Import Game Data.command` and documented in `Docs/PLAYING.md`. Requires `python3` on the user machine|

## Platforms

Two supported platforms, both arm64, built from the same CMake tree:

- **macOS 15+** — presets `macos-arm64`, `macos-arm64-asan-ubsan`,
  `macos-arm64-tsan`, `macos-arm64-full-smoke`, `macos-arm64-vulkan`,
  `macos-arm64-retail`.
- **Linux (arm64)** — presets `linux-arm64`, `linux-arm64-asan-ubsan`,
  `linux-arm64-tsan`, `linux-arm64-full-smoke`, `linux-arm64-retail`.

`Docs/OPERATIONS.md` predates the Linux presets and still says "macOS 15+ on
arm64 only" in places — the presets in `CMakePresets.json` are the source of
truth for what actually exists, not that sentence.

## Known Stale References

Left over from the removed Rust rewrite; do not treat as live, do not extend:

- `Docs/OPERATIONS.md` "Cargo era (hp2rs)" section — describes a `crates/`
  workspace that no longer exists in this tree.
- `Build/package_rust_app.py`, `Build/check_bundle_rs.py`,
  `Tests/RustBundleTests.py` — reference `dist/macos-arm64-rs/` and a
  `target/release/hp2rs` binary that are never produced by this repo's build.
- Anything mentioning `cargo`, `crates/`, or `hp2rs` outside this section.

If a task touches these paths, flag the drift to the user rather than quietly
"fixing" cross-cutting doc/script debt as a side effect of an unrelated
change.

## Non-Negotiables

1. **Never claim done from compile alone.** A green build proves nothing about
   behavior. Run the named ctest targets that cover the change and report the
   observed result.
2. **Artifacts go to `HP2_ARTIFACT_DIR`.** Test env provides isolated
   `HOME`/`TMPDIR`, `LC_ALL=C`, `TZ=UTC`, `HP2_TEST_NAME`, and a pre-created
   `HP2_ARTIFACT_DIR`. Structured reports, logs, captures land there — never
   in ad-hoc stdout or the repo root.
3. **Data-profile honesty.** State which profile a claim holds for:
   `data-none` (no game data), `data-prototype` (`HarryPotter2/Unreal`),
   `data-retail` (overlay from `prepare_retail_data.py`). A result under one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cefege/harry-potter-2-chamber-of-secrets-pc](https://github.com/cefege/harry-potter-2-chamber-of-secrets-pc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
