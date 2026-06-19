---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`vo` is a macOS 26+ Apple Silicon CLI that performs **on-device** live transcription and translation. It listens to the user's microphone and the system speaker output simultaneously, transcribes both streams via Apple's `SpeechTranscriber`, optionally translates each finalized chunk through `TranslationSession`, and prints results to STDOUT. No network calls.

## Build / Run

```bash
swift build                            # debug build, produces .build/debug/vo
swift build -c release --arch arm64    # release
./scripts/build.sh                     # release + embeds Info.plist + ad-hoc codesign
.build/debug/vo --help                 # help
.build/debug/vo --doctor               # environment check (no TCC required)
```

## Test

```bash
./scripts/test.sh                      # Swift Testing suite (swift test under the hood)
```

Tests use Swift Testing (`import Testing`, `@Test`) in `Tests/voTests/`. They cover only the
TCC-free pure logic: `StreamRenderer` JSONL output (source-order commit, volatile suppression,
null-target on EOF), `VoError` messages, and `detectRenderMode`. The audio / Speech / Translation
paths require TCC grants and macOS 26 hardware, so they are not unit-tested.

`scripts/test.sh` exists because on a Command Line Tools-only install (no full Xcode) the Swift
Testing framework and its `lib_TestingInterop` dylib are off the default search / rpath; the script
injects the needed `-F` / `-rpath` flags only when that layout is detected. Under full Xcode (CI's
`macos-26` runner) it runs a plain `swift test`. CI lives in `.github/workflows/test.yml`.

## Runtime requirements

- macOS 26+ (uses `SpeechTranscriber`, `SpeechAnalyzer`, `TranslationSession`, all macOS 26 only)
- Apple Silicon (Neural Engine)
- TCC permissions granted on first run: Microphone, Speech Recognition, and (unless `--no-speaker` is passed) Audio Recording. The speaker channel uses a Core Audio process tap, **not** ScreenCaptureKit, so it needs only the Audio Recording grant, never Screen Recording. When launched from Terminal.app, the grants attach to Terminal.app rather than `vo` itself unless `vo` is properly bundled and signed.
- On-device models. `Pipeline.run()` resolves both before any channel starts (once, since both channels share `--src`). The speech model for `--src` downloads headlessly via `AssetInventory.downloadAndInstall()` on first run, with a one-line `Downloading speech model…` notice on **stderr** (so JSONL on stdout stays clean). The translation model for `--src → --dst` **cannot** be downloaded headlessly (the Translation framework only downloads via a UI sheet a CLI can't present), so `ensureTranslationModel` checks `LanguageAvailability.status` up front and fails fast with `VoError.translationModelNotInstalled` (install via System Settings) or `.unsupportedTranslationPair`, rather than letting every chunk surface `[translation failed]`.

## CLI surface

Flat command (no subcommands). `--doctor` is the only "different mode"; everything else configures the live listen loop.

```
vo [--src LOCALE] [--dst LOCALE] [--no-mic] [--no-speaker]
   [--voice-processing] [--select-device] [--transcript PATH] [--doctor] [--json]
```

- `--src` defaults to `Locale.current.identifier(.bcp47)`. Must be in `SpeechTranscriber.supportedLocales` (all regional, no bare `en` / `ja`). Unsupported values produce a helpful error suggesting matching regional variants.
- `--dst` is optional. Without it, `vo` is transcribe-only and never calls `TranslationSession`.
- `--voice-processing` enables AVAudioInputNode voice processing (echo cancellation + noise reduction). **Default off** because enabling it puts the OS audio session into communication mode and lowers system speaker volume. Use only when running mic + speaker on the same physical device without headphones.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k1LoW/vo](https://github.com/k1LoW/vo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
