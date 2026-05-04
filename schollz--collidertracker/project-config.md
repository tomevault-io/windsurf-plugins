---
trigger: always_on
description: ColliderTracker is a terminal-based music tracker that integrates with SuperCollider for real-time audio synthesis and sampling. It's built in Go using the Bubble Tea TUI framework and communicates with SuperCollider via OSC (Open Sound Control).
---

# ColliderTracker Development Guide

## Project Overview

ColliderTracker is a terminal-based music tracker that integrates with SuperCollider for real-time audio synthesis and sampling. It's built in Go using the Bubble Tea TUI framework and communicates with SuperCollider via OSC (Open Sound Control).

**Repository Stats:**
- **Language:** Go 1.25
- **Size:** Medium-sized codebase (~50MB)
- **Code:** Multiple packages in `internal/` directory with comprehensive test coverage
- **Type:** Terminal UI application with audio engine integration
- **Platforms:** Linux, macOS, Windows (with platform-specific code)

## Build Requirements and Setup

### Required System Dependencies

**CRITICAL:** Always install these dependencies BEFORE attempting to build or test:

```bash
# Linux (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y libasound2-dev

# macOS
brew update
brew install pkg-config rtmidi

# Windows (MSYS2)
pacman -S --noconfirm mingw-w64-x86_64-rtmidi mingw-w64-x86_64-toolchain
```

**Why these are required:**
- `libasound2-dev` (Linux): Required for ALSA MIDI support via rtmidi
- `rtmidi`: MIDI I/O library (macOS/Windows)
- `pkg-config`: Build tool for finding library paths

### Environment Variables

**Always set these environment variables before building:**

```bash
export CGO_ENABLED=1
export CGO_CXXFLAGS="-D__RTMIDI_DEBUG__=0 -D__RTMIDI_QUIET__"
```

**Why these matter:**
- `CGO_ENABLED=1`: Required because the project uses C bindings for MIDI
- `CGO_CXXFLAGS`: Disables verbose RTMIDI debug output

### Additional Platform-Specific Setup

**Windows:**
```bash
export CC=x86_64-w64-mingw32-gcc
export CGO_LDFLAGS=-static
```

**Linux (static build):**
Requires building ALSA library from source. See `.github/workflows/build.yml` alpine job for the complete process.

## Build Commands

### Standard Build Process

**Order matters! Always follow this sequence:**

1. **Install dependencies first** (see above)
2. **Set environment variables** (see above)
3. **Build:**
   ```bash
   go build -v -o collidertracker
   ```
4. **Verify:**
   ```bash
   ./collidertracker --help
   ```

**Expected build time:** 10-30 seconds on modern hardware

**Common build failure:** If you get `fatal error: alsa/asoundlib.h: No such file or directory`, you forgot to install `libasound2-dev` on Linux.

### Testing

**Always run tests with the environment variables set:**

```bash
export CGO_CXXFLAGS="-D__RTMIDI_DEBUG__=0 -D__RTMIDI_QUIET__"
go test -v ./...
```

**Test execution time:** ~1-5 seconds

**Test files:** 22 test files (`*_test.go`) across the `internal/` directory

**Important:** Tests include audio file processing tests with WAV files in `internal/getbpm/`. Don't delete these.

## Project Architecture

### Directory Structure

```
collidertracker/
├── main.go                          # Application entry point with cobra CLI
├── main_test.go                     # Main application tests
├── go.mod, go.sum                   # Go module files
├── build.sh                         # Docker-based static build script
├── ecosystem.config.js              # PM2 config for jackd and supercollider
├── .github/
│   ├── workflows/
│   │   ├── build.yml               # Main CI/CD: builds for macOS, Linux, Windows, Alpine
│   │   ├── auto-update.yml         # Weekly dependency updates
│   │   └── homebrew.yml            # Updates Homebrew tap on releases
│   └── copilot-instructions.md     # This file
└── internal/
    ├── audio/                       # Audio length calculation
    ├── getbpm/                      # BPM detection (includes test WAV files)
    ├── hacks/                       # Platform-specific workarounds
    ├── input/                       # Keyboard input handling and user interaction
    ├── midiconnector/               # MIDI device connection (platform-specific)
    ├── midiplayer/                  # MIDI playback engine
    ├── model/                       # Core data model (large, central state management)
    ├── modulation/                  # Note modulation engine
    ├── music/                       # Music theory utilities
    ├── project/                     # Project selection UI
    ├── storage/                     # Save/load functionality
    ├── supercollider/               # SuperCollider integration
    │   ├── collidertracker.scd     # SuperCollider server code
    │   ├── DX7.scd, DX7.afx        # DX7 synthesizer
    │   └── dx7.json                # DX7 patch library
    ├── ticks/                       # Timing/tempo utilities
    ├── types/                       # Core type definitions (data structures)
    └── views/                       # TUI views (song, chain, phrase, etc.)
```

### Key Files

**Main entry point:** `main.go` - Sets up Cobra CLI, initializes SuperCollider, starts Bubble Tea app

**Core model:** `internal/model/model.go` - Central state management and application logic

**Type definitions:** `internal/types/types.go` - Data structures for songs, chains, phrases, etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schollz/collidertracker](https://github.com/schollz/collidertracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
