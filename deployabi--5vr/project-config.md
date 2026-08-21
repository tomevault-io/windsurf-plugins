---
trigger: always_on
description: 5VR is a single-player-only VR injection mod for RAGE-engine games
---

# Repository Guidelines

## Mission & Non-negotiables
5VR is a single-player-only VR injection mod for RAGE-engine games
(GTA V Legacy first). Hard rules, enforced in code and review:
- **Story Mode only.** Online sessions (GTA Online) and BattlEye presence
  hard-disable the mod via `OVRInject/Game/OnlineGuard` (ADR-0003). There is
  no config bypass; never add one; never weaken it "for testing".
- **No DRM/anti-tamper circumvention** and no committed game assets or
  third-party mod material. Prior-art/reference material from other mods is
  never committed (see `docs/LEGAL.md`).
- **Version-pin everything:** AOB patterns, offsets, and shader hashes live in
  `manifests/*.ini`, never hardcoded in source (ADR-0004).
- Label `UNVERIFIED` anything not confirmed on real hardware in a real headset.
- Design decisions live in `docs/01-architecture.md` + `docs/ADR/`; update them
  when behavior/conventions change.

## Project Structure & Module Organization
- `GTAVOVR.sln` hosts the Visual Studio solution (projects: GTAVOVR, OVRInject,
  OVRInjectShim; slice + tests projects are being added).
- `GTAVOVR/` is the dev launcher (env setup + CreateRemoteThread injection;
  target process via argv[1] or `GTAVR_TARGET_PROCESS`). `--check` runs
  preflight only (disabled marker, build detection vs `manifests/`, VR
  runtime, BattlEye posture); exit codes 0 ok-to-try, 2 unsupported build,
  3 runtime missing, 4 injection failure, 5 mod disabled by `gtavr.disabled`.
- `OVRInject/` holds the core VR injection logic, D3D hooks, OpenXR/OpenVR
  backends, overlay UI, and shaders. Subdirs: `D3DHook/` (Present/ResizeBuffers
  hooks, dummy-device late-injection, HudRedirect), `VR/` (backend
  abstraction), `OpenXR/`, `Game/` (title plugin: camera/FOV/state, OnlineGuard,
  BuildManifest — all pattern resolution on a bounded background worker; the
  render thread is O(1)), `Stereo/` (StereoEngine + EyeDelivery AER state
  machine, ComfortRuntime), `Perf/` (PerfStats frametime ring + CSV export),
  `Overlay/`, `Vive/` (HMDRenderer only).
- `OVRInjectShim/` is the shipped loader: a proxy `dxgi.dll` that loads
  `OVRInject.dll` (ADR-0005).
- `tools/` holds `install.bat` / `uninstall.bat` (idempotent game-dir
  installer/reverser via `runtime_package.ps1`; install/verify refuse game
  builds the pinned ScriptHookV cannot load — `OfficialGameBuild` in
  `tools/launcher/ScriptHookPackage.cs` — or builds with no manifest
  section; smoke-test against throwaway dirs only),
  `bootstrap_thirdparty.ps1` (hash-verifies/fetches the vendored libs a
  clean clone needs),
  `GTAVR-Play.bat` (one-click game launcher), `GTAVR-Panel.bat` +
  `gtavr_panel.py` (control panel: start game, backend switch, inject, live
  log tail), `collect_logs.bat/.ps1` (support bundle to Desktop),
  `scan_patterns.py` + `verify_camera_chain.py` (new-build pattern
  resolution against the running game).
- `samples/D3D11Cube/` is the Phase 2 vertical-slice target + golden-image
  harness (`check_golden.py`).
- `tests/` holds the unit-test runner (38 suites incl. TestEyeDelivery;
  402 tests / 75111 checks green on 2026-08-11) and the
  injection-lifecycle harness (`injection_lifecycle.py`).
- `manifests/` holds per-title, per-build signature/offset/hash INIs.
- `docs/` holds feasibility, architecture, ADRs, legal, known-issues,
  test-matrix, perf-report, hud-postfx, and `docs/user/` guides.
- `ThirdParty/` is vendored dependencies (OpenVR, OpenXR, ImGui, minhook,
  DirectXMath) — avoid edits unless required.
- Root `.ini` files (`gtavr_camera.ini`, `gtavr_settings.ini`) are templates
  for runtime config; built artifacts go under `x64/` and `build_*/` (ignored).

## Build, Test, and Development Commands
- Visual Studio: open `GTAVOVR.sln`, build `Release|x64` (toolset v145).
- CLI build (Git Bash — use dash switches; `/p:` gets path-mangled):
  `env -u GTAV_INSTALL_DIR "/c/Program Files/Microsoft Visual Studio/18/Community/MSBuild/Current/Bin/MSBuild.exe" GTAVOVR.sln -p:Configuration=Release -p:Platform=x64 -m -v:m`
  (`env -u GTAV_INSTALL_DIR` is required: the post-build copy into the game
  directory fails the build if that env var points at a non-writable path.)
- Unit tests: build `tests/GTAVRTests.vcxproj` and run the exe
  (`tests/run_tests.bat`).
- Slice golden test: `GTAVR_SLICE_GOLDEN=40 D3D11Cube.exe` twice, then
  `python samples/check_golden.py` (determinism + stereo disparity).
- Prereqs: OpenVR headers in `ThirdParty/openvr/headers`, OpenXR SDK per
  `ThirdParty/openxr/README.md`, ImGui per `ThirdParty/imgui/SETUP.md`
  (`HAS_IMGUI`).
- Clean-clone bootstrap: the build links vendored binaries
  (`ThirdParty/openvr/lib/x64/openvr_api.lib`,
  `ThirdParty/openxr/lib/x64/openxr_loader.lib`, plus the runtime
  `openvr_api.dll` / `openxr_loader.dll` payloads), which are committed so a
  clean clone has them. The non-redistributable ScriptHookV SDK import
  library (`GTAVRBridge/ScriptHookV.lib`) is deliberately NOT committed —
  building the GTAVRBridge project needs a manual fetch. If anything is
  missing or fails verification, run
  `powershell -ExecutionPolicy Bypass -File tools/bootstrap_thirdparty.ps1`
  — it hash-verifies (SHA-256, pinned) what is present, re-fetches OpenVR

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeployAbi/5VR](https://github.com/DeployAbi/5VR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
