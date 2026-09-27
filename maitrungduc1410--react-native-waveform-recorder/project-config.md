---
trigger: always_on
description: This file is the entry point for AI coding agents working in this repo. Read it before making changes. It's deliberately short — for the full deep-dive read [ARCHITECTURE.md](./ARCHITECTURE.md), for the field report of past bugs and their lessons read [LESSONS_LEARNED.md](./LESSONS_LEARNED.md), and for the user-facing surface read [README.md](./README.md).
---

# AGENTS.md

This file is the entry point for AI coding agents working in this repo. Read it before making changes. It's deliberately short — for the full deep-dive read [ARCHITECTURE.md](./ARCHITECTURE.md), for the field report of past bugs and their lessons read [LESSONS_LEARNED.md](./LESSONS_LEARNED.md), and for the user-facing surface read [README.md](./README.md).

> **Read [LESSONS_LEARNED.md](./LESSONS_LEARNED.md) before changing any state-machine, async-callback, native-gesture, memory, or codegen-shape code.** Most of the subtle bugs we've already hit live there with both the root cause and the fix pattern. Re-deriving them from scratch wastes context.

---

## What this repo is

`react-native-waveform-recorder` — a React Native (Fabric / New Architecture) audio recorder with a native live waveform. Standalone (zero JS peer deps). Pairs with [`react-native-waveform-player`](https://github.com/maitrungduc1410/react-native-waveform-player) for the playback half of a voice-message product.

- **Languages**: TypeScript (JS surface), Swift (iOS native), Kotlin (Android native), a tiny Objective-C++ bridge (`ios/WaveformRecorderView.mm`).
- **Build target**: React Native 0.85+ with New Architecture (Fabric + TurboModules). iOS 13+, Android API 24+ (API 29+ for opus).
- **Package manager**: **Yarn 4 workspaces.** Do not use `npm`.

---

## Repo layout

```
.
├── src/                                   JS/TS public surface
│   ├── WaveformRecorderViewNativeComponent.ts   Codegen spec (source of truth)
│   ├── WaveformRecorderView.tsx                 Public types + non-native fallback
│   ├── WaveformRecorderView.native.tsx          Native wrapper (ref, permissions, CSV→array)
│   ├── pcm-stream/index.tsx                     Opt-in PCM helpers (subpath import)
│   └── index.tsx                                Public re-exports
│
├── ios/                                   Native iOS sources
│   ├── WaveformRecorderView.h / .mm             Objective-C++ Fabric bridge
│   ├── WaveformRecorderViewImpl.swift           Composite view, state machine, layout
│   ├── AudioRecorderEngine.swift                Recording engine (AVAudioRecorder/AudioRecord)
│   ├── AudioPlayerEngine.swift                  Preview playback (AVPlayer)
│   ├── WaveformBarsView.swift                   Live + preview ribbon, scrub gesture
│   ├── WaveformDecoder.swift                    64-bucket downsampler
│   └── PlayPauseButton.swift                    Preview-only play/pause control
│
├── android/src/main/java/com/waveformrecorder/   Native Android sources
│   ├── WaveformRecorderPackage.kt
│   ├── WaveformRecorderViewManager.kt           Fabric view manager
│   ├── WaveformRecorderView.kt                  Composite view (mirror of iOS impl)
│   ├── AudioRecorderEngine.kt                   Recording engine
│   ├── AudioPlayerEngine.kt                     Preview playback
│   ├── WaveformBarsView.kt                      Live + preview ribbon
│   ├── WaveformDecoder.kt                       64-bucket downsampler
│   ├── PlayPauseButton.kt                       Preview-only play/pause control
│   ├── WaveformRecorderBackgroundService.kt     Microphone foreground service
│   └── WaveformRecorderEvent.kt                 DirectEvent helpers
│
├── plugin/                                Expo config plugin (plain JS, no build step)
│   └── withWaveformRecorder.js                  iOS mic/UIBackgroundModes + Android service
├── app.plugin.js                          Expo plugin entry point (re-exports plugin/)
│
├── example/                               Comprehensive example app + recipes
│   ├── src/App.tsx                              Stack navigator entry
│   ├── src/screens/*.tsx                        Per-screen demos + recipes
│   └── src/components/*.tsx                     Shared UI (SentVoiceNote, etc.)
│
├── README.md                              User-facing docs
├── ARCHITECTURE.md                        Internals, threading, memory bounds
├── CONTRIBUTING.md                        How to set up + run + PR
└── package.json                           Library manifest + codegen config
```

When adding files, follow the existing patterns — don't introduce parallel folders.

---

## Common commands

Run from repo root unless stated otherwise.

| Command | What it does |
| --- | --- |
| `yarn` | Install deps (root + example workspace). |
| `yarn typecheck` | TypeScript check of the library + example. |
| `yarn lint` | ESLint over `**/*.{js,ts,tsx}`. |
| `yarn lint --fix` | Auto-fix lint + format. |
| `yarn example start` | Start Metro for the example app. |
| `yarn example android` | Build + install + run the example app on Android. |
| `yarn example ios` | Build + install + run the example app on iOS. |
| `yarn example build:android` | Headless Android Debug build (CI-friendly). |
| `yarn example build:ios` | Headless iOS Debug build. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/react-native-waveform-recorder](https://github.com/maitrungduc1410/react-native-waveform-recorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
