---
trigger: always_on
description: generates a function stub at that address. The manifest already had nearby
---

# AGENTS.md - Project guide for AI agents

## Project

Static recompilation port of **Dante's Inferno** (Xbox 360) to native PC using
the ReXGlue SDK (v0.10.0). ReXGlue translates PowerPC XEX -> C++ ahead of time.

## Key paths

- `dantes_inferno_manifest.toml` - ReXGlue project manifest (SDK-managed; regen with `rexglue init --force`)
- `generated/rexglue.cmake` - SDK build boilerplate (auto-generated, DO NOT EDIT)
- `generated/default/` - codegen output (gitignored, produced during build)
- `src/dantes_inferno_app.h` - **user-owned** app class; override ReXApp hooks here
- `src/main.cpp` - entry point (SDK-managed, preserved on first init only)
- `game/` - extracted Xbox 360 game files (gitignored, copyrighted - never commit)
- `thirdparty/rexglue-sdk/` - SDK clone (gitignored, via setup.ps1)
- `docs/rexglue_notes.md` - ReXGlue workflow & command reference
- `docs/ultrawide_research.md` - Ultrawide support RE findings & implementation plan
- `tools/asset_tool.py` - asset extraction/packing tool (BIG/STR/TG4D/VP6)
- `tools/ASSET_TOOL_README.md` - asset tool documentation
- `tools/Gibbed.Visceral/` - format reference source (gitignored, Zlib license)

## Build commands (Windows)

```powershell
# One-time: build the rexglue CLI from the SDK
cmake --preset win-amd64-release -DREXSDK_DIR=thirdparty\rexglue-sdk
cmake --build out\build\win-amd64-release --target rexglue

# Regenerate SDK-managed files (requires game/default.xex present)
rexglue init --force --project_name dantes_inferno --project_root . --xex_path game\default.xex --game_root game

# Build the port (codegen runs automatically as a build dependency)
cmake --preset win-amd64-release -DREXSDK_DIR=thirdparty\rexglue-sdk
cmake --build out\build\win-amd64-release
```

Run: `out\win-amd64\Release\dantes_inferno.exe`

## Toolchain

- Clang 18+ required (NOT MSVC/GCC). Detected: Clang 22 at `C:\Program Files\LLVM\bin\clang.exe`
- CMake 3.25+, Ninja, Visual Studio 2022 (Windows SDK for D3D12)
- C++23, D3D12 graphics backend on Windows

## Conventions

- `src/dantes_inferno_app.h` is the ONLY place for custom app behavior. Do not
  edit `main.cpp` or `generated/rexglue.cmake` - they are SDK-managed and get
  overwritten by `rexglue init`/`rexglue migrate`.
- For per-instruction custom C++ injection, use `[[mid_asm_hooks]]` in the
  manifest (see docs/rexglue_notes.md).
- Game assets under `game/` are copyrighted and gitignored. Never commit them.
- The SDK under `thirdparty/rexglue-sdk/` is gitignored; re-clone via `setup.ps1`.

## Naming

Project name `dantes_inferno` -> snake_case `dantes_inferno`, PascalCase
`DantesInferno`, UPPER `DANTESINFERNO`. CMake target: `dantes_inferno`.

## Improvement plan

See `docs/improvements_plan.md` for full research findings. Summary:

1. **Graphics quality** (Phase 1, no RE): resolution_scale, anisotropic_override,
   swap_post_effect=fxaa cvars in OnPreSetup. Optionally FidelityFX FSR build.
2. **Input config** (Phase 2, no RE): SDL backend + MnK keybind defaults in
   OnPreSetup. DualShock/DualSense/Xbox all supported via SDL3.
3. **DLC auto-install** (Phase 3, no RE): OnPostSetup hook scans dlc/ folder,
   calls ContentManager::InstallContent() on each STFS package.
4. **Ultrawide** (Phase 4, requires RE): midasm_hook on projection matrix to
   patch aspect ratio. Hor+ anamorphic render strategy.
5. **Button glyphs** (Phase 5, requires RE): replace game's button prompt
   textures based on active input device. Needs SDK patch for device detection
   or glyph_family cvar. Glyph art in metadata/glyphs/.
6. **Installer** (Phase 6): asks user for ISO + DLC folder, extracts to game/
   and dlc/. No STFS logic in installer.

~~Known blocker: ReXGlue issue #75 — Dante's Inferno crashes at startup due to
unimplemented VMX/Altivec PPC instructions (v0.1.1).~~ **Resolved in v0.10.0.**
Game boots and runs. VMX builder bugs causing FMV corruption were found and
fixed (see `docs/vp6_fmv_corruption_fix.md`).

## SDK patches

The SDK under `thirdparty/rexglue-sdk/` has local patches to
`src/codegen/builders/vector.cpp` that fix VMX instruction builder bugs.
Full technical documentation: `docs/vp6_fmv_corruption_fix.md`.

If the SDK is re-cloned, these patches must be re-applied. The fixes are:

1. **`vpkuwus` / `vpkuhus` in-place aliasing** (root cause of FMV corruption):
   Element-by-element packing loops aliased the destination's narrowed array
   with the source's wider array. Replaced with SSE intrinsics.
2. **`vmsum3fp128` dot product mask** (`0x7F` → `0xEF`): A previous "fix"
   incorrectly changed the mask from `0xEF` to `0x7F`, which excluded PPC
   element 0 (X component) from 3-element dot products instead of excluding
   PPC element 3 (W). This broke physics/collision code, causing the
   character to fall through the map after the opening cutscene. Reverted
   to `0xEF` after the `ppc_tests` test suite caught the regression.
3. **Pack builder `unpackhi_epi64` removal**: Pack builders discarded half
   the packed elements via `unpackhi_epi64`. Removed and operand-swapped for
   byte reversal.

## Build & run notes

- The executable loads `rexgpu-xenos.dll` from its own directory, not from
  the SDK output. After rebuilding the SDK, copy:
  `thirdparty\rexglue-sdk\out\win-amd64\rexgpu-xenos.dll` →

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [florinp93/hells-gate-recomp](https://github.com/florinp93/hells-gate-recomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
