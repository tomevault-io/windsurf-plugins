---
trigger: always_on
description: Voice-to-text system using RealtimeSTT with Whisper backend, supporting both wake word ("Jarvis") and manual double-command activation.
---

# 🎤 RealtimeSTT Whisper Voice Recognition System

## Project Overview
Voice-to-text system using RealtimeSTT with Whisper backend, supporting both wake word ("Jarvis") and manual double-command activation.

## Project Structure
```
whisper/
├── src/
│   ├── backends/
│   │   ├── realtimestt_backend.py    # RealtimeSTT wrapper (core recording logic)
│   │   └── transcription_base.py     # Base transcription interface
│   ├── services/
│   │   ├── keyboard_service.py       # Double command & manual recording
│   │   └── wake_word_service.py      # Wake word detection & processing
│   ├── utils/
│   │   ├── clipboard.py              # Clipboard copy/paste operations
│   │   ├── recording_events.py       # Event system for inter-service communication
│   │   ├── accessibility.py          # System accessibility helpers
│   │   └── process.py                # Process/thread management
│   └── core/
│       └── transcription_state.py    # Transcription state tracking
├── bin/
│   └── run.sh                        # Launch script
└── Jarvis_en_mac_v3_0_0.ppn         # Wake word model file
```

## Architecture Summary

### Two Independent Workflows

#### 1. Double Command Workflow (Manual Trigger)
**Flow**: Double Right Cmd → Recording starts → Right Cmd → Stop & Paste
- **Entry Point**: `keyboard_service.py` → `DoubleCommandKeyListener`
- **Recording**: `RealtimeSTTCommunicator` (separate instance)
- **No wake words**: Direct recording without "Jarvis" detection
- **Threading**: Background thread for recording

#### 2. Wake Word Workflow
**Flow**: Say "Jarvis" → Recording starts → Right Cmd → Stop & Paste
- **Entry Point**: `wake_word_service.py` → `WakeWordService`
- **Recording**: `RealtimeSTTWrapper` with wake word detection
- **Wake word**: "Jarvis" triggers recording
- **Manual stop**: Via event system from keyboard service

### Core Components

#### RealtimeSTT Backend (`src/backends/realtimestt_backend.py`)
- **Class**: `RealtimeSTTWrapper`
- **Purpose**: Wraps RealtimeSTT recorder with configuration
- **Key Methods**:
  - `transcribe()`: Main recording method (blocks until complete)
  - `abort_and_transcribe()`: DEPRECATED - use stop() + text()
  - `_initialize_recorder()`: Sets up recorder with callbacks
- **Configuration**: Handles VAD, wake words, silence detection

#### Keyboard Service (`src/services/keyboard_service.py`)
- **Classes**:
  - `RealtimeSTTCommunicator`: Manual recording handler
  - `DoubleCommandKeyListener`: Key press detection
- **Key Logic**:
  - Tracks `last_press_time` for double-click detection
  - Manages `is_transcribing` flag for state tracking
  - Emits events for wake word integration

#### Wake Word Service (`src/services/wake_word_service.py`)
- **Class**: `WakeWordService`
- **Key Methods**:
  - `transcribe()`: Single recording session
  - `_on_manual_stop_requested()`: Handles Right Cmd during recording
  - `_process_final_text()`: Final text processing/pasting
- **Event Integration**: Subscribes to `MANUAL_STOP_REQUESTED` event

#### Utility Components
- **ClipboardManager** (`src/utils/clipboard.py`): macOS clipboard operations
- **RecordingEventManager** (`src/utils/recording_events.py`): Event bus for inter-service communication
- **RecordingEvent** enum: Event types (MANUAL_STOP_REQUESTED, etc.)

#### Recording Popup GUI (`src/gui/recording_popup.py`)
- **RecordingPopup**: Always-on-top popup window with real-time waveform visualization
- **RecordingPopupManager**: Manages popup lifecycle and audio monitoring integration
- **Features**:
  - Dynamic waveform that reacts to voice input (bars change with audio levels)
  - Stop/Cancel buttons with callback support
  - Real-time audio level monitoring via AudioLevelMonitor
  - Auto-positioning in center of screen
  - Integrated with double command workflow

#### Audio Monitoring (`src/utils/audio_monitor.py`)
- **AudioLevelMonitor**: Real-time microphone level detection using PyAudio
- **MockAudioMonitor**: Fallback with simulated audio levels for development
- **Features**:
  - RMS audio level calculation with smoothing
  - 50 updates per second for smooth visualization
  - Automatic fallback when PyAudio unavailable

## Current Issues & Root Causes

### 🐛 Issue 1: Double Command Not Working
**Location**: `src/services/keyboard_service.py:172` and line 182

**Current Code (BROKEN after our fix):**
```python
# Line 172 - We changed this to -999
self.last_press_time = -999

# Line 182 - But this logic is incompatible!
time_diff = current_time - self.last_press_time if self.last_press_time > 0 else 999
```

**Problem**:
- `last_press_time = -999` (negative)
- Condition `self.last_press_time > 0` is FALSE
- So `time_diff = 999` (the else value)
- Double-click check `0 < time_diff < 2.0` FAILS (999 > 2.0)

**Solution**: Fix the condition to handle negative initialization:
```python
# Option 1: Change condition
time_diff = current_time - self.last_press_time if self.last_press_time >= 0 else 999

# Option 2: Better logic
if self.last_press_time < 0:  # First press
    time_diff = 999
else:
    time_diff = current_time - self.last_press_time
```

### 🐛 Issue 2: Double Pasting in Wake Word
**Locations**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jainnam-1993/whisper](https://github.com/jainnam-1993/whisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
