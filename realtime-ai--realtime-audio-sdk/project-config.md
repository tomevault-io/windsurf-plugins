---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Realtime Audio SDK for the Web — provides audio capture, voice activity detection (VAD), and real-time encoding (Opus/PCM). Designed for real-time transcription, translation, and AI voice conversations.

## Development Commands

```bash
# Install dependencies
npm install

# Development server (runs examples)
npm run dev

# Build library
npm run build

# Type checking
npm run type-check

# Run tests
npm test
npm run test:ui
npm run test:coverage
```

## Project Structure

```
src/
├── core/
│   ├── EventEmitter.ts       # Event system base class
│   └── RTA.ts   # Main SDK entry point
├── capture/
│   ├── AudioCapture.ts            # Audio capture with AudioWorklet
│   └── audio-worklet-processor.ts # Worklet processor (runs in audio thread)
├── devices/
│   └── DeviceManager.ts      # Device enumeration and hot-plug detection
├── processing/
│   └── AudioProcessor.ts     # VAD and audio normalization
├── encoding/
│   ├── OpusEncoder.ts        # WebCodecs Opus encoder
│   └── PCMEncoder.ts         # Fallback PCM encoder
└── types/
    └── index.ts              # TypeScript type definitions
```

## Architecture

### Audio Pipeline Flow

1. **Device Management** ([DeviceManager.ts](src/devices/DeviceManager.ts))
   - Lists available audio input devices
   - Monitors device hot-plug events
   - Auto-switches on device removal (if enabled)

2. **Audio Capture** ([AudioCapture.ts](src/capture/AudioCapture.ts))
   - Uses `getUserMedia` to access microphone
   - Creates `AudioContext` with specified sample rate
   - Loads AudioWorklet processor for precise time chunking

3. **AudioWorklet Processing** ([audio-worklet-processor.ts](src/capture/audio-worklet-processor.ts))
   - Runs in separate audio thread
   - Buffers 128-frame blocks from Web Audio API
   - Extracts exact frame counts (320/640/960 frames for 20/40/60ms @ 16kHz)
   - Sends chunks to main thread via `postMessage`

4. **Audio Processing** ([AudioProcessor.ts](src/processing/AudioProcessor.ts))
   - Normalizes audio to [-1, 1] range (optional)
   - Calculates energy (RMS)
   - Performs energy-based VAD with hysteresis

5. **Encoding** ([OpusEncoder.ts](src/encoding/OpusEncoder.ts) / [PCMEncoder.ts](src/encoding/PCMEncoder.ts))
   - OpusEncoder: Uses WebCodecs API for Opus encoding
   - PCMEncoder: Fallback for browsers without WebCodecs
   - Converts Float32 to Int16 PCM or Opus frames

6. **Event Emission** ([RTA.ts](src/core/RTA.ts))
   - Emits `audio-data` events with encoded chunks
   - Emits `processed-audio` events with VAD results
   - Emits device and state change events

### Key Design Patterns

- **EventEmitter Pattern**: All modules extend EventEmitter for loose coupling
- **Async Initialization**: Encoders and capture must be initialized before use
- **Resource Cleanup**: All modules provide cleanup methods (close/stop/destroy)
- **Graceful Degradation**: Falls back to PCM when WebCodecs unavailable

### Important Implementation Details

1. **Precise Time Chunking**
   - AudioWorklet receives 128-frame blocks (Web Audio API standard)
   - Processor buffers blocks and extracts exact frame counts
   - Formula: `frames = (frameSize_ms * sampleRate) / 1000`

2. **Device Switching**
   - Stops current stream
   - Creates new MediaStream with new deviceId
   - Reconnects AudioWorklet
   - Optionally resumes recording state

3. **VAD Hysteresis**
   - Requires sustained energy above threshold before marking as speech
   - Requires sustained silence before marking as non-speech
   - Prevents flickering on threshold boundary

4. **WebCodecs Opus Configuration**
   - Codec: 'opus'
   - frameDuration in microseconds (frameSize * 1000)
   - Format: 'opus' (raw Opus packets)

## Common Tasks

### Adding a New Audio Processor

1. Create processor class in `src/processing/`
2. Add configuration to `ProcessingConfig` in [types/index.ts](src/types/index.ts)
3. Integrate in [AudioProcessor.ts](src/processing/AudioProcessor.ts) process method
4. Update SDK config in [RTA.ts](src/core/RTA.ts)

### Adding a New Encoder

1. Create encoder class in `src/encoding/`
2. Implement interface: `initialize()`, `encode()`, `flush()`, `close()`
3. Add codec type to `AudioCodec` in [types/index.ts](src/types/index.ts)
4. Add initialization logic in [RTA.ts](src/core/RTA.ts):initializeEncoder()

### Modifying AudioWorklet

- AudioWorklet code must be inlined as string or loaded from blob URL
- Cannot use ES6 imports in worklet context
- Must use `postMessage` for communication with main thread
- See [AudioCapture.ts](src/capture/AudioCapture.ts):getWorkletCode()

## Browser Compatibility Notes

- **AudioWorklet**: Chrome 66+, Firefox 76+, Safari 14.1+
- **WebCodecs**: Chrome 94+, Edge 94+, Safari 16.4+
- **getUserMedia**: All modern browsers

Always check feature availability:
```typescript
if (!('AudioEncoder' in window)) {
  // Use PCM fallback
}
```

## Testing Considerations

- Audio APIs require user gesture in browsers (can't automate easily)
- Use mocked MediaStream for unit tests
- Integration tests need actual microphone access
- Test device switching with virtual audio devices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realtime-ai/realtime-audio-sdk](https://github.com/realtime-ai/realtime-audio-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
