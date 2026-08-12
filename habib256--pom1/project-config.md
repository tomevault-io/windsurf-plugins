---
trigger: always_on
description: Architecture / invariants / gotchas for the **emulator side** of POM1. User walkthrough → `README.md`; open work → `TODO.md`; history → `CHANGELOG.md` / `git log`. **Full doc map → [`doc/README.md`](doc/README.md)**.
---

# CLAUDE.md

Architecture / invariants / gotchas for the **emulator side** of POM1. User walkthrough → `README.md`; open work → `TODO.md`; history → `CHANGELOG.md` / `git log`. **Full doc map → [`doc/README.md`](doc/README.md)**.

**Contents:** [Overview](#project-overview) · [Build](#build--run) · [Architecture](#architecture) · [Invariants](#invariants--gotchas) · [Memory map](#memory-map) · [Testing](#testing) · [Version bump](#version-string-locations)

- Apple 1 software (BASIC, SID tunes, microSD shell tools, games) → `sketchs/doc/APPLE1DEV.md` + `sketchs/doc/Programming_Apple1_ASM.md`.
- CLI flags → [`doc/CLI.md`](doc/CLI.md) (impl: `CliDispatcher.cpp`).
- DevBench / cc65 details → [`doc/DEVBENCH.md`](doc/DEVBENCH.md) + [`doc/CC65_WASM.md`](doc/CC65_WASM.md).
- GEN2 HGR card → [`doc/GEN2_RELEASE.md`](doc/GEN2_RELEASE.md).
- 6502 ASM sources for every shipped program → `dev/` (`lib/{apple1,m6502,tms9918,gen2,gen2c,games,…}/`, `cc65/`, `codetank/` = cartridge composition ONLY) and `sketchs/<card>/<name>/` (every program source, single- or multi-file). Compiled artefacts land under `software/<dir>/` — that's what POM1 loads. Official release packages bundle `dev/` (linker cfgs + runtime libs) next to the cc65 toolchain so the in-app DevBench compiles asm/C; a bare source build without cc65 omits both.

## Project Overview

Apple 1 emulator (Dear ImGui, MOS 6502 + display + keyboard + ACI cassette) plus expansion cards: Uncle Bernie's GEN2 HGR, P-LAB A1-SID (6581/8580), TMS9918, microSD (65C22+ATMEGA), **IEC daughterboard** (1541 drive on microSD's spare VIA pins), MODEM BBS (65C51+TCP), Terminal Card, A1-IO & RTC, Juke-Box, CodeTank, Rich Dreher's CFFA1, SWTPC GT-6144 + PR-40 (1976, Jobs' *Interface Age* mod), **Uncle Bernie's Extended ACI** ($C500 page). Linux / macOS / Windows / Web (Emscripten).

## Build & Run

```bash
./setup_pom1.sh             # one-time deps (Linux/macOS)
cd build && cmake .. && make
./run_emulator.sh            # runs from repo root
```

Windows: `setup_pom1.bat` + vcpkg + `cmake --build . --config Release`. `compile_commands.json` symlinked for clangd.

**Graphics backend:** `POM1_RENDERER` cache option picks `opengl` or `metal`. Defaults: **Metal on macOS-non-WASM**, OpenGL everywhere else (Linux, Windows, WASM). Override with `cmake -DPOM1_RENDERER=opengl ..` for the legacy macOS GL path; selecting `metal` outside Apple is a hard configure error. Single seam in `src/PomRenderer.h` (the `pom1::renderer()` singleton fronts both backends — no direct `gl*` calls outside `PomRenderer_GL.cpp` / `PomRenderer_Metal.mm`).

**GL tier — `POM1_GL_ES` (`src/POM1Build.h`):** orthogonal to the backend choice above. It says *"we speak GLES"*, **not** *"we are a browser"* — the distinction the old `__EMSCRIPTEN__` guards conflated. On: always under WASM (WebGL 2.0 **is** GLES 3.0), and natively via `cmake -DPOM1_GLES=ON` (**Raspberry Pi 4/5** — Mesa's V3D caps *desktop* GL at 3.1, so the default GL 3.2 core request fails outright; also most ARM SoCs / old iGPUs). POM1's GL usage (VAO/VBO, shaders, FBO, 2D textures) is the common subset of GL 3.2 core and GLES 3.0, so **one set of sources covers both** — only four things differ, all keyed on `POM1_GL_ES`: the header (`<GLES3/gl3.h>` vs `<GL/gl.h>`+`GLProcs.h`, plus `GLFW_INCLUDE_ES3` in `PomRenderer_GL.cpp`), the `#version 300 es` + precision prologue (`OpenGLShader.cpp`), **direct** entry points instead of the lazy `glfwGetProcAddress` slots (libGLESv2 exports them; same branch as macOS), and the GLFW hints (`GLFW_OPENGL_ES_API` + `GLFW_EGL_CONTEXT_API` — GLX can only hand out a GLES context where `GLX_EXT_create_context_es2_profile` exists, which V3D lacks). Native GLES links `GLESv2`+`EGL` instead of `libGL`; `-DPOM1_GLES=ON` with `POM1_RENDERER=metal` or under Emscripten is a hard configure error. CI compiles the four affected TUs on every push (`ci.yml` → "GLES 3.0 tier"), since the release job that ships it is tag-only. **Two shipped GLES packages, deliberately different**: `release.yml`'s `raspberry` job builds a **generic** aarch64 AppImage (must run Pi 3 → Pi 5), while `pi-borne.yml` (manual dispatch, ported from NeoST) builds for **one core** (`-mcpu=cortex-a72` = Pi 4/400) with two-pass PGO + LTO and emits both an AppImage and a kiosk `tar.gz` — see `packaging/raspberrypi/{build_in_bookworm_pi.sh,README.md}`. Both build inside a `debian:bookworm` container, never on the `ubuntu-24.04-arm` runner: Pi OS *is* bookworm (glibc 2.36) and the runner would stamp `GLIBC_2.39`, which starts on no Pi.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habib256/pom1](https://github.com/habib256/pom1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
