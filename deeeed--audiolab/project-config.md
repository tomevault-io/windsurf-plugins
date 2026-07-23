---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**expo-audio-stream** is a comprehensive audio processing monorepo for React Native and Expo applications. It provides real-time audio recording, analysis, visualization, and AI-powered processing capabilities across iOS, Android, and web platforms.

### Core Packages
- **`@siteed/expo-audio-studio`** - Main audio processing library with dual-stream recording, device management, and format conversion
- **`@siteed/audio-ui`** - React Native Skia-based audio visualization components (waveforms, spectrograms)
- **`@siteed/react-native-essentia`** - Advanced audio analysis using Essentia (feature extraction, classification)
- **`@siteed/sherpa-onnx.rn`** - Speech-to-text and text-to-speech capabilities (development)

### Applications
- **`apps/playground`** - Full-featured demo app showcasing all capabilities
- **`apps/minimal`** - Simple integration example
- **`apps/essentia-demo`** - Audio analysis demonstrations

## Agent Constraints
1. **NEVER IMPLEMENT UNLESS ASKED** - No unsolicited changes
2. **ALWAYS VERIFY IN SOURCE CODE** - No hallucinations accepted
3. **MINIMIZE DIFF** - Smallest possible changes
4. **NO WORKAROUNDS** - Fix root causes, not symptoms
5. **REAL TESTING ONLY** - No simulated results accepted

## Essential Commands

```bash
yarn install                    # Install dependencies
./scripts/setup-lfs.sh         # Setup Git LFS for models
cd apps/playground && yarn build:deps && yarn start  # Run playground app

# Building packages
yarn workspace @siteed/expo-audio-studio build
yarn workspace @siteed/audio-ui build
```

## Agentic Validation (CDP Bridge)

All agent commands run from `apps/playground/`. See `apps/playground/docs/AGENT_STARTING_TEMPLATE.md` for the step-by-step workflow.

```bash
node scripts/agentic/cdp-bridge.mjs list-devices     # List connected devices
scripts/agentic/app-navigate.sh "/(tabs)/record"      # Navigate
scripts/agentic/app-state.sh state                    # Query state
scripts/agentic/app-state.sh eval "__AGENTIC__.startRecording({ sampleRate: 44100, channels: 1 })"
scripts/agentic/screenshot.sh my-label                # Screenshot
scripts/agentic/reload-metro.sh                       # Hot reload after edits
scripts/agentic/native-logs.sh android|ios            # Kotlin/Swift logs
scripts/agentic/start-metro.sh                        # Start Metro (:7365)
```

All scripts accept `--device <name>` for multi-device targeting.

### Native Module Validation (fire-and-store pattern)

The agentic bridge exposes test methods for validating native module calls on-device. Since CDP uses `awaitPromise: false`, async results are stored and polled via `getLastResult()`.

```bash
# Test extractPreview (also exercises extractAudioAnalysis)
scripts/agentic/app-state.sh eval "__AGENTIC__.testExtractPreview()"
sleep 5
scripts/agentic/app-state.sh eval "__AGENTIC__.getLastResult()"

# Test extractAudioData
scripts/agentic/app-state.sh eval "__AGENTIC__.testExtractAudioData()"
sleep 3
scripts/agentic/app-state.sh eval "__AGENTIC__.getLastResult()"

# Test trimAudio
scripts/agentic/app-state.sh eval "__AGENTIC__.testTrimAudio()"
sleep 5
scripts/agentic/app-state.sh eval "__AGENTIC__.getLastResult()"

# Test extractMelSpectrogram (Android only)
scripts/agentic/app-state.sh eval "__AGENTIC__.testExtractMelSpectrogram()"
sleep 5
scripts/agentic/app-state.sh eval "__AGENTIC__.getLastResult()"
```

Each returns `{ op, status: 'pending' }` immediately. Poll `getLastResult()` for `status: 'success'` or `status: 'error'`. Always check `native-logs.sh android` for Kotlin bridge crashes after running these.

## Important Notes

- Always run `yarn build:deps` in playground before development
- Git LFS setup required for ONNX models
- Native changes require pod install (iOS) or gradle sync (Android)
- Background recording requires special permissions configuration
- **VPN Interference**: Disconnect VPNs during iOS E2E tests; Android uses ADB port forwarding (VPN-resistant)

## Dependency Strategy (Expo / React Native)

This monorepo uses Yarn 4 with `nodeLinker: node-modules` and `nmHoistingLimits: workspaces`. Each workspace gets its own `node_modules`, which makes it easy for nested packages to ship duplicate copies of singleton runtime packages such as `react`, `react-native`, or React Navigation packages. Two copies of any singleton in one bundle = silent context Symbol mismatch and a `<Stack>` / `<SceneView>` "Element type is invalid" crash that points at an unrelated provider in the stack trace.

Guard rails in place:
- Root `package.json` `resolutions` pins `react` and `react-dom` to one version per name.
- Expo Router SDK 56 bundles its React Navigation internals; app code should import router-owned entrypoints such as `expo-router/react-navigation`, `expo-router/js-tabs`, and `expo-router/stack` instead of direct `@react-navigation/*` packages.
- Apps may still provide `@react-navigation/native` only when a non-router library declares it as an external peer, for example `@siteed/design-system`; do not use that package for app-code navigation imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deeeed/audiolab](https://github.com/deeeed/audiolab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
