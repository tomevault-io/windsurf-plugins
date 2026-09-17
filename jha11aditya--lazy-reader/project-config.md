---
trigger: always_on
description: This file serves as the permanent context, architectural blueprint, and task checklist for the **Lazy Reader** Android app. Any development session started in this directory reads this file to ensure absolute continuity.
---

# Lazy Reader - Project Mandates & Memory

This file serves as the permanent context, architectural blueprint, and task checklist for the **Lazy Reader** Android app. Any development session started in this directory reads this file to ensure absolute continuity.

---

## 1. Project Overview

**Lazy Reader** is a 100% offline, privacy-first Android document reader (PDF & EPUB) designed for hands-free reading. It uses local, on-device audio classification to listen for voice commands to control the reader, preventing the screen from dimming or locking while in reading mode.

### v1 Scope
To ship a first version with the least amount of unproven, bespoke work, v1 is intentionally narrowed:
- **PDF only.** EPUB support (custom parser + WebView styling) is deferred to v2 — it's the most bespoke, time-consuming piece of the original design and isn't needed to prove out the core hands-free reading experience.
- **3-class voice model only** (`"go"`, `"backward"`, `"stop"`). `"backward"` (not `"back"`) was chosen specifically so the model can be trained entirely on Google's public Speech Commands v0.02 dataset (CC-BY-4.0), which has no clips for the word "back" but does have "backward" — avoiding a bespoke, single-speaker recording effort in favor of a dataset with thousands of speakers. `"bookmark"`, `"night"/"day"`, and the sensitivity slider (see Section 2C) stay deferred until the core loop is proven.

### Core Features
- **Recents History Dashboard**: Displays a visual list of recently read books with progress percentages.
- **Robust Local Reader**:
  - Native `PdfRenderer` for smooth offline PDF reading.
  - *(v2)* Custom lightweight EPUB parser for customizable text size and colors.
- **Offline Voice Gestures**:
  - `"go"`: Navigate to the next page.
  - `"backward"`: Navigate to the previous page.
  - `"stop"`: Stop voice listening completely and enter a locked overlay reading mode.
- **Ambient Screen Lock Prevention**: Keeps the screen awake during active reading.

---

## 2. Technical Architecture & Expert Upgrades

Based on architectural research, we have upgraded the implementation strategy for maximum reliability, speed, and clean code:

### A. Speech AI: MediaPipe Audio Classifier (Better than Raw TFLite)
* **The Problem with Raw TFLite**: Processing raw microphone bytes to feed a raw TFLite model requires writing complex digital signal processing (DSP) logic in Kotlin: recording floats, computing Hamming/Hanning windows, executing Fast Fourier Transforms (FFT), building Mel-frequency filterbanks, and managing sliding overlapping circular buffers.
* **The MediaPipe Solution**: We will use **Google MediaPipe Audio Classifier Tasks (`com.google.mediapipe:tasks-audio`)**.
  - MediaPipe runs on top of TFLite but **completely abstracts audio feature extraction**.
  - It takes raw audio samples directly from the microphone and handles all windowing, spectrogram conversions, and model sliding window overlaps internally.
  - This reduces our AI integration footprint from ~800 lines of complex DSP math to less than 50 lines of clean Kotlin code.
  - It supports standard TFLite audio classification models (such as the YAMNet model or fine-tuned Keyword Spotting models).

### B. Screen Awake & Touch Lock
- **Keep-Awake**: Handled via Compose UI's `keepScreenOn` property on the container view:
  ```kotlin
  val currentView = LocalView.current
  DisposableEffect(Unit) {
      currentView.keepScreenOn = true
      onDispose { currentView.keepScreenOn = false }
  }
  ```
- **App Lock State**: When `"stop"` is heard:
  - Deactivate the microphone recorder.
  - Show a dark, beautiful overlay stating *"Voice Controls Stopped. Tap Lock icon to Unlock."* with a physical lock icon. This acts as a "pocket-lock" mode, protecting the screen from accidental inputs while lying down.
  - **Decision (2026-07-15): no full device lock.** Locking the phone programmatically requires the device-admin permission, whose scary system prompt and uninstall friction would undermine the app's privacy-first trust story (a device-admin implementation was built, then deliberately reverted). The overlay + the user pressing the power button covers the same need permission-free. If ever revisited, it should be an **opt-in setting** (v2), never the default.

### C. Recommended Additional Features (To be implemented)
1. **"bookmark" voice command**: Saves the current page instantly without touching the screen.
2. **"night" / "day" voice command**: Instantly toggles reading view themes (dark mode vs. light mode).
3. **Threshold Sensitivity Slider**: A UI slider in the settings to let users adjust the detection confidence (e.g., 70% to 90%) depending on their ambient room noise.

---

## 3. Directory Layout

```
lazy_reader/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── assets/             # Place model.tflite here
│   │   │   ├── java/com/lazyreader/
│   │   │   │   ├── data/           # Room Database, Recents entity
│   │   │   │   ├── voice/          # MediaPipe Audio Classifier & Mic loop
│   │   │   │   ├── ui/             # Jetpack Compose Screens, Theme, Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jha11aditya/lazy_reader](https://github.com/jha11aditya/lazy_reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
