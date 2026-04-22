---
trigger: always_on
description: **AceForge** is a local-first AI music workstation for macOS Silicon that provides a comprehensive interface for AI-powered music generation, training, and audio processing. The project integrates multiple AI models including ACE-Step (text-to-music), Demucs (stem separation), XTTS v2 (voice cloning), and basic-pitch (MIDI generation).
---

# GitHub Copilot Custom Instructions for AceForge

## Project Overview

**AceForge** is a local-first AI music workstation for macOS Silicon that provides a comprehensive interface for AI-powered music generation, training, and audio processing. The project integrates multiple AI models including ACE-Step (text-to-music), Demucs (stem separation), XTTS v2 (voice cloning), and basic-pitch (MIDI generation).

### Key Characteristics
- **Platform**: macOS-first with Apple Silicon (M1/M2/M3/M4) optimization
- **Architecture**: Flask backend + React/TypeScript frontend
- **AI/ML Focus**: Heavy use of PyTorch with MPS (Metal Performance Shaders) acceleration
- **Deployment**: Both source-based and PyInstaller-bundled macOS app
- **Privacy**: 100% local processing - no cloud dependencies after model downloads

## Technology Stack

### Backend (Python)
- **Framework**: Flask with Blueprint-based architecture
- **ML/AI**: PyTorch (with MPS support), Transformers, ACE-Step, Demucs
- **Audio Processing**: pydub, audioread, soundfile, librosa
- **Dependencies**: See `requirements_ace_macos.txt`
- **Python Version**: 3.10+

### Frontend (TypeScript/React)
- **Framework**: React 19 with TypeScript
- **Build Tool**: Vite
- **UI Library**: Custom components with Lucide icons
- **State Management**: React hooks and context
- **Package Manager**: Bun (preferred) or npm

### Build & Distribution
- **Bundling**: PyInstaller for macOS app bundles
- **Code Signing**: Ad-hoc signing with optional Developer ID support
- **CI/CD**: GitHub Actions for automated builds and tests

## Code Style and Conventions

### Python Backend

#### File Organization
- Prefix module names with `cdmf_` for core functionality (e.g., `cdmf_generation.py`, `cdmf_training.py`)
- Use Blueprint pattern for Flask routes (e.g., `create_generation_blueprint()`)
- Keep business logic separate from route handlers

#### Type Hints
- Use `from __future__ import annotations` for forward references
- Prefer typed function signatures:
  ```python
  def create_blueprint(
      html_template: str,
      ui_defaults: Dict[str, Any],
      callback: Callable[..., Dict[str, Any]],
  ) -> Blueprint:
  ```

#### Error Handling
- Use try-except blocks with specific exception types
- Use Python's `logging` module for error logging and diagnostics
- For quick debugging, `print(..., flush=True)` is acceptable but prefer logging
- Return structured error responses in API endpoints:
  ```python
  import logging
  logger = logging.getLogger(__name__)
  
  try:
      # ... operation
  except Exception as e:
      logger.error(f"Operation failed: {e}", exc_info=True)
      return jsonify({"error": "Description", "details": str(e)}), 500
  ```

#### Documentation
- Use docstrings for functions explaining purpose, parameters, and return values
- Add inline comments for complex logic, especially ML/audio processing steps
- Document environment variables and configuration options

### TypeScript/React Frontend

#### Component Structure
- Use functional components with hooks
- Prefer named exports for components
- Co-locate types when component-specific, otherwise use `types.ts`

#### Type Safety
- Define interfaces for props and state
- Use discriminated unions for view states
- Avoid `any` type - use `unknown` when type is truly dynamic

#### State Management
- Use React Context for global state (auth, theme, responsive)
- Keep component state local when possible
- Use refs for imperative DOM access and intervals

#### API Calls
- Centralize API calls in `services/api.ts`
- Use async/await with proper error handling
- Show user-friendly error messages in UI

## Architecture Patterns

### Backend Architecture

#### Blueprint Pattern
Each major feature has its own Blueprint:
- `cdmf_generation.py` - Music generation
- `cdmf_training.py` - LoRA training
- `cdmf_stem_splitting.py` - Audio stem separation
- `cdmf_voice_cloning.py` - TTS voice cloning
- `cdmf_midi_generation.py` - Audio-to-MIDI transcription

#### State Management
- Global state in `cdmf_state.py` for model loading and training status
- Track library management in `cdmf_tracks.py`
- Path handling centralized in `cdmf_paths.py`

#### Model Loading
- Lazy loading: Models loaded on first use, not at startup
- Singleton pattern: One instance per model type
- MPS optimization: Automatic device selection (`mps`, `cuda`, `cpu`)

### Frontend Architecture

#### Component Hierarchy
```
App.tsx (main container)
├── Sidebar (navigation)
├── CreatePanel (music generation UI)
├── LibraryView (track browser)
├── TrainingPanel (LoRA training UI)
├── StemSplittingPanel (Demucs UI)
├── VoiceCloningPanel (XTTS UI)
├── MidiPanel (basic-pitch UI)
├── Player (audio playback)
└── RightSidebar (queue/info)
```

#### View Management
- Single `currentView` state determines which panel is visible
- Views: `'create'`, `'library'`, `'training'`, `'stem-splitting'`, `'voice-cloning'`, `'midi'`
- Responsive design with mobile/desktop layouts

## AI/ML Specific Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [audiohacking/AceForge](https://github.com/audiohacking/AceForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
