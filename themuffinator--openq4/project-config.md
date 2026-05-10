---
trigger: always_on
description: This file describes project goals, rules, and upstream credits for anyone working on OpenQ4.
---

# OpenQ4 Agent Guide

This file describes project goals, rules, and upstream credits for anyone working on OpenQ4.

**Project Metadata**
- Name: OpenQ4
- Author: themuffinator
- Company: DarkMatter Productions
- Version: 0.1.010
- Website: `www.darkmatter-quake.com`
- Repository: `https://github.com/themuffinator/OpenQ4`
- Companion GameLibs Repo (local): `E:\Repositories\OpenQ4-GameLibs`

**Goals**
- Deliver a complete, open-source code replacement for Quake 4 (engine + game code).
- Preserve behavior required by the shipped Quake 4 assets (base PK4s) where practical.
- Maintain full single-player and multiplayer parity with in-tree game code.
- Modernize the engine and game code while keeping stock-asset compatibility as a guiding constraint.
- Package both SP/MP under one unified game directory (`baseoq4/`) with `game_sp` + `game_mp`.
- Establish a cross-platform foundation targeting modern systems (Windows, Linux, macOS; x64 first) through SDL3 and Meson.
- Keep Quake4SDK-derived game-library source ownership in `OpenQ4-GameLibs`, with OpenQ4 consuming those sources directly at build time.
- Keep BSE source integrated in-tree under `src/bse/` and treat it as first-party OpenQ4 code.

**Rules**
- Do not target compatibility with the proprietary Quake 4 game DLLs; OpenQ4 ships its own game modules and keeps full freedom to evolve the project.
- Treat `E:\Repositories\OpenQ4-GameLibs` as part of the same development workspace for planning, edits, and validation.
- For SDK/game-library work, make canonical source edits in `OpenQ4-GameLibs` first; OpenQ4 loads game sources from that companion repo and should not mirror them under `src/game`.
- Treat `src/bse/` as the canonical BSE source location.
- Build BSE into the client executable; do not reintroduce an external `OpenQ4-BSE_<arch>` runtime module without an explicit project decision.
- Dedicated server builds keep the disabled BSE manager path unless a change proves they genuinely need the full effect runtime.
- Keep `baseoq4/` as the single unified game directory; do not split SP/MP into separate mod folders.
- Keep repo-authored runtime overrides under `content/baseoq4/`; treat `.install/baseoq4/` as staged output rather than an editing target.
- Prefer changes that match Quake 4 SDK expectations and shipped content behavior.
- Document significant changes in the documentation and keep `README.md` accurate.
- Treat release changelog maintenance as part of feature completion. User-facing curated release notes belong in `docs-dev/releases/vX.Y.Z.md`, and the manual release workflow will publish that file when it exists.
- Write release notes for end users first: lead with the visible benefit, call out any action or compatibility note the reader needs, and avoid dumping internal implementation trivia unless it materially helps the audience.
- Use `builddir/` as the standard Meson build output directory for local builds, VS Code tasks, and launch configurations.
- Treat `.install/` as the release-style package root; stage built binaries into `.install/` and `.install/baseoq4/`.
- Keep game-module outputs available under both `builddir/baseoq4/` (direct run) and `.install/baseoq4/` (staged package).
- Keep `.install/` focused on runtime/staged content: engine executables in `.install/`, game DLLs and staged overrides/assets in `.install/baseoq4/`.
- Do not rely on `.install/` as a linker artifact store; keep compiler/linker intermediates and development-only outputs in `builddir/`.
- MSVC import libraries (`*.lib`) are not runtime requirements for OpenQ4 execution; prefer keeping them in `builddir/` (or other developer artifact output), not in release-style `.install/` packages.
- Use `meson install -C builddir --no-rebuild --skip-subprojects` (via `tools/build/meson_setup.ps1`) when staging `.install/` to avoid third-party subproject installs outside the package tree.
- `tools/build/meson_setup.ps1` can trigger SDK/game-library builds in `../OpenQ4-GameLibs` during `compile` when `OPENQ4_BUILD_GAMELIBS=1`; OpenQ4 no longer syncs a local `src/game` mirror.
- On Windows, do not invoke raw `meson ...` from an arbitrary shell; use `tools/build/meson_setup.ps1 ...` (or run `tools/build/openq4_devcmd.cmd` first) so `cl.exe`/MSVC tools are always available.
- Prefer platform abstractions through SDL3 and avoid introducing new platform-specific dependencies in shared engine code when an SDL3 path exists.
- Keep Meson as the primary build entry point and keep dependency management through Meson subprojects.
- Treat x64 as the baseline architecture for active support while staging additional modern architectures incrementally.
- Keep credits accurate and add new attributions when incorporating upstream work.
- Localize all user-facing UI text: never introduce hardcoded display strings in GUIs/UI code when a `#str_*` lookup is possible; add/update language-table entries whenever new text is needed.
- Avoid adding engine-side content files (e.g., custom material scripts) unless absolutely required for compatibility; the goal is to run with the original game assets and only OpenQ4 binaries (engine + game modules, plus minimal external libs).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themuffinator/OpenQ4](https://github.com/themuffinator/OpenQ4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
