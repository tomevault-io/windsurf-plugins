---
trigger: always_on
description: **Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.
---

# Glossa — AI Agent Context

---

## Coding guidelines

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

## What Is Glossa

Glossa is a **headless speech-to-text daemon** for **Ubuntu + GNOME + Wayland**. It captures microphone audio on a global hotkey, transcribes it via a cloud API (Groq / OpenAI), places the text into the Wayland clipboard with `wl-copy`, and pastes it into the active window with `dotool`.

**Target platform (only):** Ubuntu, GNOME, Wayland, user session.
No support for KDE, Sway, X11, macOS, or Windows.

---

## Project Layout

```
glossa/
├── Cargo.toml               # workspace root
├── rust-toolchain.toml       # stable toolchain, clippy + rustfmt
├── AGENTS.md                 # ← this file
├── build-release-tarball.sh  # packages release tarball + checksums + updater asset
├── install.sh                # interactive installer for local user setup
├── uninstall.sh              # uninstall helper for local user setup
├── update.sh                 # bootstrap updater that fetches the latest release updater
├── contrib/
│   ├── assets/sounds/        # start.wav, stop.wav cue sounds
│   ├── assets/tray/          # tray icons (light/dark themes)
│   ├── dotool/               # bundled dotool binary + vendored source
│   ├── examples/config.toml  # reference TOML config
│   ├── release/              # release-time updater template
│   └── systemd/              # user services for glossa + dotool
└── crates/
    ├── glossa-core/          # pure types, config, state, enums
    ├── glossa-app/           # state machine, orchestration, port traits
    ├── glossa-audio/         # CPAL capture, WAV I/O, silence trim, cue playback
    ├── glossa-platform-linux/# portal, tray, wl-copy, dotool, IPC, doctor
    ├── glossa-stt/           # Groq/OpenAI/compatible HTTP clients
    └── glossa-bin/           # CLI (clap), bootstrap, composition root
```

### Crate Dependency Graph

```
glossa-bin
  ├── glossa-app
  │     └── glossa-core
  ├── glossa-audio        (implements glossa-app port traits)
  │     └── glossa-core
  ├── glossa-stt          (implements glossa-app::SttClient)
  │     ├── glossa-core
  │     └── glossa-app
  ├── glossa-platform-linux (implements remaining port traits)
  │     ├── glossa-core
  │     └── glossa-app
  └── glossa-core
```

---

## Crate Responsibilities

### `glossa-core` — Domain Types

Pure data crate. **No async, no platform deps, no I/O.**

| Module | Contents |
|--------|----------|
| `config/` | `AppConfig` with 7 sections: `InputConfig`, `ControlConfig`, `ProviderConfig`, `AudioConfig`, `PasteConfig`, `UiConfig`, `LoggingConfig`. Validated on load. |
| `command.rs` | `AppCommand` enum (`StartRecording`, `StopRecording`, `ToggleRecording`, `Restart`, `Shutdown`) + `CommandOrigin` |
| `state.rs` | `AppState` enum (`Idle`, `Recording`, `Processing`, `Pasting`, `ShuttingDown`) with payload structs |
| `ids.rs` | `SessionId(Uuid)` — unique per recording cycle |
| `audio.rs` | `RecordSpec`, `AudioFormat`, `CapturedAudio` |
| `paste.rs` | `PasteMode` (`CtrlV`, `CtrlShiftV`, `ShiftInsert`) |
| `provider.rs` | `ProviderKind` (`Groq`, `OpenAi`, `OpenAiCompatible`), `ProviderConfig` |
| `status.rs` | `AppStatus` — lightweight snapshot for `glossa status` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Glaicer/Glossa](https://github.com/Glaicer/Glossa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
