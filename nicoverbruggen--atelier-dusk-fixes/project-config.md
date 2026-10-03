---
trigger: always_on
description: Fixes for the Steam Dusk-trilogy DX ports: Atelier Ayesha DX, Atelier Escha & Logy DX, Atelier Shallie DX. This project is still pre-release. The detailed investigation record and the work queue are kept privately and are deliberately not published here.
---

# AGENTS.md

## Project scope

Fixes for the Steam Dusk-trilogy DX ports: Atelier Ayesha DX, Atelier Escha & Logy DX, Atelier Shallie DX. This project is still pre-release. The detailed investigation record and the work queue are kept privately and are deliberately not published here.

## Source layout

Split by engine, because the trilogy spans two and they share nothing a fix can reach:

- `src/core/` — engine-agnostic: the D3D11 proxy (`main.cpp`), engine dispatch (`engine.cpp`), the capability matrix (`game.cpp`), the `dusk-fix.ini` layer (`config.cpp`), D3D11 vtable ownership and hook installation (`d3d11_hooks.cpp`), the high-resolution fix and its render-target census (`highres.cpp`), the verdict on which surface is the scene (`scene_pass.cpp`), the per-engine answers the pre-UI pass needs (`scene_policy.h`), the antialiasing features (`smaa.cpp`, `supersample.cpp`, `sharpen.cpp`), logging.

  **`d3d11_hooks.cpp` is the only place that hooks a D3D11 vtable.** Features own their detours and their policy and declare them in a "wiring for d3d11_hooks.cpp" section of their own header; they do not call MinHook. Two modules hooking one vtable is how an enable/disable race gets written by accident, and how a half-installed set survives a failure that should have rolled everything back. Add a slot by adding a row to a spec table there, not by hooking from a feature.

  Rendering features may still need one engine-specific decision each. Keep it out of core: `src/core/scene_pass.h` takes a `SceneTargetTest` callback for "which bind is the scene", and each engine supplies its own — `src/engines/phyre/scene_target.cpp` for Ayesha, `src/engines/ktgl/scene_target.cpp` for Escha & Logy and Shallie. Core declines and says so in the log when no engine has registered one, which is now a fallback rather than the state any shipped game is in. Both SMAA's pre-UI pass and supersampling depend on that one answer.
- `src/engines/phyre/` — Ayesha (PhyreEngine, old MSVC CRT). Arland ports live here: the atlas cache, field physics, and the scene-target rule.
- `src/engines/ktgl/` — Escha & Logy and Shallie (KTGL, UCRT). Fingerprinting and the engine-specific address fixes, including loading text, startup flow, save integrity and world-map movement. Each detour or patch is gated by the recognized executable row and its own expected-byte window.

  Plural on purpose. `src/core/engine.{h,cpp}` is the **dispatch layer** — it resolves which engine this process is and forwards to one module. `src/engines/` holds the modules it forwards to. Singular for the dispatcher, plural for the implementations.
- `src/launcher/` — both launcher pieces, neither of which is an engine module and neither of which shares code with the game DLL: `launcher_gui.cpp` is the 64-bit `dusk-fix-launcher.exe` settings window, and `launcher_proxy.cpp` is the 32-bit `msimg32.dll` the games' own front-ends load. They agree on ini key names and nothing else.

One `d3d11.dll` covers all three games. Do not split it per game or per engine: address-based and Direct3D fixes are gated on both the capability matrix and an exact executable fingerprint. The only exception is a small set of window-API hooks that must be installed before D3D11 initialization; those may change a call only when narrow runtime facts identify the game window, and must forward everything else untouched. `msimg32.dll` is a separate target only because the front-ends that load it are 32-bit processes.

Neither engine module may include the other's headers, and no address pack belongs in `src/core`. **Nor may `src/core` name an engine module.** Only `engine.cpp` may, and only to dispatch: it resolves the running executable and returns that engine's `SsaaPolicy` and `ScenePolicy`. Core asking one engine a question about the other is how Ayesha's pre-UI pass came to be gated on a KTGL module reporting itself idle, and how a decline meant for Ayesha was logged in KTGL's words.

User-facing options go in `dusk-fix.ini` through the capability matrix's `Descriptor`; environment switches are diagnostics and must not be given an ini key. `[Diagnostics] VerboseLogging` is the one diagnostic with a key, because it only decides how much the mod writes about itself and costs a user nothing to turn on when a bug report asks for it. A diagnostic that makes the game slower keeps its environment switch.

**No ini ships. The mod writes its own.** `configPath()` creates `dusk-fix.ini` when it is absent and seeds `[Launcher] SkipLauncher`; every other key is seeded lazily by the read that wants it, with the capability matrix's default for the game actually running. The settings launcher then writes the file whenever the user saves. A shipped file could not do better: it would carry one set of values for three games, including keys a given game ignores.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicoverbruggen/atelier-dusk-fixes](https://github.com/nicoverbruggen/atelier-dusk-fixes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
