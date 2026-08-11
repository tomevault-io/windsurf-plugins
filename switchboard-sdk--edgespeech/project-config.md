---
trigger: always_on
description: MOST IMPORTANT: Don't make any assumptions without asking my opinion.
---

MOST IMPORTANT: Don't make any assumptions without asking my opinion.

## Environment Requirements

- **Node.js 20+** required (use `nvm use 20` before running npm/expo commands)
- **Don't call `pod install` directly** - use `npx expo run:ios` instead (CocoaPods is deprecated in React Native)

## Process Management

- Keep a log of your work in `PROGRESS.md` (session-based chronological log)
- Maintain TODO list in `TODO.md` (task checklist organized by phase)
- Don't work in /tmp, use ./tmp and git ignore it
- Check in with the user after completing each phase before moving to the next phase

# Switchboard Voice Toolkit for React Native

## Project Goal

Build a **React Native library** (npm package) that wraps Switchboard SDK's on-device voice processing into a simple text-based callback interface. Ship with an example app that demonstrates usage.

**Platform:** iOS only (initial release)
**Language:** English only (initial release)

## Repository Purpose

**The primary deliverable is the React Native module** - not the example app. The example exists only to demonstrate how to use the module.

**User workflow:**

1. Clone this repo
2. Run setup scripts (to download SDK frameworks)
3. Build and run the example app to verify everything works
4. Use the module in their own React Native app

The module must be usable independently of the example app. Users should be able to import it into their own projects.

## Deliverables

1. **`switchboard-voice-rn`** - The React Native library (npm package) - **PRIMARY**
2. **`example/`** - A minimal app that demonstrates the library

## Core Value Proposition

Voice AI developers work entirely in text. The library handles all audio complexity:

- On-device VAD (Voice Activity Detection)
- On-device STT (Speech-to-Text via Whisper)
- On-device TTS (Text-to-Speech via Silero)
- Simple JavaScript callbacks and methods

## Target API

```typescript
import { SwitchboardVoice } from 'switchboard-voice-rn'

// Configuration
SwitchboardVoice.configure({
  sttModel: 'whisper-base-en',
  ttsVoice: 'silero-en-us',
  vadSensitivity: 0.5,
})

// Event handlers
SwitchboardVoice.onTranscript = (text: string, isFinal: boolean) => {}
SwitchboardVoice.onInterrupted = () => {}
SwitchboardVoice.onError = (error: VoiceError) => {}
SwitchboardVoice.onStateChange = (state: VoiceState) => {}

// Actions
await SwitchboardVoice.start()
await SwitchboardVoice.stop()
await SwitchboardVoice.speak(text)
await SwitchboardVoice.stopSpeaking()
```

## Project Structure

```
switchboard-voice-rn/
├── CLAUDE.md
├── package.json                     # Library package config
├── tsconfig.json
├── switchboard-voice-rn.podspec    # CocoaPods spec for iOS native code
├── src/                             # JS/TS source for the library
│   ├── index.ts                     # Main export
│   ├── SwitchboardVoice.ts          # JS API wrapper
│   └── types.ts                     # TypeScript definitions
├── ios/                             # Native iOS implementation
│   ├── SwitchboardVoiceModule.swift
│   ├── SwitchboardVoiceModule.m     # RN bridge header
│   ├── AudioGraphManager.swift      # Manages Switchboard graphs
│   └── Bridging-Header.h
└── example/                         # Example app (separate RN project)
    ├── package.json                 # Depends on parent library
    ├── App.tsx
    ├── ios/
    │   └── Podfile                  # Links to parent library
    └── ...
```

## Library vs App Separation

### The Library (`/src`, `/ios`)

- No UI code
- Exposes `SwitchboardVoice` API
- Handles all Switchboard SDK integration
- Published to npm

### The Example App (`/example`)

- Imports library via `"switchboard-voice-rn": "link:.."`
- Demonstrates the full voice loop
- Minimal UI: start button, transcript display, text input for TTS

## Reference Implementations

### Primary Reference: daw-react-native

```
git@github.com:switchboard-sdk/daw-react-native.git
```

Use this as the structural reference for:

- React Native library structure (not app structure)
- Podspec configuration for Switchboard extensions
- Native module bridging patterns
- How to link an example app to the parent library

Pull apart and adapt patterns from this codebase freely.

### Secondary Reference: voice-app-control-example-ios

```
https://github.com/switchboard-sdk/voice-app-control-example-ios
```

Native iOS example showing VAD + Whisper STT pipeline. Key patterns:

- JSON-based AudioGraph configuration
- Silero VAD → Whisper STT node connections
- Event listener setup for transcription callbacks

## AudioGraph Configurations

### Node Types (SDK 3.1.0)

| Extension   | Node Type       | Action                                |
| ----------- | --------------- | ------------------------------------- |
| SileroVAD   | `SileroVAD.VAD` | -                                     |
| Whisper STT | `Whisper.STT`   | `transcribe` (params: `start`, `end`) |
| Sherpa TTS  | `Sherpa.TTS`    | `synthesize` (param: `text`)          |

Note: Engine type is `"Realtime"` (not `"RealTimeGraphRenderer"`).

### Event Names (IMPORTANT)

For `SileroVAD.VAD` node:

- **Events**: `speechStarted`, `speechEnded` (NOT `start`/`end`)
- **Data connection format**: `vadNode.speechEnded` → `sttNode.transcribe`

For `Whisper.STT` node:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [switchboard-sdk/EdgeSpeech](https://github.com/switchboard-sdk/EdgeSpeech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
