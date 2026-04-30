---
trigger: always_on
description: Replicate the **Windows-only FlexRadio SmartSDR client** (written in C#) as a
---

# AetherSDR — Project Context for Claude

## Project Goal

Replicate the **Windows-only FlexRadio SmartSDR client** (written in C#) as a
**Linux-native C++ application** using Qt6 and C++20. The aim is to mirror the
look, feel, and every function SmartSDR is capable of. The reference radio is a
**FLEX-8600 running firmware 4.1.5**, which speaks **SmartSDR protocol v1.4.0.0**.

## AI Agent Guidelines

When helping with AetherSDR:
- Prefer C++20 / Qt6 idioms (std::ranges, concepts if clean, Qt signals/slots over lambdas when possible)
- Keep classes small and single-responsibility
- Use RAII everywhere (no naked new/delete)
- Comment non-obvious protocol decisions with firmware version
- When suggesting code: show **diff-style** changes or full function/class if small
- Test suggestions locally if possible (assume Arch Linux build env)
- Never suggest Wine/Crossover workarounds — goal is native
- Flag any proposal that would break slice 0 RX flow
- If unsure about protocol behavior → ask for logs/wireshark captures first
- **Use `AppSettings`, never `QSettings`** — see "Settings Persistence" below
- **Read `CONTRIBUTING.md`** for full contributor guidelines, coding conventions,
  and the AI-to-AI debugging protocol (open a GitHub issue for cross-agent coordination)

### Autonomous Agent Boundaries

AI agents (including AetherClaude/pi-claude) may autonomously fix:
- **Bugs with clear root cause** — persistence missing, guard missing, crash fix
- **Protocol compliance** — matching SmartSDR behavior confirmed by pcap/FlexLib
- **Build/CI fixes** — missing dependencies, platform compat

AI agents must **NOT** autonomously change:
- **Visual design** — colors, fonts, layout, theme (user preferences ≠ project direction)
- **UX behavior** — how controls work, what clicks do, keyboard shortcuts
- **Architecture** — adding new threads, changing signal routing, new dependencies
- **Feature scope** — adding features beyond what the issue describes
- **Default values** — changing defaults that affect all users based on one report

When in doubt, the agent should implement the fix and note in the PR that
design decisions need maintainer review. The project maintainer (Jeremy/KK7GWY)
is the sole authority on visual design and UX direction.

## C++ Style Guide

- **No `goto`** — use early returns, break, or restructure the logic
- **No raw `new`/`delete`** — use `std::unique_ptr`, `std::make_unique`, or Qt parent ownership
- **No `#define` macros for constants** — use `constexpr` or `static constexpr`
- **Braces on all control flow** — even single-line `if`/`else`/`for`/`while`
- **`auto` sparingly** — use explicit types unless the type is obvious from context (e.g. `auto* ptr = new Foo` is fine, `auto x = foo()` is not)
- **Naming**: classes `PascalCase`, methods/variables `camelCase`, constants `kPascalCase`, member variables `m_camelCase`
- **Platform guards**: prefer `Q_OS_WIN` / `Q_OS_MAC` / `Q_OS_LINUX` for new code. Existing `_WIN32`/`__APPLE__` guards can be migrated opportunistically — don't do a blanket rewrite.
- **Don't remove code you didn't add** — if rebasing, ensure upstream changes are preserved. Review the diff before submitting.
- **Atomic parameters for cross-thread DSP** — main thread writes via `std::atomic`, audio thread reads. Never hold a mutex in the audio callback for parameter updates.
- **Error handling**: log with `qCWarning(lcCategory)`, don't throw exceptions

## Build

```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build -j$(nproc)
./build/AetherSDR
```

Full dependency list is in `README.md` — don't duplicate it here.

Current version: **0.8.16** (set in both `CMakeLists.txt` and `README.md`).

---

## CI/CD Workflow

CI runs in Docker image `ghcr.io/ten9876/aethersdr-ci:latest` (~3.5 min builds).
**If you add a new `find_package(...)` to CMakeLists.txt, also add the
corresponding `-dev` package to `.github/docker/Dockerfile` and push.** The
`docker-ci-image.yml` workflow rebuilds the image automatically (~3 min); wait
for that before the next CI run can use it.

**`git ship`** alias — squashes local commits ahead of origin/main, creates a
branch, pushes, opens a PR with auto-squash-merge enabled. Commit freely
locally, then ship once.

Branch protection: signed commits required on main, CI must pass, CODEOWNERS
review required, branches auto-delete after merge.

---

## Architecture Overview

Key source directories: `src/core/` (protocol, audio, DSP), `src/models/`
(RadioModel, SliceModel, etc.), `src/gui/` (MainWindow, SpectrumWidget, applets).

**Key classes:**
- `RadioModel` — central state, owns connection + all sub-models
- `AudioEngine` — RX/TX audio, NR2/RN2/NR4/BNR/DFNR DSP pipeline
- `SpectrumWidget` — GPU-accelerated FFT spectrum + waterfall (QRhiWidget)
- `MainWindow` — wires everything together, signal routing hub
- `PanadapterStream` — VITA-49 UDP parsing, routes FFT/waterfall/audio/meters

**Threading:** up to 11 threads — see `docs/architecture-pipelines.md` for the
full thread diagram, data flow, cross-thread signal map, and GPU rendering notes.

**Design principle:** RadioModel owns all sub-models on the main thread.
Worker threads communicate exclusively via auto-queued signals. Never hold

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ten9876/AetherSDR](https://github.com/ten9876/AetherSDR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
