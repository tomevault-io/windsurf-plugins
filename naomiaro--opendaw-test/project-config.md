---
trigger: always_on
description: This project demonstrates headless usage of the OpenDAW SDK for browser-based audio recording and playback.
---

# OpenDAW Headless Development Guide

## Project Overview
This project demonstrates headless usage of the OpenDAW SDK for browser-based audio recording and playback.

## Key OpenDAW APIs

### Reactive Box Graph Subscriptions (pointerHub)
```typescript
// Prefer pointerHub subscriptions over AnimationFrame polling for structural changes.
// Use AnimationFrame ONLY for continuous rendering (e.g., waveform peaks at 60fps).

// Reactive subscription chain: audioUnit → tracks → regions → field changes
const subs: Terminable[] = [];
const trackSub = audioUnitBox.tracks.pointerHub.catchupAndSubscribe({
  onAdded: (pointer) => {
    const trackBox = pointer.box;
    const regionSub = (trackBox as any).regions.pointerHub.catchupAndSubscribe({
      onAdded: (regionPointer: any) => {
        const regionBox = regionPointer.box as AudioRegionBox;
        // Subscribe to scalar field changes (e.g., mute)
        const muteSub = regionBox.mute.subscribe((obs: any) => {
          const isMuted = obs.getValue();
        });
        subs.push(muteSub);
      },
      onRemoved: () => {},
    });
    subs.push(regionSub);
  },
  onRemoved: () => {},
});
subs.push(trackSub);
// Cleanup: terminate all subs (outer first to prevent cascading callbacks)
```

**Key rules:**
- `catchupAndSubscribe` fires immediately for existing data + future changes (preferred)
- `subscribe` fires only for future changes (use when initial state already known)
- `pointerHub.incoming()` is a snapshot read, NOT reactive
- Pointer callbacks receive `PointerField` — access box via `pointer.box`
- Terminate pointer hub subs BEFORE engine cleanup when stopping recording
- After recording stops, reactive subscriptions are terminated — update React state directly for user-initiated changes (e.g., mute toggle)

### SoundfontService (Disabled via Proxy Guard)
- `SoundfontService` constructor auto-fetches `api.opendaw.studio/soundfonts/list.json` (CORS error in dev)
- SDK declares `soundfontService` in `ProjectEnv` but never reads it (verified in 0.0.129)
- We pass a Proxy that throws a clear error if a future SDK version accesses it
- None of the demos use soundfont instruments (MIDI demo uses Vaporisateur built-in synth)

### SampleService (SDK 0.0.124+)
- `new SampleService(audioContext)` required in `ProjectEnv` for recording finalization
- `CaptureAudio.prepareRecording()` injects it into `RecordingWorklet` automatically

### Engine State Observables
```typescript
// Subscribe to engine state changes
project.engine.isRecording.catchupAndSubscribe(obs => {
  const recording = obs.getValue();
});

project.engine.isPlaying.catchupAndSubscribe(obs => {
  const playing = obs.getValue();
});

project.engine.isCountingIn.catchupAndSubscribe(obs => {
  const countingIn = obs.getValue();
});

project.engine.countInBeatsRemaining.catchupAndSubscribe(obs => {
  const beats = Math.ceil(obs.getValue());
});
```

### Engine Preferences (SDK 0.0.87+)
```typescript
// Access via project.engine.preferences.settings
const settings = project.engine.preferences.settings;

// Metronome
settings.metronome.enabled = true;
settings.metronome.gain = -6; // dB
settings.metronome.beatSubDivision = 1; // 1=quarter, 2=eighth, 4=16th, 8=32nd

// Recording
settings.recording.countInBars = 1; // 1-8 bars
```

### AudioContext Suspension
Browser autoplay policy means `AudioContext` starts suspended until a user gesture.
`initializeOpenDAW()` registers click/keydown listeners to auto-resume it (one-shot).
iOS Safari can re-suspend after backgrounding/locking. Before calling `play()`:
```typescript
if (audioContext.state !== "running") {
  await audioContext.resume();
  // iOS Safari may not be "running" yet — wait for statechange event
}
```

## Important Patterns

### Option Types Are Always Truthy
OpenDAW uses Option types that are **always truthy** (even `Option.None`):
```typescript
// WRONG - Option.None is truthy, this never triggers
if (!sampleLoader.peaks) { ... }

// CORRECT
const peaksOption = sampleLoader.peaks;
if (peaksOption.isEmpty()) { return; }
const peaks = peaksOption.unwrap();
```
API: `.isEmpty()`, `.nonEmpty()`, `.unwrap()`, `.unwrapOrNull()`, `.unwrapOrUndefined()`

### Always Use editing.modify() for State Changes
```typescript
project.editing.modify(() => {
  // All box graph modifications go here
  project.timelineBox.bpm.setValue(120);
});
```

### Pointer Re-Routing: Separate Transaction from Creation
`createInstrument()` internally routes `audioUnitBox.output` to master. Re-routing with
`output.refer(newTarget)` in the same `editing.modify()` may not disconnect the old
connection, causing dual routing. Always re-route in a separate transaction. Similarly,
`targetVertex` traversal on pointers created in the same transaction may return stale data.
This also applies to `captureDevices.get(uuid)` — resolve captures and set their fields
(deviceId, requestChannels) in a **separate** transaction after `createInstrument` commits.

### createInstrument Must Be Destructured Inside editing.modify()
`project.api.createInstrument()` returns `{ audioUnitBox, trackBox }` directly — no `.unwrap()`.
But `editing.modify()` does NOT forward return values, so capture via outer variable:
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naomiaro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
