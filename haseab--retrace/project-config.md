---
trigger: always_on
description: > **Standard**: This file follows the [AGENTS.md](https://agents.md) specification - a vendor-agnostic standard for AI agent guidance. For human-readable project information, see [README.md](README.md).
---

# Retrace - Agent Guide

> **Standard**: This file follows the [AGENTS.md](https://agents.md) specification - a vendor-agnostic standard for AI agent guidance. For human-readable project information, see [README.md](README.md).

Retrace is a local-first screen recording and search application for macOS, inspired by Rewind AI. It captures screens, extracts text via OCR, and makes everything searchable—all locally on-device.

**Status**: Core screen capture (CGWindowListCapture), OCR (Vision), full-text search (FTS5), HEVC encoding, and Rewind import are working. Audio transcription and vector search are planned for future releases.

---

## Quick Reference

- **Module-Specific Instructions**: Each module has its own `AGENTS.md` file in its directory
- **Human Documentation**: [README.md](README.md), [CONTRIBUTING.md](CONTRIBUTING.md), and [AI_ISSUE_TEMPLATE.md](AI_ISSUE_TEMPLATE.md)
- **Issue Reporting**: Use `AI_ISSUE_TEMPLATE.md` and `gh issue create --body-file ...` for AI-authored GitHub issues
- **Bug Fixes by Non-Owners**: If the user is fixing a bug/crash and does not appear to be the repo owner, encourage them to create or link a GitHub issue before making code changes
- **Technical Audit Docs**: `local/docs/` (includes deep-dive implementation and performance audit notes)

---

## Project Commands

### Build & Test

```bash
# Build all targets
swift build

# Run all tests
swift test

# Run specific module tests
swift test --filter DatabaseTests

# Run specific test
swift test --filter testSpecificMethod

# Clean build artifacts
rm -rf .build/
```

---

## Project Structure

```
retrace/
├── AGENTS.md                    # This file - main agent coordination
├── .env.example                 # Template for local release credentials (copy to .env)
├── .github/                     # GitHub configuration
│   ├── CODEOWNERS
│   ├── FUNDING.yml
│   └── ISSUE_TEMPLATE/
│       └── bug_report.yml       # GitHub bug report form aligned with AI issue template
├── AI_ISSUE_TEMPLATE.md         # Canonical markdown template for AI-authored bug reports
├── README.md                    # Human-readable project overview
├── CONTRIBUTING.md              # Contribution guidelines
├── Package.swift                # Swift Package Manager configuration
├── scripts/                     # Build/release/validation scripts
│   ├── release.sh               # End-to-end release automation
│   ├── create-release.sh        # Release build + packaging helper
│   ├── check_no_nanoseconds_sleep.sh # Guardrail for Task.sleep(nanoseconds:)
│   ├── validate_sleep_wake_stability.sh # Sleep/wake soak validation workflow
│   └── validate_darkwake_watchdog.sh # Automated darkwake watchdog regression validation
│
├── Shared/                      # CRITICAL: Shared types and protocols
│   ├── Logging.swift            # Central log utility (Log.debug/info/warning/error)
│   ├── AppPaths.swift           # Application path configuration
│   ├── BGRAImageUtilities.swift # Shared BGRA conversion + patch extraction helpers
│   ├── MasterKeyManager.swift   # Keychain-backed master key creation + recovery phrase export
│   ├── ReversibleOCRScrambler.swift # Deterministic reversible OCR patch scrambling + text protection
│   ├── Models/                  # Data types used across modules
│   │   ├── Frame.swift          # FrameID, CapturedFrame, VideoSegment
│   │   ├── Text.swift           # ExtractedText, OCRTextRegion
│   │   ├── TextRegion.swift     # OCR text region types
│   │   ├── Search.swift         # SearchQuery, SearchResult
│   │   ├── Segment.swift        # Segment data model
│   │   ├── Config.swift         # Configuration types
│   │   ├── Errors.swift         # Error types
│   │   ├── Audio.swift          # Audio model types (Release 2)
│   │   ├── FilterCriteria.swift # Timeline/search filter criteria
│   │   ├── Source.swift         # Data source enum (native, rewind, etc.)
│   │   ├── Tag.swift            # Tag model types
│   │   └── Comment.swift        # Segment comment and attachment models
│   └── Protocols/               # Module interfaces
│       ├── DatabaseProtocol.swift
│       ├── StorageProtocol.swift
│       ├── CaptureProtocol.swift
│       ├── ProcessingProtocol.swift
│       ├── SearchProtocol.swift
│       └── MigrationProtocol.swift
│
├── Database/                    # SQLite + FTS5 storage
│   ├── AGENTS.md                # Module-specific agent instructions
│   ├── DatabaseManager.swift    # Main database coordinator
│   ├── DatabaseConnection.swift # SQLite connection management
│   ├── DatabaseConfig.swift     # Database configuration
│   ├── FTSManager.swift         # Full-text search management
│   ├── IDMappingService.swift   # ID mapping between sources
│   ├── Schema.swift             # Current schema definition
│   ├── Migrations/              # Schema migration scripts
│   ├── Queries/                 # Query implementations
│   └── Tests/
│
├── Storage/                     # File I/O, HEVC encoding
│   ├── AGENTS.md
│   ├── StorageManager.swift
│   ├── ImageExtractor.swift     # Extract frames from video files
│   ├── IncrementalSegmentWriter.swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haseab/retrace](https://github.com/haseab/retrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
