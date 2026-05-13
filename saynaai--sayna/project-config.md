---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sayna is a real-time voice processing server built in Rust that provides unified Speech-to-Text (STT) and Text-to-Speech (TTS) services through WebSocket and REST APIs. It integrates with LiveKit for real-time audio streaming and includes advanced noise filtering using DeepFilterNet.

## Development Commands

```bash
cargo run                    # Run development server
cargo run -- -c config.yaml  # Run with YAML config file
cargo test                   # Run all tests
cargo test test_name         # Run specific test
cargo build --release        # Build for release
cargo check                  # Check without building
cargo fmt                    # Format code
cargo clippy                 # Run linter
docker build -t saynaai/sayna .      # Build Docker image
```

### Feature Flags
By default, no optional features are enabled.

- `stt-vad` (disabled by default): Silero-VAD voice activity detection with integrated ONNX-based turn detection. When enabled, VAD monitors audio for silence and triggers the turn detection model to confirm if the speaker's turn is complete.
- `noise-filter` (disabled by default): DeepFilterNet noise suppression. Disable to reduce dependencies.
- `openapi` (disabled by default): OpenAPI 3.1 specification generation using utoipa crate.

```bash
cargo check                                        # Default build, no optional features
cargo check --no-default-features                  # Explicitly disable optional features
cargo build --features stt-vad                     # Enable VAD with turn detection
cargo build --features stt-vad,openapi             # Enable specific features
cargo run --features openapi -- openapi -o docs/openapi.yaml  # Generate OpenAPI spec
```

## High-Level Architecture

### Development Rules

The codebase includes detailed development rules in `.cursor/rules/`:
- **`rust.mdc`**: Rust best practices, design patterns, performance, security, testing
- **`core.mdc`**: STT/TTS provider abstractions (`BaseSTT`, `TTSProvider` traits)
- **`axum.mdc`**: Axum framework patterns for WebSocket and REST APIs
- **`livekit.mdc`**: LiveKit integration patterns and WebSocket API details
- **`openapi.mdc`**: OpenAPI 3.1 documentation guidelines using utoipa

Always consult these rule files when implementing new features.

### Core Components

1. **VoiceManager** (`src/core/voice_manager/`): Central coordinator for STT/TTS
   - Manages provider lifecycle and switching
   - Thread-safe with `Arc<RwLock<>>` for concurrent access
   - Handles callbacks for STT results and audio output

2. **Provider System** (`src/core/stt/` and `src/core/tts/`):
   - Trait-based abstraction for pluggable providers
   - **STT**: Deepgram, Google (gRPC), ElevenLabs, Microsoft Azure, Cartesia
   - **TTS**: Deepgram, ElevenLabs, Google, Microsoft Azure, Cartesia

3. **WebSocket Handler** (`src/handlers/ws/`):
   - Real-time bidirectional communication at `/ws`
   - Processes audio streams, config updates, control messages

4. **LiveKit Integration** (`src/livekit/`):
   - WebRTC audio streaming with room/participant management
   - Audio track subscription and data message forwarding

5. **DeepFilterNet** (`src/utils/noise_filter.rs`):
   - Advanced noise reduction with thread pool for CPU-intensive operations

6. **Authentication** (`src/auth/` and `src/middleware/auth.rs`):
   - Optional JWT-based auth with external validation service

7. **VAD + Turn Detection** (`src/core/vad/`, `src/core/turn_detect/`) - Feature-gated: `stt-vad`:
   - **SileroVAD**: ONNX-based voice activity detection model
   - **SilenceTracker**: Tracks continuous silence duration from VAD
   - **Turn Detection**: ONNX-based model that confirms turn completion when silence is detected
   - Integrates with VoiceManager for audio processing
   - VAD and turn detection are always bundled together under `stt-vad` feature

### Voice Activity Detection (VAD) with Turn Detection

When `stt-vad` feature is enabled, Sayna provides both Silero-VAD for audio-level silence detection and smart-turn v3 model for semantic turn detection. Both features are always bundled together under the `stt-vad` feature flag.

**Why Use Both VAD and Smart-Turn?**

VAD (Silero) provides:
- Fast, efficient silence detection (~2ms per frame)
- Low latency feedback on speech activity
- Immediate signal when user stops speaking

Smart-Turn provides:
- Semantic understanding of turn completion
- Higher accuracy (>90% for most languages)
- Context-aware decision making

**Combined Pattern**: VAD detects silence first (fast, cheap), then smart-turn confirms if the turn is semantically complete (more accurate, slightly slower). This two-stage approach provides both speed and accuracy.

**How it works:**
```
Audio In -> VAD -> SilenceTracker -> TurnEnd Event -> Smart-Turn -> speech_final
                         ^                               |
                         |                               |
                         +-- If false, reset and wait ---+
```

1. Audio frames are processed through Silero-VAD ONNX model

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaynaAI/sayna](https://github.com/SaynaAI/sayna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
