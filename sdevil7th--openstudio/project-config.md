---
trigger: always_on
description: A hybrid DAW (Digital Audio Workstation) with a **JUCE C++ backend** for audio processing and a **React/TypeScript frontend** rendered in WebView2.
---

# Studio13-v3

A hybrid DAW (Digital Audio Workstation) with a **JUCE C++ backend** for audio processing and a **React/TypeScript frontend** rendered in WebView2.

## Architecture

```
C++ (JUCE) Backend          React/TypeScript Frontend
┌─────────────────────┐     ┌──────────────────────────┐
│ AudioEngine          │◄───►│ NativeBridge.ts          │
│ PlaybackEngine       │     │   (window.__JUCE__)      │
│ AudioRecorder        │     ├──────────────────────────┤
│ TrackProcessor       │     │ useDAWStore.ts (Zustand)  │
│ PluginManager        │     ├──────────────────────────┤
│ MIDIManager          │     │ Timeline.tsx (Konva)      │
│ Metronome            │     │ MixerPanel / ChannelStrip │
│ MainComponent        │     │ TransportBar / MenuBar    │
│   (WebBrowserComponent)    │ FXChainPanel / PianoRoll  │
└─────────────────────┘     └──────────────────────────┘
```

- **C++ backend** handles: audio I/O, recording to disk, clip playback with sample-rate conversion, VST3 plugin hosting, MIDI device management, metering, offline render/export
- **React frontend** handles: all UI, state management (Zustand), canvas-based timeline (Konva/react-konva), keyboard shortcuts, drag-and-drop, project save/load
- **Communication**: synchronous bridge via `window.__JUCE__.backend.*` functions (defined in NativeBridge.ts, exposed in MainComponent.cpp)

## Directory Structure

```
Studio13-v3/
├── Source/                      # C++ backend
│   ├── Main.cpp                 # JUCE app entry point
│   ├── MainComponent.h/cpp      # Hosts WebBrowserComponent + AudioEngine, exposes native functions to JS
│   ├── AudioEngine.h/cpp        # Core audio callback, device management, track graph, render, pitch correction entry point
│   ├── PlaybackEngine.h/cpp     # Clip playback scheduling, sample-rate-aware mixing, replaceClipAudioFile
│   ├── AudioRecorder.h/cpp      # Thread-safe recording via juce::AudioFormatWriter::ThreadedWriter
│   ├── TrackProcessor.h/cpp     # Per-track juce::AudioProcessor: metering, FX chain, input monitoring
│   ├── PluginManager.h/cpp      # VST3/CLAP/LV2 plugin scanning and loading
│   ├── PluginWindowManager.h/cpp# Native plugin editor window management
│   ├── CLAPPluginFormat.h/cpp   # CLAP plugin hosting (parameter discovery, GUI, state)
│   ├── MIDIManager.h/cpp        # MIDI device enumeration and input routing
│   ├── MIDIClip.h/cpp           # MIDI note event storage and time-range queries
│   ├── Metronome.h/cpp          # Click track generation (BPM, time sig, accent patterns)
│   ├── AudioConverter.h/cpp     # Channel/sample-rate conversion utilities
│   ├── PeakCache.h/cpp          # REAPER-style multi-resolution peak cache (.s13peaks sidecar files)
│   ├── AudioAnalyzer.h/cpp      # Audio analysis utilities
│   ├── BuiltInEffects.h/cpp     # Built-in audio effects (EQ, compressor, etc.)
│   ├── BuiltInEffects2.h/cpp    # Additional built-in effects
│   │
│   │   # Pitch Editor / Correction Pipeline (see "Pitch Editor Subsystem" below)
│   ├── PitchAnalyzer.h/cpp      # YIN monophonic pitch detection, note segmentation, pitchDrift
│   ├── PitchDetector.h/cpp      # Low-level YIN pitch detection algorithm
│   ├── PitchMapper.h/cpp        # Maps detected pitch to corrected pitch (scale/key snapping)
│   ├── PitchShifter.h/cpp       # Phase vocoder pitch shifter (FFT 2048, hop 512, FIFO-based)
│   ├── PitchResynthesizer.h/cpp # Offline graphical pitch correction via native VSF pitch-only renderer
│   ├── FormantPreserver.h/cpp   # WORLD vocoder (DIO+StoneMask+CheapTrick+D4C) for formant-preserving pitch shift (fallback)
│   ├── PolyPitchDetector.h/cpp  # Polyphonic pitch detection via Basic-Pitch ONNX model
│   ├── PolyResynthesizer.h/cpp  # Polyphonic pipeline: STFT→Wiener masks→per-note shift→accumulate→ISTFT
│   ├── HarmonicMaskGenerator.h/cpp # Wiener-filter soft masks at harmonic positions for poly separation
│   ├── SpectralPitchShifter.h/cpp  # Phase vocoder on masked spectrograms with cepstral formant preservation
│   ├── SpectralProcessor.h/cpp  # STFT/ISTFT utilities for spectral processing
│   ├── S13PitchCorrector.h/cpp  # Real-time inline pitch corrector (auto-tune style)
│   │
│   │   # Plugin System
│   ├── S13FXProcessor.h/cpp     # JSFX/Lua script-based audio processor (wraps YSFX)
│   ├── S13FXGfxEditor.h/cpp     # JSFX @gfx rendering via juce::Image framebuffer at 30fps
│   ├── S13PluginEditors.h/cpp   # Built-in plugin editor windows
│   ├── S13ScriptWindow.h/cpp    # Lua gfx API framebuffer window
│   ├── ScriptEngine.h/cpp       # Lua scripting engine (sol2)
│   │
│   │   # Other Features
│   ├── StemSeparator.h/cpp      # AI stem separation (vocals/drums/bass/other)
│   └── ARAHostController.h/cpp  # ARA plugin hosting controller
│
├── frontend/
│   ├── src/
│   │   ├── App.tsx              # Main layout: MenuBar → MainToolbar → workspace(TCP + Timeline) → TransportBar → LowerZone → Modals
│   │   ├── main.tsx             # React entry point
│   │   ├── index.css            # Tailwind theme with daw-* custom colors
│   │   ├── store/
│   │   │   ├── useDAWStore.ts   # Zustand store — all app state and actions (~3400 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdevil7th/OpenStudio](https://github.com/sdevil7th/OpenStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
