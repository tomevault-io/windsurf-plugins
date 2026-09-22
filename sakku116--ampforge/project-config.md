---
trigger: always_on
description: Authoritative project context and workflow for every coding agent. Read this before touching files.
---

# Amp Forge — Agent Instructions

Authoritative project context and workflow for every coding agent. Read this before touching files.

## Conventions

- Write all codebase and git artifacts in English: code, comments, UI text, documentation, commits, branches, PRs, and reviews. Chat may use the user's language.
- Treat this file as the project-context source of truth. `CLAUDE.md` only points here.

## Agent skills

### Issue tracker

GitHub Issues via `gh`. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default five labels. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: root `CONTEXT.md` and `docs/adr/`. See `docs/agents/domain.md`.

## Handoffs

Use the `handoff` skill when creating or resuming a handoff, and offer one after significant work. Store every generated handoff in `.handoffs/` as `YYYYMMDD_NN_title.md`: `NN` is the zero-padded daily sequence, starting at `01`; `title` is a concise kebab-case summary. On resume, inspect `.handoffs/` rather than a root `HANDOFF.md`.

## Project

Amp Forge is a Windows-only, C++20/JUCE 8.0.2 real-time guitar VST3/VST2 host. It has two executables:

| Target | Entry point | Responsibility |
|---|---|---|
| `AmpForge` | `src/main.cpp` | GUI, audio, and host |
| `AmpForgeScanWorker` | `src/scan_worker_main.cpp` | Isolated per-plugin scanner; emits XML to stdout so a crashing plugin cannot crash the host |

```powershell
cmake -S . -B build -G "Visual Studio 18 2026" -A x64
cmake --build build --config Debug --parallel
# or: make build / make release
```

The host and copied scan worker are in `build/AmpForge_artefacts/Debug/`. Place optional SDKs at `asio/` (`JUCE_ASIO=1`) and `vst2sdk/` (`JUCE_PLUGINHOST_VST=1`).

Runtime data lives in `%APPDATA%\AmpForge\`: `host.log`, `presets/*.tfpreset`, settings, and `pluginCache.xml`. The scanner persists `KnownPluginList` plus file modification times: startup uses incremental `scanAll(false)` and Rescan uses full `scanAll(true)`.

## Source Map

| Area | Files | Responsibility |
|---|---|---|
| Root UI | `MainComponent.*` | Owns UI, callbacks, timer, persistence, MIDI learn, and chain refresh |
| Audio | `AudioEngine.*` | `AudioDeviceManager`, real-time I/O, atomic master gain/volume/mute, CPU metrics, MIDI routing |
| Hosting | `PluginHost.*`, `PluginChain.*` | Formats, instances, editors, chain snapshots, crossfade, sections, bypass, stable slot identity |
| Scanning | `PluginScanner.*`, `ScanSubprocess.*`, `PluginScanGuard.*` | Plugin discovery, cache, worker subprocess, Windows SEH load guard |
| Persistence | `Preset.*`, `TemplateManager.*`, `ControlMap.*` | Presets, named chain templates, trigger/action and expression mappings; see `docs/control-mapping-lifecycle.md` for map ownership |
| Keyboard capture | `KeyboardControlController.*`, `KeyboardCaptureAdapter.*` | Session-global keyboard policy, Win32 hook translation, exclusive ownership, and message-thread action delivery |
| Chain UI | `ChainListBox.*` | Vertical rows and horizontal columns, section/slot interactions, levels, volume controls |
| Theme and logs | `ToneForgeLookAndFeel.*`, `HostDebug.h` | `tf::colour` stage palette and `[AmpForge]` logger |

## Chain Rules

`PluginChain` uses immutable `SlotList` snapshots published atomically:

- The message thread performs structural edits under its edit lock, builds a replacement list, then publishes it.
- The audio thread loads `activeList` once per block; it never blocks or deletes. Per-slot bypass and gains are atomics.
- A crossfade holds the incoming `fadeInList`, blends it, then promotes it to `activeList`.
- `activeList.load()` is the chain playing on the audio thread. Use it only for audio work plus `publishWithCrossfade` and `prepare`.
- `displayList()` is `fadeInList` when present, otherwise `activeList`; it is the chain the UI shows. `currentList()` delegates to it, so every message-thread edit targets the visible chain during a template-switch fade. `publish()` cancels a pending fade for an immediate edit.

### Stable identity and sections

- `slotId` is stable across reordering and persistence. Assign with `nextSlotId++`; restore a positive persisted ID and advance `nextSlotId`; assign a new ID for `0`.
- `ControlAction::index` holds a template index only for `loadTemplate`. For `toggleBypass` and `activatePresetSlot`, it holds `slotId`; resolve it with `findSlotIndexById()` at execution time.
- Rebuild in `sectionDefs` order. Section identity is stable; positional bindings are invalid after a reorder.
- Stomp slots bypass independently. Activating a preset slot atomically bypasses every other slot in its section.
- Section bypass is persisted and propagated atomically to its slots. Section output gain applies at the last slot; slot post-gain applies after its plugin.
- Refresh the visible chain with `refreshChainList()` after any chain or control-map change that affects rows, hints, or state.

## UI and Capabilities

- The library scans VST3/VST2 safely, supports persistent custom paths, and adds selected plugins to the chain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sakku116/ampforge](https://github.com/sakku116/ampforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
