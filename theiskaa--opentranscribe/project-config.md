---
trigger: always_on
description: Your memory is OptMem:
---

# opentranscribe

## Memory

Your memory is OptMem:

- The tool is `~/.optmem/memo`
- Your memories are in `~/.optmem/memory`

OptMem outlives every session, compaction, model and vendor change.
Without it you do not know who you are, or what was decided and tried.

### At startup: activating OptMem (mandatory)

Run `~/.optmem/memo wake` before any other tool call, in every session, and
then do exactly what it prints, to the end of its output.

### While working: register memories (mandatory)

Call `~/.optmem/memo note "<1 line, max 280 chars>"` whenever you learn
something new, or something worth keeping happens. That covers a task
worth real effort, a fact or insight the user teaches you, anything you
learn about their life (even indirectly), any event of lasting effect.

Do not register redundant memories.

If `~/.optmem/memo note` asks a compression: do it before your next action.

Never edit or delete anything under `~/.optmem/memory`: the tool manages it.

### When you need an old memory: search, or navigate

`~/.optmem/memo recall <regex>` searches every memory, word for word.

Your memories also form a binary tree: #0-1, #2-3 ... exist as one-line
summaries, pairs of those as #0-3, and so on -- every `#a-b` line wake
prints is one node of it. `~/.optmem/memo zoom <a-b>` opens a node into its
two halves, down to the raw memories.

### If you're a subagent: skip everything above

## What this is

A private, offline voice journal. You speak your mind, it writes it down, and it stays on your device. Flutter, iOS-first, no other platform is supported today.

## The one rule

**Nothing leaves the phone.** No network calls, no accounts, no analytics, no third-party SDK that phones home. The app must work fully in airplane mode. This is not a feature, it is the architecture, and it constrains every decision below. If a change would open a socket or ship data off-device, it does not belong here. The one carve-out, and the only connection the app ever opens: fetching a public Whisper model file the user asked for (the model, or the Neural Engine encoder the switch under the models adds), from one pinned host, through one allowlisted file (`packages/transcriber/lib/src/whisper/model_fetcher.dart`). It sends nothing, and `test/one_rule_test.dart` holds the tree to exactly that file and that host.

Corollaries that shape the code:

- Transcription runs on-device, behind one contract: `TranscriptionEngine` in `packages/transcriber`. Three engines ship, user-switchable on the transcription screen: `AppleSpeechEngine` (the iOS 26 SpeechAnalyzer, with managed model downloads), `AppleDictationEngine` (the classic recognizer behind iOS dictation), and `WhisperEngine` (whisper.cpp over `dart:ffi`, batch-only, one downloaded model of five serving every language it knows). Streaming, downloadable-model, model-choice, paced-batch, progress-reporting and acceleration behavior are separate interfaces an engine may also implement, not flags: `StreamingTranscriptionEngine`, `ManagedModelEngine`, `ModelChoiceEngine`, `PacedBatchEngine`, `ReleasableEngine`, `ProgressBatchEngine`, `AcceleratedModelEngine`.
- `TranscriptionEngine.onDeviceOnly` is a hard gate. The app refuses an engine that answers false, so nothing can quietly route audio off the phone.
- Nothing in `view/`, `core/services/`, or `core/state/` names a concrete engine. `Deps.init()` is the only place allowed to, plus the engine registry it builds (`EngineEntry` list in `core/app/engine_registry.dart`) for every surface that lists engines: registry order is preference order, auto mode runs the first available entry, and the stored choice lives in `EngineSettings`.
- Audio capture is recorder-owned, not engine-owned. Buffers stay native; only paths, durations, levels and text cross a channel. Raw audio for each entry is kept on-device by default so entries can be re-transcribed later by a better engine. Keeping is a preference: with keep-audio off, a recording is deleted after its first successful transcription and the entry becomes transcript-only (`Entry.audioPath` is nullable). Bulk reclaim of kept history is only ever the Cache screen's explicit, confirmed action.
- Continuing an entry records a normal take and merges it onto the kept file natively through `AudioComposer`; the merged file replaces the old one only after the entry is saved. The new words are stitched onto what the entry reads as; a base never transcribed gets one pass over the merged file instead, and one without a recording takes the take as its file, its old words untimed.
- The supporter purchase is direct StoreKit 2 (`SupportStore.swift` under `ios/Runner/`, wrapped by `core/support/`), no third-party purchase SDK and no server. The OS talks to the App Store; no journal content is in that conversation, entitlements are verified on-device from StoreKit's own record, and only the act of buying needs a connection. The club gates looks only; no function is ever gated, and `test/core/no_club_gate_test.dart` keeps it that way: nothing under `core/services/` or `core/state/` may ask for the tier except the support, theme, and app icon holders.

## Architecture

Two layers only. There is no `features/` layer, and we do not want one.

`lib/core/`, everything non-UI:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theiskaa/opentranscribe](https://github.com/theiskaa/opentranscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
