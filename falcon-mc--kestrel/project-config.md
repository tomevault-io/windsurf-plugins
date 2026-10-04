---
trigger: always_on
description: Instructions for AI coding agents working in the Kestrel repository.
---

# AGENTS.md

Instructions for AI coding agents working in the Kestrel repository.

If you are a human, read [CONTRIBUTING.md](CONTRIBUTING.md) instead. It is the authoritative document, and
where the two disagree, CONTRIBUTING.md wins.

---

## The project

Kestrel is a native Minecraft: Bedrock Edition client written from scratch in C++. It is not derived from the
official client: rendering, UI, networking and the protocol are implemented here and in this organisation's
other repositories.

- **Language:** C++20 (Objective-C++ for the macOS platform layer).
- **Build system:** CMake with Ninja, dependencies pulled with `FetchContent`.
- **Platforms:** Windows (Direct3D 12, MSYS2 UCRT64 gcc), Linux (Vulkan, SDL3) and macOS (Metal, clang), all
  built by CI.
- **License:** LGPL-3.0.
- **Branch:** `main`.
- **Versions:** `1.0.1+1.26.50`, Kestrel's own version, then the Bedrock version it speaks.

---

## Required reading

| File                               | Read it for                                                        |
|------------------------------------|--------------------------------------------------------------------|
| [CONTRIBUTING.md](CONTRIBUTING.md) | **Mandatory.** Contribution rules, AI disclosure, closing reasons. |
| [SECURITY.md](SECURITY.md)         | What counts as a vulnerability and how to report one privately.    |
| [README.md](README.md)             | Project overview, command line, the mod API.                       |

---

## Commands

| Task                          | Command                                                       |
|-------------------------------|---------------------------------------------------------------|
| Configure and build           | `./build.sh`                                                  |
| Configure and build (Windows) | `build.bat`                                                   |
| Incremental rebuild           | `cmake --build build`                                         |
| Regression tests              | configure with `-DKESTREL_BUILD_TESTS=ON`, then `ctest --test-dir build` |
| Example mods                  | configure with `-DKESTREL_BUILD_EXAMPLE_MODS=ON`              |

The first configure downloads dependencies and needs network access. `build.sh` and `build.bat` write their
output to `build.txt`. A change that does not compile is worse than no change.

`particle-effects` reads the vanilla resource pack of an installed game and fails without one. CI skips it.

---

## Layout

```
include/            Headers, mirroring src/
  mod/              The public mod API. Mods compile against it.
src/
  client/           The client loop, session, account and Realms services, player motion
  world/            Chunks, meshing, block and entity assets, particles, resource packs
  render/           rhi/ interface plus the d3d12/, metal/ and vulkan/ backends
  ui/, menu/        JSON UI, fonts, the HUD, menus, forms, the inventory
  audio/            Sound loading and playback (miniaudio)
  modding/          Mod loading and the host side of the mod API
  agent/            The JSON control port behind --agent
  platform/         win32/, macos/ and linux/ windows, input, clipboard and paths
shaders/            GLSL for the Vulkan backend, compiled to SPIR-V during the build
data/               Small tables and the app art, embedded into the binary
tests/              Regression tests
examples/mods/      Example mods
```

The network transport, protocol packets, NBT, block state upgrades and Bedrock data files live in separate
`Falcon-MC` repositories. Never edit their copies under `build/_deps`.

---

## Hard rules

1. **Never invent APIs.** Before calling a function, field, packet or JSON UI property, search the codebase
   and confirm it exists with that exact signature. If you cannot find it, say so.
2. **Never guess vanilla behaviour.** Packet formats, UI layout values, physics constants and texture paths
   must match the game. If you cannot verify a value, say which one.
3. **Never commit game assets.** Textures, sounds, UI JSON, fonts and models are read at runtime from the
   player's installation. Nothing taken from Minecraft goes into the repository, not even for tests.
4. **Keep every backend building.** A change to `render/rhi` must land in Direct3D 12, Metal and Vulkan, and a
   change to `platform/` must cover all three platforms or say why not.
5. **Keep mods compiling.** Changes to `include/mod` must stay source compatible unless the task says otherwise.
6. **One logical change per branch.** Report unrelated problems you notice instead of fixing them.
7. **No repository-wide reformatting** and no refactors outside the task.
8. **No new dependencies** without an issue first.
9. **No dead code** and no debug logging left in the diff.
10. **Do not weaken security checks** on anything a server sends, or on the agent port's token. If one is in
    your way, stop and explain.
11. **Do not touch** `.github/workflows/` or release automation unless that is the task.
12. **Do not commit, push or open a pull request** unless the human explicitly asks.

---

## Architecture

- **Platform code goes behind interfaces.** Windows, input, clipboard and file dialogs go through

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Falcon-MC/Kestrel](https://github.com/Falcon-MC/Kestrel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
