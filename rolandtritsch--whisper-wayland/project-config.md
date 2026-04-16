---
trigger: always_on
description: This document explains the architecture, implementation decisions, and development workflow for the Whisper Wayland voice-to-text service.
---

# Whisper Wayland - Developer Documentation

This document explains the architecture, implementation decisions, and development workflow for the Whisper Wayland voice-to-text service.

> **For users**: See [README.md][readme-md] for installation instructions, usage guide, and user documentation.

## Architecture Overview

### Design Philosophy

Whisper Wayland follows a **modular architecture** with clear separation of concerns:

- **Application Layer**: Main orchestrator that coordinates all components
- **Component Management**: Manages lifecycle of audio, transcription, key monitoring, and text insertion
- **Service Components**: Independent modules for core functionality
- **Configuration Management**: Environment-based configuration with validation

### Core Components

#### 1. Application (`whisper_wayland/application/`)
- **`application.py`**: Main orchestrator coordinating all components
- **`component_manager.py`**: Creates and manages component instances
- **`runtime.py`**: Handles main application loop and signal management
- **`hotkey_handler.py`**: Manages push-to-talk functionality
- **`transcription_processor.py`**: Processes audio data through transcription pipeline

#### 2. Audio Recording (`whisper_wayland/audio_recorder/`)
- **`audio_recorder.py`**: High-level audio recording interface
- **`recording_engine.py`**: Low-level PyAudio recording implementation
- **`audio_system_validator.py`**: Validates audio system availability
- **`wav_converter.py`**: Converts audio frames to WAV format

#### 3. Key Monitoring (`whisper_wayland/key_monitor/`)
- **`key_monitor.py`**: High-level keyboard monitoring interface
- **`monitor_loop.py`**: Main monitoring loop with device handling
- **`event_handler.py`**: Processes key events and triggers callbacks
- **`device_manager.py`**: Manages input device discovery and lifecycle
- **`key_mapping.py`**: Handles key combination parsing and mapping

#### 4. Text Insertion (`whisper_wayland/text_inserter/`)
- **`text_inserter.py`**: High-level text insertion interface
- **`method_executors.py`**: Implements different insertion methods (wtype, ydotool, xdotool, clipboard)
- **`capability_tester.py`**: Tests which insertion methods are available
- **`fallback_handler.py`**: Manages fallback between insertion methods
- **`text_processor.py`**: Cleans and processes text for insertion

#### 5. Transcription Client (`whisper_wayland/transcription_client/`)
- **`transcription_client.py`**: High-level OpenAI API interface
- **`transcription_engine.py`**: Handles API communication and retry logic
- **`model_mapper.py`**: Maps model names to API parameters
- **`connection_tester.py`**: Tests API connectivity
- **`test_audio_generator.py`**: Generates test audio for validation

#### 6. Configuration (`whisper_wayland/config/`)
- **`config.py`**: Main configuration orchestrator
- **`property_handlers.py`**: Environment variable processing and validation
- **`config_validator.py`**: Configuration validation and requirements checking
- **`env_loader.py`**: Environment file loading (.env support)
- **`logging_setup.py`**: Logging configuration and setup

### Key Design Decisions

#### Why Modular Architecture?
- **Testability**: Each component can be tested in isolation
- **Maintainability**: Clear boundaries make changes safer and easier
- **Extensibility**: New components or alternatives can be added easily
- **Debugging**: Issues can be traced to specific components

#### Why Configuration-First Design?
- **Environment Variables**: Follows 12-factor app principles
- **No Hard-coding**: All behavior can be controlled externally
- **Easy Deployment**: Configuration changes don't require code changes
- **Testing**: Different configurations can be tested independently

#### Why Push-to-Talk vs. Voice Activation?
- **Privacy**: Audio only sent when user explicitly requests it
- **Accuracy**: No false triggers from background noise
- **Control**: User has complete control over when transcription occurs
- **Cost**: Only pay for intentional transcriptions

## Development Workflow

### Repository Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/rolandtritsch/whisper-wayland.git
   cd whisper-wayland
   ```

2. **Install dependencies:**
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh  # Install uv
   uv sync  # Install project dependencies
   ```

3. **Configure environment:**
   ```bash
   cp .env.example .env
   # Edit .env and add your OPENAI_API_KEY
   ```

### Branch Management

**Branch naming convention:**
- Format: `roland/<ticket-id>/<3-word-description>`
- If no ticket exists: `roland/ad-hoc/<3-word-description>`
- Examples:
  - `roland/ISSUE-123/fix-audio-recording`
  - `roland/ad-hoc/update-dependencies`

**Creating a branch:**
```bash
git checkout trunk
git pull origin trunk
git checkout -b roland/<ticket-id>/<3-word-description>
```

### Pull Request Process

**PR title format:**
- `<ticket-id>: <3-word-description>`
- Examples:
  - `ISSUE-123: Fix audio recording`
  - `ad-hoc: Update dependencies`

**Creating a PR:**
```bash
# Push your branch
git push -u origin roland/<ticket-id>/<3-word-description>

# Create PR with gh CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rolandtritsch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
