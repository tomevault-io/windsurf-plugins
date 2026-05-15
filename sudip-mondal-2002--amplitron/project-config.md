---
trigger: always_on
description: This document outlines the **Agent Architecture** within the Amplitron Guitar Amp Simulator. In the context of this high-performance DSP (Digital Signal Processing) pipeline, an "Agent" is defined as an autonomous, encapsulated component responsible for specific processing, coordination, or state management tasks.
---

# Amplitron Processing Agents & System Architecture

This document outlines the **Agent Architecture** within the Amplitron Guitar Amp Simulator. In the context of this high-performance DSP (Digital Signal Processing) pipeline, an "Agent" is defined as an autonomous, encapsulated component responsible for specific processing, coordination, or state management tasks. 

These agents operate concurrently across high-priority audio threads and UI threads, ensuring ultra-low latency (~1.3ms) and glitch-free real-time performance.

---

## 1. System-Level Coordination Agents

These are the core manager agents that oversee the lifecycle, data flow, and hardware interaction of the Amplitron ecosystem.

### 1.1 The Audio Engine Agent (`audio_engine.cpp`)
The master coordinator of the system. Operating at the highest system priority via PortAudio, this agent is responsible for the critical real-time audio callback loop.
* **Role:** Fetches raw mono float32 samples from the hardware input (USB interface/guitar cable), routes them sequentially through the active DSP agents, and pushes the processed frames to the output hardware.
* **Responsibilities:** * Auto-detecting input/output hardware.
  * Buffer size negotiation (32 to 512 samples) and sample rate enforcement (44.1kHz - 96kHz).
  * Enforcing safety clamps (hard limiting to ±1.0) to prevent hardware or auditory damage.
  * Handling lock-free or `try_lock` mutex polling to ensure the GUI thread never blocks the audio thread.

### 1.2 The GUI Manager Agent (`gui_manager.cpp`)
The user-facing orchestration agent built on SDL2 and Dear ImGui.
* **Role:** Translates human interaction into system state changes without interrupting the DSP pipeline.
* **Responsibilities:** Rendering the application window, parsing hardware input (mouse/keyboard), painting the pedal board visually, and updating the global state tree that the Audio Engine Agent consumes.

### 1.3 The Pedal Board Agent (`pedal_board.cpp`)
The state-manager agent for the signal chain.
* **Role:** Acts as a dynamic registry that maintains the ordered list of active DSP Pedal Agents.
* **Responsibilities:** Handling the insertion, deletion, reordering, and bypass toggling of effects. It safely mutates the chain state while communicating with the Audio Engine.

### 1.4 The Preset Manager Agent (`preset_manager.cpp`)
The persistence agent for signal chain configurations.
* **Role:** Serializes and deserializes the complete pedal chain state (effect types, parameter values, enabled states, ordering) to and from JSON files.
* **Responsibilities:** Saving presets, loading presets, creating effect instances by name during deserialization, and managing the presets directory. Integrates with the file dialog system for save/load UI.

### 1.5 The Command History Agent (`command_history.cpp`)
The undo/redo state-tracking agent.
* **Role:** Maintains a stack-based history of all user actions (parameter changes, pedal additions/removals, reordering) to enable full undo/redo functionality.
* **Responsibilities:** Recording commands, executing undo/redo, managing the history stack with proper cleanup when new actions branch off from a past state.

### 1.6 The Snapshot Manager Agent (`snapshot_manager.h`, `gui_snapshots.h/.cpp`)
The in-session A/B/C/D board-state switching agent.
* **Role:** Stores up to 4 complete board configurations in memory for instant, glitch-free recall during a live performance session — without any file I/O.
* **Responsibilities:** Capturing the full effect chain state (effect instances, enabled/mix flags, all parameter values, input/output gains) into numbered slots; restoring a slot via `RecallSnapshotCommand` (undoable via Ctrl+Z); and rendering the [A][B][C][D] toolbar row with visual indication of the active slot. Left-click recalls a filled slot; right-click opens a context menu to save or clear any slot; Ctrl/Cmd+1–4 recalls.

### 1.7 The Spectrum Analyzer Agent (`spectrum_analyzer.cpp`)
The frequency-domain visualization agent.
* **Role:** Performs real-time frequency analysis of the audio signal and renders a visual spectrum display in the GUI.
* **Responsibilities:** Accepting audio data from the lock-free SPSC queue, computing frequency bins, and rendering the spectrum graph.

### 1.8 The Recorder Agent (`recorder.cpp`)
The WAV recording agent.
* **Role:** Captures processed audio output and writes it to WAV files on disk.
* **Responsibilities:** Managing recording state (start/stop), writing WAV headers, buffering audio data, and flushing to disk.

### 1.9 The MIDI Manager Agent (`midi_manager.h/.cpp`, `gui_midi.h/.cpp`)
The MIDI CC mapping and learn agent.
* **Role:** Receives MIDI Control Change messages from hardware controllers via RtMidi (callback thread) and routes them to effect parameters, bypass toggles, or master gains via the GUI/main thread.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sudip-mondal-2002/Amplitron](https://github.com/sudip-mondal-2002/Amplitron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
