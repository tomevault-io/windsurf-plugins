---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ProcTap is a cross-platform Python library for capturing audio from specific processes. It provides platform-optimized backends for Windows, Linux, and macOS.

**Platform Support:**
- **Windows**: ✅ Fully implemented using WASAPI Process Loopback (C++ native extension)
- **Linux**: ✅ Fully implemented - PipeWire Native/PulseAudio (per-process isolation, v0.3.0+)
- **macOS**: ✅ **Officially supported** - ScreenCaptureKit (macOS 13+, bundleID-based)

**Key Characteristics:**
- Per-process audio isolation (not system-wide)
  - Windows/Linux: PID-based capture
  - macOS: bundleID-based capture (ScreenCaptureKit)
- Low-latency streaming (10-15ms on macOS, 10ms on Windows, 2-5ms on Linux with PipeWire Native)
- Platform-specific implementations:
  - Windows: WASAPI C++ extension (Windows 10 20H1+)
  - Linux: PipeWire Native API / PulseAudio (fully supported, v0.3.0+)
  - **macOS: ScreenCaptureKit Swift helper (macOS 13+) - RECOMMENDED**
- Dual API: callback-based and async iterator patterns

## Development Guidelines

### Test File Organization

**IMPORTANT:** Use `.claude_test/` directory for all temporary and experimental test files:

**Required Usage of `.claude_test/`:**
- All temporary test scripts (for quick testing/debugging)
- Experimental or throw-away code
- Test audio files and sample data
- Any files created for verification purposes
- Example/demo files used during development

**Do NOT use `.claude_test/` for:**
- Official test suite files (use `tests/` directory)
- Production examples (use `examples/` directory)
- Production code (use `src/` directory)

**Cleanup:**
- `.claude_test/` is gitignored
- Clean up files in `.claude_test/` after completing tests
- Only commit to `tests/`, `examples/`, or `src/` when ready for production

### Testing Standards

**IMPORTANT:** When creating official test code, ALWAYS follow pytest conventions:
- Use pytest framework for all tests
- Place tests in `tests/` directory or name files with `test_*.py` pattern
- Use pytest fixtures, parametrize, and markers
- Follow pytest discovery conventions
- Use `.claude_test/` for experimental scripts before moving to `tests/`

### Setup and Building

```bash
# Install in editable mode with dev dependencies
pip install -e ".[dev]"

# Build wheel (requires Visual Studio Build Tools + Windows SDK)
python -m build --wheel

# Build source distribution
python -m build
```

**Important:** After modifying C++ code in [_native.cpp](src/proctap/_native.cpp), you must rebuild:
```bash
pip install -e . --force-reinstall --no-deps
```

### Testing and Type Checking

```bash
# Run tests
pytest

# Type check
mypy src/
```

### Running Examples

```bash
# Windows example
python examples/windows_basic.py --pid 12345 --output audio.wav
python examples/windows_basic.py --name "VRChat.exe" --output audio.wav

# Linux example (requires pulseaudio-utils)
python examples/linux_basic.py --pid 12345 --duration 5 --output output.wav

# macOS example (requires macOS 14.4+, PyObjC)
python examples/macos_basic.py --pid 12345 --duration 5 --output output.wav
```

### CLI Usage (Pipe to FFmpeg)

The package installs a `proctap` command for direct use:

```bash
# Direct command (recommended)
proctap --pid 12345 --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Or using python -m (alternative)
python -m proctap --pid 12345 --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Using process name instead of PID
proctap --name "VRChat.exe" --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Low-latency mode with fast resampling (for real-time streaming)
proctap --pid 12345 --resample-quality fast --stdout | ffmpeg -f s16le -ar 48000 -ac 2 -i pipe:0 output.mp3

# Available quality modes:
# --resample-quality best   (highest quality, ~1.3-1.4ms latency, default)
# --resample-quality medium (medium quality, ~0.7-0.9ms latency)
# --resample-quality fast   (lowest quality, ~0.3-0.5ms latency)
```

### macOS Setup

**Recommended: ScreenCaptureKit Backend (macOS 13+)**

```bash
# Build Swift helper binary
cd src/proctap/swift/screencapture-audio
swift build -c release

# Enable Screen Recording permission
# System Settings → Privacy & Security → Screen Recording → Enable for Terminal/IDE

# Test
python examples/macos_screencapture_test.py --bundle-id com.apple.Safari --duration 5
```

**Fallback: PyObjC Backend (Experimental, macOS 14.4+)**

```bash
# Install PyObjC dependencies
pip install pyobjc-core pyobjc-framework-CoreAudio

# Or install with optional dependencies
pip install -e ".[macos]"

# Note: PyObjC backend has IOProc callback issues and is not recommended
```

## Architecture

### Multi-Platform Backend Architecture

The library uses platform-specific backends selected at runtime:

```
ProcTap (core.py - Public API)
    ↓
backends/__init__.py (Platform Detection)
    ↓
┌─────────────────┬──────────────────┬──────────────────────────┐
│ Windows         │ Linux            │ macOS                    │
│ (Implemented)   │ (Implemented)    │ (Implemented)            │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m96-chan/ProcTap](https://github.com/m96-chan/ProcTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
