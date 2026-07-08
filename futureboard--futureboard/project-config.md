---
trigger: always_on
description: Before implementing any task spec, read:
---

# CLAUDE.md — Futureboard Studio Agent Rules

Before implementing any task spec, read:

1. `tasks/SKILL.md`
2. `DESIGN.md` if the task touches UI, layout, styling, windows, dialogs, panels, or component structure
3. The smallest relevant task file/section only

This file is a quick operating contract for Claude Code inside the Futureboard Studio repository.  
`tasks/SKILL.md` is the deeper source of truth.

---

## Prime Directive

Work in the smallest safe scope.

Do not rewrite the repository.  
Do not implement an entire roadmap unless explicitly requested.  
Do not "improve" unrelated code while fixing a specific bug.

Every patch must be:

- scoped
- buildable
- reversible
- validated
- honest about what was and was not tested

Before editing:

```txt
1. Restate the exact requested scope.
2. Inspect the relevant files.
3. Identify the current behavior.
4. Identify the smallest safe patch.
5. List likely files to change.
6. Implement only that patch.
7. Run the smallest relevant validation.
8. Report changed files, validation, and remaining TODOs.
```

Never claim validation passed if it was not run.

---

## Project Map

Futureboard surfaces:

- **Futureboard Express** — WebUI
- **Futureboard Lite** — Electron
- **Futureboard Studio** — Native Rust / GPUI
- **SphereDirectAudioEngine / DAUx** — native realtime audio engine
- **SpherePluginHost** — plugin scanning, loading, processing, and editor hosting
- **SphereWebAudioCore** — WASM/WebAudio fallback engine
- **SphereUIComponents** — shared native UI components

Common paths, but always inspect the repo because paths may differ:

```txt
apps/
  web/
  electron/
  native/
  experimental/native/

crates/
  SphereUIComponents/
  SphereDirectAudioEngine/
  SphereWebAudioCore/
  SpherePluginHost/

external/
  vst3sdk/
  clap/
  ARA_SDK/
  zed/

tasks/
  native/
  audio/
  plugin/
```

For WebUI WASM DSP work, inspect:

```txt
crates/SphereWebAudioCore
```

---

# PluginHost

If editing or creating PluginHostWrapper / plugin host integration, inspect:

```txt
crates/SpherePluginHost
external/vst3sdk
external/clap
```

Planned/target support includes:

- VST3
- CLAP
- AU
- LV2
- Linux/macOS platform paths

Native Studio should use the pure host core without requiring N-API.

Do not force JUCE into the project unless explicitly requested.

---

# Audio / Plugin Bridge Rules — Do Not Break Realtime

Futureboard audio work must be realtime-aware.

## Realtime hot paths must not contain

- heap allocation in steady-state processing
- `println!`, `eprintln!`, tracing/logging directly from audio callback
- filesystem I/O
- plugin scanning
- JSON parsing
- `serde_json::Value` lookup
- `HashMap<String, ...>` lookup per block
- blocking locks
- sleeps
- waits on UI thread
- Node/Electron calls
- unbounded queues
- panics across FFI

Use instead:

- preallocated buffers
- immutable runtime snapshots
- compact enums/indices resolved before playback
- atomics
- bounded lock-free/SPSC queues
- diagnostics rings drained by non-realtime threads

## Classify touched code before editing

When touching audio/plugin code, classify every changed function as one of:

```txt
Realtime callback / hot path
Audio control thread
Plugin host producer thread
UI/control path
Scanner/offline path
Test-only path
```

If it is realtime or producer-hot, apply realtime rules.

## Bridge producer rules

The plugin host producer must not rely on `sleep(250µs)` polling as the main wake mechanism.

Preferred architecture:

```txt
Engine/audio callback publishes request_seq
Engine signals named event / SetEvent
Host producer WaitForSingleObject
Host producer processes exact target instance
Host publishes response_seq
Engine freshness guard verifies response
```

Windows bridge threads should use appropriate scheduling hardening where relevant:

- `timeBeginPeriod(1)` while bridge is active
- MMCSS `"Pro Audio"` for producer thread
- suitable thread priority
- cleanup on shutdown

Do not remove freshness guards to hide dropouts.  
If the guard says stale, fix producer timing or bridge sequencing.

## Instance routing rules

Never broadcast MIDI or parameter events to all loaded plugin voices unless the feature explicitly says "broadcast".

Route by:

- `instance_id`
- region identity
- insert id
- track/insert mapping

Required behavior:

```txt
MIDI for insert A reaches only insert A.
Param event for insert A reaches only insert A.
Plugin state for insert A restores only insert A.
```

## Plugin state is P0

DAW project save/load is not usable unless plugin state is persisted.

VST3 state work must support:

- component state
- controller state when available
- opaque binary blobs
- project snapshot persistence
- restore after plugin instantiation and before playback/editor open
- clear error reporting if restore fails

Do not silently discard plugin state.

## VST3 ProcessContext must be real

Do not hardcode:

```txt
tempo = 120
time signature = 4/4
playing = true
projectTimeSamples = 0
```

ProcessContext should come from actual engine transport and timeline state:

- sample rate
- block frames
- playing/stopped
- recording if available
- project time samples
- tempo map
- time signature map
- PPQ/bar position when available
- loop/cycle state when available


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [futureboard/Futureboard](https://github.com/futureboard/Futureboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
