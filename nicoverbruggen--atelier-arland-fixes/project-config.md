---
trigger: always_on
description: This repository releases a 64-bit `d3d11.dll` and a 32-bit settings-launcher `msimg32.dll` for the Steam releases of Atelier Rorona DX, Atelier Totori DX, and Atelier Meruru DX, covering both the English and the multilingual (Japanese/Chinese) executables. Keep the released implementation Arland-specific. Atelier Ayesha support is under investigation but must remain disabled until its atlas-only path is validated. Do not add support code for newer Atelier games; direct those users to upstream `a
---

# AGENTS.md

## Project scope

This repository releases a 64-bit `d3d11.dll` and a 32-bit settings-launcher `msimg32.dll` for the Steam releases of Atelier Rorona DX, Atelier Totori DX, and Atelier Meruru DX, covering both the English and the multilingual (Japanese/Chinese) executables. Keep the released implementation Arland-specific. Atelier Ayesha support is under investigation but must remain disabled until its atlas-only path is validated. Do not add support code for newer Atelier games; direct those users to upstream `atelier-sync-fix` instead.

The current tree contains:

- D3D11 CPU shadow-copy synchronization;
- coherent Map/Unmap handling with deferred shadow uploads;
- successful `.PSSG` path-validation caching for all three Arland games;
- a queue-scoped font-atlas read cache for all three games, extended to the complete menu-construction frame in Rorona, and extended across frames in Meruru while a conversation balloon is live (the conversation text-render cache);
- old-Arland render-target and viewport/scissor correction;
- old-Arland game-side 1440p/4K render-target and raster correction;
- signature-gated launcher mode injection and an optional INI resolution override;
- optional high-resolution shadow-map twins (`ShadowMultiplier`);
- SMAA anti-aliasing applied before UI composition, at a per-title draw boundary (Totori injects at a depth-state change, Rorona and Meruru at the scene render target);
- high-resolution UI text rendered from bundled scalable fonts, on the English builds;
- frame-rate-independent field movement in all three games and travel-map analog cursor movement in Totori and Meruru;
- Rorona battle-shadow restoration, battle-state tracking with a battle-end watchdog, and the optional cut-in shadow/dim handling (all three games, both builds), all in `src/engines/phyre/battle_shadow_restore.cpp`;
- a per-game capability matrix (`src/core/game.cpp`) that centralizes feature availability and defaults;
- crash post-mortem logging and log rotation.

## Repository layout

- `src/core/` — engine-agnostic: the DLL entry points (`main.cpp`), the per-game capability matrix (`game.cpp`), the `arland-fix.ini` layer (`config.cpp`), the MinHook install helpers and per-game `Game` descriptor (`hook_util.{h,cpp}`), the proxy vtable dispatch types (`d3d11_procs.h`), guarded game-memory reads (`mem.h`), companion-file path building shared with the 32-bit DLL (`path_util.h`), the pipeline-state save and restore the post-process passes share (`pipeline_state.h`), the page-patch transaction (`page_patch.h`), the SMAA, supersampling and sharpening passes, crash logging, and the controller and window corrections that name no game. `.def` files export the DLL symbols.
- `src/engines/phyre/` — the PhyreEngine work all three Arland games share, which is everything carrying a per-build address: the D3D11 proxy layer (`sync_fix.cpp`), with the cut-in shadow feature carved into `battle_shadows.cpp` behind `sync_internal.h`; the executable-specific menu hooks (`menu_fix.cpp`), with the battle-shadow-restore subsystem carved into `battle_shadow_restore.cpp` behind `menu_internal.h`; high-resolution UI text (`font_hires.cpp`); and the field, save-menu, item, shop, stream, world-map and startup fixes.

  Singular on purpose, and there is no dispatch layer: Rorona, Totori and Meruru are all PhyreEngine, so unlike the Dusk project there is only ever one module. The directory exists because the two repositories carry about forty files in common, and identical paths are what make a drift between them visible in a diff rather than only in a review.
- `src/launcher/` — both launcher pieces, neither of which shares code with the game DLL: `launcher_gui.cpp` is the 64-bit `arland-fix-launcher.exe` settings window, and `launcher_proxy.cpp` the 32-bit `msimg32.dll` that redirects Koei Tecmo's own front-end to it.
- `vendor/minhook/` contains the unmodified vendored MinHook dependency and its license; `vendor/stb/` holds stb_truetype; `vendor/font/` holds the bundled replacement fonts (`.ttf`) and the per-glyph fallback face.
- `scripts/embed_font.py` compiles the vendored fonts into the DLL at build time; the generated sources live under the build directory and are not committed.
- `.github/workflows/build.yml` builds and publishes both Windows DLLs.
- `README.md` is the user-facing overview: what the mod does per game, and how to install it. It is the only prose document in the repository.
- The settings launcher is the option surface. No ini ships: the mod writes `arland-fix.ini` itself, and the launcher writes it again on every Save. Environment switches are diagnostics and must not be given a key.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicoverbruggen/atelier-arland-fixes](https://github.com/nicoverbruggen/atelier-arland-fixes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
