---
trigger: always_on
description: **Updated:** 2026-07-08
---

# AGENTS.md
**Updated:** 2026-07-08

Guide for AI agents working on the MetalSharp repository.

## What This Project Is

MetalSharp is a macOS app that runs Windows Steam games and Windows programs via Wine + Metal translation. It's an Electron app with a Rust HTTP backend, a C++ native D3D/Metal engine, per-game engine routing, runtime bottles, installer profiles, and Linux `.deb` packaging.

## Repository Structure

```
app/
├── src-rust/                    Rust HTTP backend (tiny_http server on port 9274)
│   └── src/
│       ├── main.rs              HTTP router — all /launch, /steam/*, /setup/*, /config, /logs endpoints
│       ├── bottles.rs           Runtime bottles, installer profiles, runtime doctor, redist/source checks
│       ├── launch.rs            Engine detection + game launch — the core routing logic
│       ├── steam.rs             Steam process management, library, install/uninstall, CEF wrapper
│       ├── setup.rs             Per-game preparation (shim builds, DLL staging, FNA runtime)
│       ├── installer.rs         Dependency installer (Wine bundle, Rosetta, Xcode CLI, GPTK, Mono)
│       ├── migrate.rs           Runtime migration + preservation of Steam/prefix/game/bottle state
│       ├── scan.rs              Game library scanner (Steam appmanifest parsing, wine path resolution)
│       ├── sharp_library.rs     Sharp Library imports, installer launch, bottle app imports
│       └── updater.rs           Self-update via GitHub releases DMG download
├── src/
│   ├── main/                    Electron main process
│   └── renderer/                Electron renderer (UI, library, setup wizard)
├── bundles/                     Pre-packaged deps (metalsharp_bundle.tar.zst, SteamSetup.exe, etc.)
├── updater/                     Python update runtime script
├── package.json                 Electron app manifest
└── src-rust/Cargo.toml          Rust backend manifest

src/                             C++ native D3D11/D3D12/DXGI/XAudio2/XInput → Metal implementations
├── d3d/d3d11/                   D3D11 device, context, shaders, resources
├── d3d/d3d12/                   D3D12 device, command queue, command list, resources
├── dxgi/                        DXGI factory, adapter, swap chain
├── metal/                       Metal device, command queue, pipeline, shader translation
├── audio/                       XAudio2 → CoreAudio backend
├── input/                       XInput → GameController backend
├── perf/                        Shader cache, pipeline cache, MetalFX upscaler, GPU profiler
├── runtime/                     PE hooks, compat database, crash diagnostics, DRM detector
├── loader/                      Native PE loader + Win32 shims (kernel32, user32, etc.)
├── wine/                        Wine-specific integration code
├── steam/                       Steam integration
├── win32/                       Win32 API shims (kernel32, user32, registry, etc.)
└── fna/                         FNA/XNA game support (Terraria, Celeste shims)

include/                         C++ public headers
tests/                           C++ test suite (20+ tests: D3D11, D3D12, DXBC, Metal, audio, input)
tools/
├── launcher/                    Native launcher (metalsharp binary + Wine prefix management)
├── dmg/                         DMG packaging scripts
└── linux/                       DEB/Docker/runtime tarball/GHCR package scripts
scripts/
└── install-gptk-runtime.sh      Homebrew GPTK runtime install
configs/                         MTSP rules + Mono DLL maps for FNA games
docs/                            Architecture docs + game compatibility matrix
CMakeLists.txt                   C++ build (native engine + tests)
install.sh                       CLI build script (cmake + test runner)
```

## Key Concepts

### MTSP Routing and Runtime Bottles

Modern runtime paths use MTSP pipeline ids and bottle profiles. Steam games get `steam_<appid>` bottles that are launch-authoritative for runtime checks. Wine Steam remains the live background Steam client; env-dependent Steam routes launch the game executable directly with the bottle prefix, route env, and Steam identity variables instead of trying to make an already-running Steam process inherit new env.

| Public route | Method | Example games |
|--------|--------|--------------|
| `M9` | D3D9 / 32-bit capable DXMT-family route | Nidhogg 2, Undertale, Blasphemous, Dave the Diver |
| `M10` | D3D10 to Metal | D3D10 apps/games |
| `M11` | D3D11 to Metal | Rain World, Schedule I, Subnautica BZ |
| `M12` | D3D12 to Metal through the isolated `dxmt-m12` runtime surface | Peak, Silksong, Elden Ring, D3D12 investigation titles |
| `Mono/FNA` | Windows XNA/FNA through native Mono, staged FNA/XNA assemblies, and host shims | Celeste, Terraria |

Internal route ids such as `dxmt`, `wine_bare`, `m32`, `steam`, `macos_steam`, and `m13` stay backend-parseable for legacy records, diagnostics, and fallback behavior, but they are not normal bottle route buttons.

### Key Paths at Runtime

- Wine runtime: `~/.metalsharp/runtime/wine/`
- Wine prefix: `~/.metalsharp/prefix-steam/`
- DXMT PE DLLs for M9/M10/M11: `~/.metalsharp/runtime/wine/lib/dxmt/x86_64-windows/`
- DXMT M12 PE DLLs: `~/.metalsharp/runtime/wine/lib/dxmt-m12/x86_64-windows/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metalsharp/MetalSharp](https://github.com/metalsharp/MetalSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
