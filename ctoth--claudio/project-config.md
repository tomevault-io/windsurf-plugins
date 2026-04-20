---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claudio is a hook-based audio plugin for Claude Code that plays contextual sounds based on tool usage and events. It uses a 5-level fallback system to map events to sounds and supports custom soundpacks.

## Build and Development Commands

```bash
# Build the main binary
go build -o claudio .

# Install to system PATH
sudo cp claudio /usr/local/bin/

# Run tests
go test ./...

# Run specific package tests with verbose output
go test ./internal/config -v
go test ./internal/cli -v

# Test audio playback manually
echo '{"session_id":"test","transcript_path":"/test","cwd":"/test","hook_event_name":"PostToolUse","tool_name":"Bash","tool_response":{"stdout":"success","stderr":"","interrupted":false}}' | claudio

# Run with different volumes
echo '...' | claudio --volume 0.7

# Use silent mode for testing without audio
echo '...' | claudio --silent

# Test file logging (with default config that enables file logging)
echo '...' | claudio --config /dev/null --silent
cat ~/.cache/claudio/logs/claudio.log

# Test with debug logging to see file creation
CLAUDIO_LOG_LEVEL=debug echo '...' | claudio --config /dev/null --silent
```

## Release Process

When releasing a new version of Claudio, follow this standardized process:

### Pre-Release Checklist
1. **Version Update**: Ensure `Version` constant in `internal/cli/cli.go` is updated
2. **Clean Workspace**: Ensure no uncommitted changes that shouldn't be released
3. **Remove Old Binaries**: Delete any existing binaries to ensure fresh build

### Release Steps
```bash
# 1. Clean build environment
rm -f claudio

# 2. Run full test suite
go test ./...

# 3. Build fresh binary
go build -o claudio .

# 4. Test binary works
./claudio --version

# 5. Functional smoke test
echo '{"session_id":"test","transcript_path":"/test","cwd":"/test","hook_event_name":"PostToolUse","tool_name":"Bash","tool_response":{"stdout":"success","stderr":"","interrupted":false}}' | ./claudio --silent

# 6. Create and push git tag
git tag v$(grep 'const Version' internal/cli/cli.go | cut -d'"' -f2)
git push origin v$(grep 'const Version' internal/cli/cli.go | cut -d'"' -f2)

# 7. Clean up build artifacts
rm -f claudio
```

### Post-Release
- Verify tag appears on GitHub
- Test `go install claudio.click/cmd/claudio@latest` works
- Update any documentation referencing version numbers

## Architecture Overview

### Core Components

1. **Hook System** (`internal/hooks/`)
   - Parses Claude Code hook JSON from stdin
   - Extracts context including tool names, success/error states, and file types
   - Maps events to sound categories: loading, success, error, interactive

2. **Sound Mapping** (`internal/sounds/`)
   - 5-level fallback system:
     1. Exact hint match: `category/hint.wav` (e.g., `success/bash-success.wav`)
     2. Tool-specific: `category/tool.wav` (e.g., `success/bash.wav`)
     3. Operation-specific: `category/operation.wav` (e.g., `success/tool-complete.wav`)
     4. Category-specific: `category/category.wav` (e.g., `success/success.wav`)
     5. Default: `default.wav`

3. **Audio System** (`internal/audio/`)
   - Uses malgo (miniaudio Go wrapper) for cross-platform audio
   - Memory-based playback with pre-loaded sounds
   - Supports WAV, MP3, and AIFF decoding with comprehensive format detection
   - AIFF support includes 16/24/32-bit depths, mono/stereo, and magic byte detection
   - Volume control with pre-processing to prevent crackling

4. **Configuration** (`internal/config/`)
   - XDG Base Directory compliant
   - Config file at `/etc/xdg/claudio/config.json`
   - Environment variable overrides: `CLAUDIO_VOLUME`, `CLAUDIO_SOUNDPACK`, etc.

5. **File Logging System** (`internal/config/`)
   - Simple idiomatic Go logging using `io.MultiWriter` + `lumberjack.v2`
   - Dual output: stderr + rotated log files
   - XDG-compliant log location: `~/.cache/claudio/logs/claudio.log`
   - Default enabled for hook-based usage (CLI flags hard to pass)
   - Graceful degradation: continues with stderr-only if file logging fails

### Key Design Decisions

1. **TDD Approach**: All components have comprehensive tests written first
2. **slog Logging**: Extensive structured logging throughout for debugging
3. **Simple v1 CLI**: Focused on core functionality, saving rich features for v2
4. **Memory-based Audio**: Pre-loads entire sound files to avoid streaming complexity

## Configuration

Default config location: `/etc/xdg/claudio/config.json`
```json
{
  "volume": 0.5,
  "default_soundpack": "default",
  "soundpack_paths": ["/usr/local/share/claudio/default"],
  "enabled": true,
  "log_level": "warn",
  "file_logging": {
    "enabled": true,
    "filename": "",
    "max_size_mb": 10,
    "max_backups": 5,
    "max_age_days": 30,
    "compress": true
  }
}
```

### File Logging Configuration

The file logging system supports the following configuration options:

- **`enabled`**: Boolean, whether file logging is active (default: `true`)
- **`filename`**: String, custom log file path (empty = XDG cache path)
- **`max_size_mb`**: Integer, max file size in MB before rotation (default: `10`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ctoth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
