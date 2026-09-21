---
trigger: always_on
description: macOS desktop app (Tauri v2, Rust backend + webview) combining dictation and
---

# AGENTS.md — Voice Tool

macOS desktop app (Tauri v2, Rust backend + webview) combining dictation and
read-aloud TTS, with on-device LLM refinement (Fn+Ctrl) over your dictation.
Personal daily-driver tool; also a Rust-business content artifact.

**Design notes live in `docs/voice-tool-architecture.md` (build order + macOS
gotchas). When this file and the architecture doc disagree, ask.**

---

## Scope — do not violate
- **macOS only.** Do not add Windows/Linux abstractions, cfg gates, or "portable"
  layers. Pick the simplest macOS-native path every time.
- **Single-user tool.** No accounts, sync, multi-user, or telemetry. (Public
  distribution as a signed/notarized DMG is now in scope — see
  `docs/releasing.md` — but the app stays single-user and phone-home-free.)
- **Free & open source under MIT** (`LICENSE`). Keep it MIT-clean: only pull in
  permissively-licensed deps — no GPL/AGPL code statically linked into the
  binary. Ask before adding anything copyleft.
- **Fully on-device.** Everything runs locally — no cloud providers, API keys, or
  network calls except model downloads. Do not re-add cloud backends (Groq,
  OpenRouter, ElevenLabs) without asking.
- Lead with the Rust engine; the webview is just overlay + settings UI.

## Stack — use these, don't substitute without asking
- App: Tauri v2 · hotkeys: `tauri-plugin-global-shortcut`
- Audio in: `cpal` · audio out: AVFoundation via `objc2`
  (`AVSpeechSynthesizer` for native TTS, `AVQueuePlayer` for Kokoro)
- STT: local `whisper-rs` (whisper.cpp, Metal). Model name in `stt_model`.
- Inject/selection: `enigo` + `arboard`
- TTS: native `AVSpeechSynthesizer` via `objc2` — **default**; local neural
  **Kokoro** (`kokoro-en`: ONNX via `ort`, CoreML) optional. Selected by
  `tts_provider` in config. Both on-device. **`kokoro-en` runs with
  `default-features = false`** — its default `g2p-espeak` backend statically
  links GPL-3.0 espeak-ng, incompatible with our MIT license; Kokoro uses its
  cmudict G2P instead. Do not re-enable it (see `THIRD-PARTY-NOTICES.md`).
- Refinement (Fn+Ctrl): local Qwen3 1.7B via `llama-cpp-2` (embedded
  llama.cpp, Metal).
- `reqwest` is used only to download models from HuggingFace.
- async: `tokio` · native FFI: `objc2*`

## Hard rules — these prevent silent, hard-to-debug failures
1. **Register global hotkeys on the main thread** or they silently fail on macOS.
2. **Inject via clipboard paste + simulated `Cmd+V`**, never per-key synthetic
   typing. Restore the clipboard by **watching the change-count, not a fixed
   timer**. Do not preserve binary clipboard contents (avoids double-paste from
   clipboard managers).
3. **Selection capture = simulate `Cmd+C` → read clipboard → restore.** Do not rely
   on the AX selected-text API; it's inconsistent across apps.
4. **Fn hold-to-dictate is a `CGEventTap` (`fn_key.rs`) gated on Accessibility
   alone** — never call `IOHIDRequestAccess`: it records an Input Monitoring
   denial that overrides the Accessibility coupling and silently wedges the tap
   off. Default chords: dictation `Cmd+Shift+D`, read-aloud `Cmd+Shift+R`, cycle
   speed `Cmd+Ctrl+S` (all hold-to-talk except speed). Two chord traps, both
   confirmed on-device: **Option-based** chords (e.g. `Alt+…`) are swallowed by
   macOS special-character input, and **`Cmd+Space`-family** chords are eaten by
   Spotlight/input-source switching — neither fires as a global shortcut, so
   avoid both.
5. **Use a stable signing identity** so Accessibility/Screen Recording grants
   survive rebuilds. Never produce a flow that re-prompts the user to grant
   Accessibility on every build.
6. **Surface microphone-permission failure explicitly** — without it macOS feeds
   empty audio silently and recording appears to work with a flat waveform.

## Backends behind traits
STT, TTS, and the refine LLM each sit behind a trait (`Transcriber`, `Speaker`,
`LlmChat`); all implementations are **on-device**. TTS has two backends selected
by `tts_provider` (native `AVSpeechSynthesizer` default, local neural Kokoro
optional); STT and refine are single-impl but keep the seam. Don't hardcode a
provider at a call site.

## Current status
Phases 0–3 shipped: Fn / chord dictation (on-device Whisper, `whisper-rs` with
Metal) with clipboard-paste injection, Fn+Ctrl LLM refinement (local Qwen3),
read-aloud TTS (native `AVSpeechSynthesizer` by default; read-aloud falls back to
the clipboard when nothing is selected), a settings window (hotkeys, engines,
mic/voice, local usage insights), a first-run onboarding window (Accessibility +
microphone grants and live model-download progress; gated on the
`onboarding_done` config flag, re-openable from the tray "Setup…" item),
auto-update (`tauri-plugin-updater` + minisign-signed artifacts; startup +
tray "Check for Updates…" checks surface an install banner in Settings —
`update.rs`; the endpoint points at the public releases `latest.json`, which
resolves once the repo is public — see `docs/launch-checklist.md`), and SQLite
dictation history. The TTS backend is
chosen via `tts_provider` in config; the local Whisper model (default `small.en`,
name in `stt_model`) auto-downloads to `<app-support>/murmur/models/` on first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letsgetrusty/Murmur](https://github.com/letsgetrusty/Murmur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
