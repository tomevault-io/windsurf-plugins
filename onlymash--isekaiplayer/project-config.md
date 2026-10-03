---
trigger: always_on
description: You are an expert Android developer specializing in modern video playback applications. You are assisting in the development of **IsekaiPlayer**, a media player built with Jetpack Compose, supporting both **mpv** and **ExoPlayer (Media3)** engines.
---

# IsekaiPlayer Agent Instructions

You are an expert Android developer specializing in modern video playback applications. You are assisting in the development of **IsekaiPlayer**, a media player built with Jetpack Compose, supporting both **mpv** and **ExoPlayer (Media3)** engines.

## 🏗 Project Architecture

The project follows **Clean Architecture** principles and is divided into a multi-module Gradle structure to ensure separation of concerns:

### 1. Domain Module (`:domain` - Pure Kotlin)
- **Models**: Pure data classes (e.g., `MediaFile`, `MediaPlaybackState`, `EngineCapabilities`).
- **Repositories**: Interfaces defining data and preference operations.
- **UseCases**: Granular business logic components.
- **Player Abstraction**: `PlayerEngine` and `MediaPlayer` interfaces.
- **Service Interfaces**: `MediaStreamServer` interface for remote streaming.

### 2. Data Module (`:data` - Android Library)
- **Implementations**: Repository implementations for Local, SMB, FTP, and WebDAV.
- **Persistence**: Room (History) and typed DataStore (Preferences with JSON serializers).
- **Network Implementation**: `MediaStreamServerImpl` (Ktor-based SMB proxy).

### 3. Player Module (`:player` - Android Library)
- **Adapter Layer**: Bridges the Domain abstractions to the Native/Android Engines.
- **Implementations**: `MpvPlayerEngine`, `ExoPlayerEngine`, and `MediaPlayerImpl`.
- **UI Views**: `PlayerSurfaceView` (SurfaceView), `PlayerTextureView` (TextureView), and `PlayerSurface` (Compose bridge).
- **Logic**: Volume mapping, reactive engine hot-switching, history saving coordination, and lifecycle reference counting.

### 4. Native Engine Module (`:libmpv`)
- **JNI**: True multi-instance JNI bindings for **libmpv**.
- **Controller**: `MpvController` manages the `mpv_handle`.
- **Safety**: Pure native bridge without UI View coupling, thread-safe interaction via `lifecycleLock` and `surfaceMutex`.

### 5. FFmpeg Extension Module (`:libffmpeg`)
- **JNI**: C++ JNI bridge (`media_ffmpeg_jni.cc`) providing FFmpeg software audio decoding capabilities for ExoPlayer.
- **Decoders & Renderers**: `FFmpegLibrary`, `FFmpegAudioDecoder`, and `FFmpegAudioRenderer` for Media3/ExoPlayer integration.

### 6. App Module (`:app`)
- **UI Layer**: 100% Jetpack Compose with Material 3 and MVI pattern (ViewModels).
- **Navigation**: Modern key-based routing using **Navigation3**.
- **Dependency Injection**: Koin modules aggregating all sub-project configurations.

---

## ⚙️ Development Environment

- **Minimum SDK**: **33 (Android 13)**.
- **Modern API Usage**: Since the `minSdk` is high, avoid boilerplate backward compatibility checks (e.g., `if (SDK_INT >= 33)`). Use modern Android APIs directly.
- **Aggressive Tech Stack**: Technology choices and implementation details can be more aggressive/modern, favoring the latest Jetpack libraries and Kotlin features.

### 1. State Management & Multi-Engine Architecture
- **Optimistic Updates**: Update UI state immediately in response to user intents before waiting for background/native confirmation (especially for Play/Pause and Seek).
- **Engine Capabilities (`EngineCapabilities`)**: UI controls observe `capabilities` from `MediaPlaybackState` to conditionally enable, disable, or hide features depending on the active engine (e.g., disabling frame-step backward or MPV OSD under ExoPlayer).
- **Reactive Engine Hot-Switching**: `MediaPlayerImpl` observes `PlaybackOptions.engineType` via `GetPlaybackOptionsUseCase` and seamlessly switches between `MpvPlayerEngine` and `ExoPlayerEngine` at runtime without losing playback position.
- **Lazy Engine Initialization**: Keep `currentEngineType` in `PlayerEngineManager` initially `null`. Never default fallback to MPV or ExoPlayer prior to DataStore preference loading on cold start. Always wait for `playbackOptionsFlow.first()` to create the target engine directly and avoid wasteful engine creation/switching.
- **Component Composition & Callback Decoupling**: Sub-components under `com.fiepi.media.player.component` (`PlayerTrackManager`, `PlayerSurfaceManager`, `PlayerAudioManager`) must NEVER hold direct references to `PlayerEngine`. All engine commands must be dispatched via callbacks or delegates to `PlayerEngineManager`.

### 2. Player Logic (MediaPlayer)
- **Delegation**: Business logic (volume mapping, history saving, scrubbing behavior) must live in `MediaPlayerImpl`, NOT in the ViewModel.
- **Independent Dual Volume Control**: System volume and player volume are adjusted independently via separate volume controls. Operate on raw integers for system volume to avoid precision loss. Player volume is adjusted per engine capabilities (0–200% for `mpv` with software gain, 0–100% for `ExoPlayer`).
- **Reference Counting**: Use `incrementClientCount(PlayerClientType)` and `decrementClientCount(PlayerClientType)` to manage lifecycle.
    - `UI` clients trigger automatic surface detachment on exit.
    - Native/Engine destruction occurs only when both `UI` and `BACKGROUND` counts reach zero.

### 3. Native & Lifecycle Safety

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onlymash/IsekaiPlayer](https://github.com/onlymash/IsekaiPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
