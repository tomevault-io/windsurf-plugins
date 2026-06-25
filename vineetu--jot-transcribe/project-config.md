---
trigger: always_on
description: Native macOS dictation utility. Press a hotkey, speak, and the transcript is pasted at the cursor. Core transcription stays on-device; optional AI features can use Apple Intelligence, local Ollama, or user-configured cloud providers. No telemetry.
---

# Jot

Native macOS dictation utility. Press a hotkey, speak, and the transcript is pasted at the cursor. Core transcription stays on-device; optional AI features can use Apple Intelligence, local Ollama, or user-configured cloud providers. No telemetry.

**Stack:** Swift / SwiftUI with AppKit interop (`NSStatusItem`, `NSPanel`). Transcription via [FluidAudio](https://github.com/FluidInference/FluidAudio) running Parakeet TDT 0.6B v3 on the Apple Neural Engine. Audio capture through `AVAudioEngine` + `AVAudioConverter` (16 kHz mono Float32). Global hotkeys via `sindresorhus/KeyboardShortcuts`. Persistence via SwiftData; prefs via `@AppStorage` / `UserDefaults`.

**Platform:** Apple Silicon only, macOS Sequoia 15.0+. Intel Macs are out of scope — Parakeet on the ANE is an Apple Silicon feature. (Deployment floor raised 14→15 to adopt CoreMLLLM for on-device AI search.)

Full product requirements live in `docs/design-requirements.md` and the shipping feature inventory in `docs/features.md`. **Read those before making non-trivial decisions.** This file is a map, not the spec.

---

## Architecture layers

Single Xcode project, one executable target. Each layer is a Swift function boundary — no IPC, no serialization between stages.

| Layer | Responsibility |
|---|---|
| **App** | `@main` entry point, scenes, `AppDelegate`, top-level observable state, permission checks |
| **MenuBar** | `NSStatusItem` owner + native `NSMenu`; dynamic "Start / Stop Recording" label; "Open Jot…", Recent Transcriptions, and "Check for Updates…" |
| **MainWindow** | Single `NSWindow` shell with a source-list sidebar (Home / Ask Jot / Settings / Help / About); owns routing between sections, sidebar history, and the deep-link contract between Settings, Help, and Ask Jot |
| **Home** | Landing pane: hotkey glance, dismissible first-run banner, full recordings search/list/detail surface |
| **Ask Jot** | Conversational help chatbot pane; grounded in bundled `help-content.md`; Apple Intelligence default, optional cloud routing; markdown answers, voice input, and in-app feature citations |
| **AskJot/Cloud** | Provider-specific streaming adapters (`OpenAI`, `Anthropic`, `Gemini`, `Ollama`) plus inline tool-calling for feature-slug navigation when cloud Ask Jot is enabled |
| **Overlay** | `NSPanel`-hosted SwiftUI status indicator (Dynamic Island-style pill under the notch) |
| **Recording** | `AVAudioEngine` tap → converter → buffer + WAV on disk; hotkey routing with dynamic Escape; CoreAudio device pinning |
| **Transcription** | FluidAudio wrapper (single in-flight), post-processing, model download/load |
| **Delivery** | Clipboard sandwich: save → write → synthetic `⌘V` → restore; optional auto-Enter |
| **Library** | SwiftData models — `Recording` (dictation) + `RewriteSession` (rewrite runs) — and the merged `LibraryItem`-driven Home list, detail views, playback (recordings only), and per-row actions |
| **Settings** | Sidebar section (not a separate scene): General / Transcription / Vocabulary / Sound / AI / Shortcuts. Per-field `info.circle` popovers with "Learn more →" deep-links into Help. Editable LLM prompts under `CustomizePromptDisclosure` |
| **Help** | In-app prose walkthrough: Basics / Advanced / Troubleshooting. Accepts deep-links from Settings popovers, Ask Jot feature links, and Help hero sparkle affordances |
| **LLM** | Provider-neutral client for transcript cleanup (Transform) + Rewrite; Apple Intelligence (on-device, default for new installs on macOS 26+), OpenAI, Anthropic, Gemini, Ollama. Apple Intelligence bypasses the HTTP client entirely and calls the on-device `FoundationModels` framework via `AppleIntelligenceClient`. Rewrite uses a regex instruction classifier (`RewriteInstructionClassifier`) to route to one of four branch prompts — voice-preserving / structural / translation / code — composed on top of a small shared-invariants block |
| **Rewrite** | Two hotkeys, one pipeline. `.rewriteWithVoice` (v1.4; raw KeyboardShortcuts storage key `rewriteSelection` preserved across the v1.4→v1.6 Swift symbol rename): selection → synthetic ⌘C → record voice instruction → classify → branch-specific LLM prompt → paste back. `.rewrite` (v1.5; raw storage key `articulate` preserved): selection → synthetic ⌘C → fixed `"Rewrite this"` instruction → LLM → paste back. No voice capture on the fixed-prompt path |
| **SetupWizard** | First-run window: Welcome → Permissions → Model → Microphone → Shortcuts → Test |
| **Sounds** | Bundled chimes wrapped in a thin `AVAudioPlayer` helper |

**Four distinct privacy capabilities** (not one boolean): Microphone, Input Monitoring, Accessibility post-events, and optional full AX trust. Each has its own grant flow and revocation behavior. Denied post-events degrades to clipboard-only delivery with a toast — never a dead end.

---

## File / directory ownership


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vineetu/JOT-Transcribe](https://github.com/vineetu/JOT-Transcribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
