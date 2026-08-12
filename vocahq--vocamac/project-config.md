---
trigger: always_on
description: VocaMac is a **native macOS menu bar application** for local voice-to-text dictation, built with **Swift 5.9+** and **SwiftUI**. It supports four on-device speech engines: **WhisperKit** (OpenAI Whisper, CoreML), **FluidAudio** (NVIDIA Parakeet TDT, CoreML on the Neural Engine), **Apple Speech** (SpeechAnalyzer, macOS 26+), and **sherpa-onnx** (specialized ONNX models: Moonshine, SenseVoice, GigaAM, Canary; CPU-only). `TranscriptionRouter` dispatches to the engine that owns the selected model. T
---

# VocaMac — AI Coding Agent Guidelines

## Project Overview

VocaMac is a **native macOS menu bar application** for local voice-to-text dictation, built with **Swift 5.9+** and **SwiftUI**. It supports four on-device speech engines: **WhisperKit** (OpenAI Whisper, CoreML), **FluidAudio** (NVIDIA Parakeet TDT, CoreML on the Neural Engine), **Apple Speech** (SpeechAnalyzer, macOS 26+), and **sherpa-onnx** (specialized ONNX models: Moonshine, SenseVoice, GigaAM, Canary; CPU-only). `TranscriptionRouter` dispatches to the engine that owns the selected model. The project also includes a **static marketing website** (`web/`) deployed to GitHub Pages at [vocamac.com](https://vocamac.com).

- **License:** AGPL-3.0
- **Minimum target:** macOS 14 (Sonoma)
- **Build system:** Swift Package Manager
- **CI:** GitHub Actions (`.github/workflows/ci.yml`)
- **Website deployment:** GitHub Pages via release trigger (`.github/workflows/deploy-website.yml`)

---

## Critical Rule: Use Git Worktrees for Parallel Tasks

**When asked to perform multiple unrelated tasks simultaneously, ALWAYS use git worktrees.**

```bash
# Create worktrees in /tmp — never pollute the main workspace
git worktree add /tmp/vocamac-<task-name> -b <branch-name> main

# After work is complete, clean up
git worktree remove /tmp/vocamac-<task-name>
git worktree prune
```

**Why:** Concurrent work on the same directory causes branch conflicts, overwritten files, and corrupted state. Each worktree gets its own isolated copy of the repo on its own branch.

**Rules:**
- Create worktrees in `/tmp/` with the prefix `vocamac-`
- One worktree per branch, one branch per PR
- Always prune worktrees after pushing and creating PRs
- Never modify files in the main workspace when worktrees are active for that task

---

## Repository Structure

```
VocaMac/
├── Sources/VocaMac/
│   ├── App/              # App entry point, MenuBarExtra, MenuBarIcon
│   ├── Models/           # AppState, TranscriptionResult, WhisperModel
│   ├── Services/         # AudioEngine, HotKeyManager, ModelManager,
│   │                     #   SystemInfo, TextInjector, TranscriptionRouter,
│   │                     #   WhisperService, ParakeetService, AppleSpeechService,
│   │                     #   SherpaService (+ Vendor/SherpaOnnxConfigBuilders)
│   ├── Views/            # MenuBarView, SettingsView
│   └── Resources/        # Bundled resources (.gitkeep placeholder)
├── Tests/VocaMacTests/   # Unit tests
├── web/                  # Static website (HTML/CSS/JS, deployed to GitHub Pages)
├── Makefile              # make build, install, test, clean
├── scripts/              # build.sh, install.sh, uninstall.sh
├── docs/                 # ARCHITECTURE.md, DATA_MODEL.md, PRD.md
├── Package.swift         # SPM manifest
└── VocaMac.entitlements  # App sandbox entitlements
```

---

## Build & Run

```bash
# Build + install to /Applications (recommended)
make install

# Build .app bundle in repo root (fast dev iteration)
make build

# Install CLI commands to ~/.local/bin
make install-cli

# Run tests
make test

# Clean build artifacts
make clean
```

Or use the scripts directly:

```bash
./scripts/build.sh              # Build .app bundle (dev)
./scripts/install.sh            # Build + install to /Applications
./scripts/install.sh --cli      # Install CLI commands
```

The project builds on **macOS only** (requires AppKit, CoreML, AVFoundation). CI runs on `macos-15`.

---

## Code Style & Best Practices

### Swift Conventions
- Use **SwiftUI** for all views — no AppKit views unless absolutely necessary for system integration
- Use **`@Observable`** (or `ObservableObject` with `@Published`) for state management
- Prefer **`async/await`** over callbacks and closures for asynchronous work
- Use **`guard`** for early returns; avoid deep nesting
- Follow Apple's [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
- Use meaningful names: `isRecording` not `flag`, `audioLevel` not `val`
- Mark sections with `// MARK: -` for organization
- Add doc comments (`///`) on all public types, methods, and non-trivial private methods

### Architecture Patterns
- **Single source of truth:** `AppState` is the central observable state object
- **Service layer:** Business logic lives in `Services/` (AudioEngine, WhisperService, etc.)
- **Views are thin:** Views observe state and dispatch actions — no business logic in views
- **Dependency injection:** Pass dependencies via `@EnvironmentObject` or init parameters

### Error Handling
- Never force-unwrap (`!`) unless the value is guaranteed (e.g., system symbols)
- Use `do/catch` with meaningful error types
- Surface errors to the user via `AppState.appStatus = .error` with clear messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VocaHQ/vocamac](https://github.com/VocaHQ/vocamac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
