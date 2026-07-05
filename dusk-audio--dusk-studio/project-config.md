---
trigger: always_on
description: Dusk Studio is a portastudio-style DAW for Linux, JUCE 8 / C++17. The authoritative spec is [DuskStudio.md](DuskStudio.md). Read it before changing anything non-trivial.
---

# Dusk Studio — instructions for Claude

Dusk Studio is a portastudio-style DAW for Linux, JUCE 8 / C++17. The authoritative spec is [DuskStudio.md](DuskStudio.md). Read it before changing anything non-trivial.

## Architecture cheat-sheet

- **Audio backend**: PipeWire (primary) via JUCE's JACK backend; ALSA fallback.
- **DSP**: extracted from the user's existing Dusk Audio plugins at `/home/marc/projects/plugins/`. Shared headers live (or will live) at `plugins/plugins/shared/dsp-cores/` so both Dusk Studio and the Dusk plugins are single-source-of-truth consumers. Resolved via `-DDUSK_PLUGINS_PATH=/path/to/plugins` or sibling `../plugins` (mirror of the JUCE pattern). Header-only cores: edit a file in the plugins repo, next Dusk Studio build picks it up — no copy step, no submodule bump.
- **JUCE**: 8.x, resolved via `-DJUCE_PATH` or sibling `../JUCE` (same scheme as the Dusk plugins repo).
- **Native plugin hosts** (Linux): CLAP (`src/engine/clap/`), LV2 (`src/engine/lv2/`, lilv/suil via pkg-config), VST3 (`src/engine/vst3/`, Steinberg SDK hosting subset via the `external/vst3sdk` submodule — Dusk-owned mirror `dusk-audio/vst3sdk`, tag `dusk-vst3sdk-v1`, GPL-3.0 arm). All implement `src/engine/hosting/INativeInstance`; compile gates `DUSKSTUDIO_HAS_NATIVE_{CLAP,LV2,VST3}` with `#else` stubs so other platforms build. GPL invariant: never bundle any third-party plugin in a distribution (see LICENSES.txt).
- **Topology**: 24 channel strips (HPF → 4-band EQ → FET/Opto comp → sends → pan → bus assign → fader → mute/solo) → 4 aux buses (EQ + comp + fader) → master (Pultec EQ + bus comp + tape sat + fader). Three banks of 8 select which 8 strips the control surface drives at a time; the full 24 are visible on screen.

## Build

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
./build/DuskStudio_artefacts/Release/DuskStudio
```

JUCE and the Dusk plugins repo are auto-discovered from sibling directories. Pass `-DJUCE_PATH=...` or `-DDUSK_PLUGINS_PATH=...` to override either.

### Cross-OS dev (macOS authoring, Linux testing)

Single canonical build dir on both OSes: `build/` (app) and `build-tests/` (Catch2 tests). The sibling repos and JUCE source differ per OS, but the build directory does not — switching machines does not require a new build dir.

| OS    | App build | Tests build      | JUCE source                              | Plugins source                 |
|-------|-----------|------------------|------------------------------------------|--------------------------------|
| macOS | `build/`  | `build-tests/`   | `../JUCE` (upstream)                     | `../plugins` (any branch)      |
| Linux | `build/`  | `build-tests/`   | `../JUCE-wayland` (plugdata-team fork)   | `../plugins` (single checkout, main) |

CMake auto-detects:
- **JUCE** — on Linux it prefers `../JUCE-wayland` if present, falls back to `../JUCE`. The wayland fork has 5 local commits Dusk Studio depends on (XEmbed mapping, X11-on-Wayland fix, peer-creation latch — see [memory](../../.claude/projects/-home-marc-projects-Dusk Studio/memory/linux_juce_wayland_pin.md)) and a divergent `addDefaultFormatsToManager` free function.
- **Plugins** — the donor is a single checkout at `../plugins`, kept on `main` as its resting state (the stable donor API). Auto-detected from the sibling `../plugins` directory; override with `-DDUSK_PLUGINS_PATH=/path/to/plugins`. If `../plugins` is ever on a feature branch when you build, you build against *that* branch — check it out to `main` first if you need the released donor API. (The former `../plugins-main` worktree was removed; the repo is consolidated to one directory. Do NOT run `git worktree add ../plugins-main main` — `main` is already checked out at `../plugins` and git will refuse it.)

The upstream-vs-fork `addDefaultFormats` API split is hidden behind [src/engine/JuceCompat.h](src/engine/JuceCompat.h) — call `duskstudio::juce_compat::addDefaultFormats(fm)` and the `#if defined(__linux__)` lives in one place. Don't sprinkle new platform `#ifdef`s into call sites.

## Phase plan

Phases 1a → 5 follow [DuskStudio.md](DuskStudio.md). Don't skip ahead. As of writing, Phase 1a (live mixer) and most of Phase 2 (multitrack recording + atomic JSON session save/load + autosave) are working. Phase 1b (send-bus plugin hosting on aux strips) and Phase 3 prep (take history) are the most recent additions. Phase 3 proper (markers, fader automation, punch+loop refinements) is next.

## Audio thread rules (MANDATORY)

The audio thread (`AudioEngine::audioDeviceIOCallbackWithContext` and any `process*` it calls) is **real-time**. Violating these rules causes glitches, clicks, dropouts, or crashes:

- **NEVER allocate memory** — no `new`, `make_unique`, `push_back`, `resize`, `std::string`, or `juce::String`. Pre-size all buffers in `prepare`/`audioDeviceAboutToStart`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dusk-audio/dusk-studio](https://github.com/dusk-audio/dusk-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
