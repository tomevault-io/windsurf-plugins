---
trigger: always_on
description: > **Note:** This file consolidates project context, workflows, and architectural details. It serves as the primary source of truth for AI agents and developers working on this repository.
---

# Ora - AI Agent & Developer Context

> **Note:** This file consolidates project context, workflows, and architectural details. It serves as the primary source of truth for AI agents and developers working on this repository.
>
> **Minimum macOS:** 26 (Tahoe) | **Chip:** Apple Silicon (M1+) | **RAM:** 16GB recommended, 32GB best

---

## Repository Guidelines

### Project Summary

**Ora** is a privacy-first macOS voice assistant that runs fully on-device:

```
Voice → (FluidAudio Parakeet) → (MLX + Qwen 2.5) → (Kokoro TTS) → Voice/UI
```

**Primary differentiator:** Fast, reliable, auditable actions (Calendar/Reminders/Contacts first), minimal cloud, and a UI that makes the assistant feel predictable and safe.

### Project Structure & Modules
- `Ora/Ora`: Main macOS app source (Audio, ASR, LLM, Tools, TTS, UI). Keep changes small and reuse existing helpers.
- `Ora/OraTests`: XCTest coverage for audio, transcription, LLM, tools; mirror new logic with focused tests.
- `scripts/`: Build/sign helpers (`build.sh`, signing scripts).
- `docs/`: Documentation (architecture, stories, design).
- `Vendor/`: External inference engines (MLX, Parakeet, Kokoro TTS).
- `project.yml`: XcodeGen configuration (generates `.xcodeproj`).

### Key Resources
| Resource | Location | Purpose |
|:---------|:---------|:--------|
| **System Prompt** | `Ora/Resources/system-prompt.txt` | LLM system prompt template with `{{variable}}` placeholders. Edit this file to change assistant behavior without modifying code. |

### Documentation Organization (MANDATORY)

**Always maintain a clean project structure:**

| Document Type | Location | Examples |
|:--------------|:---------|:---------|
| User stories | `docs/stories/` | Feature specs, PRD, implementation plans |
| Reports & investigations | `docs/` (in appropriate subfolders) | Performance reports, bug investigations, audits |
| Architecture docs | `docs/stories/` | System design, component diagrams |
| Setup & guides | `docs/` | Environment setup, testing guides |

- **All user stories MUST be collected in `docs/stories/`**
- **All reports, investigations, or other documents MUST be collected in appropriate folders under `docs/`**
- Create subfolders as needed (e.g., `docs/reports/`, `docs/investigations/`)
- Keep documentation up-to-date when implementation changes

### Build, Test, Run

**Build script commands:**
| Command | When to Use |
|:--|:--|
| `./build.sh` | Build only, don't launch |
| `./build.sh run` | Build and launch (kills old instance first) |
| `./build.sh clean` | Fresh start (removes DerivedData) |
| `./build.sh reset-perms` | Reset TCC permissions (Accessibility, Calendar, Reminders, Contacts) |
| `./build.sh test` | Run tests with token-optimized output |
| `./build.sh test-perms` | Run tests with permission prompts enabled |
| `./build.sh test-tsan` | Run tests with Thread Sanitizer enabled |
| `./build.sh logs` | Stream unified logs (includes debug level) |
| `./build.sh logs --category <name>` | Stream logs for specific category (e.g., `chunker`, `tts`, `llm`) |
| `./build.sh open-results` | Open `.xcresult` bundle in Xcode |
| `./build.sh sign` | Build, sign with Developer ID, notarize, and optionally create DMG (for distribution) |

### Viewing Logs (macOS Unified Logging)

**Important:** macOS filters out debug and info level logs by default. Use these commands to see all log levels:

**Stream logs in real-time:**
```bash
# All Ora logs (recommended)
./build.sh logs

# Specific category
./build.sh logs --category chunker
./build.sh logs --category tts
./build.sh logs --category llm
```

**View recent logs (last N minutes):**
```bash
# All levels including debug/info (REQUIRED for most debugging)
log show --debug --info --predicate 'subsystem == "com.ora.app"' --last 5m

# Specific category
log show --debug --info --predicate 'subsystem == "com.ora.app" AND category == "chunker"' --last 5m

# Search for specific text
log show --debug --info --predicate 'subsystem == "com.ora.app"' --last 5m | grep -i "search term"
```

**Log categories:** `audio`, `asr`, `llm`, `tools`, `tts`, `chunker`, `ui`, `orchestration`, `permissions`, `models`, `persistence`

**Log levels (in order of severity):**
- `.debug` - Detailed debugging (filtered by default, use `--debug` flag)
- `.info` - General flow info (filtered by default, use `--info` flag)
- `.notice` - Important events (always visible)
- `.error` - Errors that don't crash
- `.fault` - Critical errors with stack traces

**Common workflows:**

```bash
# Normal development
./build.sh run

# Permissions stopped working (prompt keeps appearing)
./build.sh reset-perms
./build.sh run
# Grant permissions when prompted

# Fresh start (new clone, weird build issues)
./build.sh clean
./build.sh reset-perms
./build.sh run
```

**Why `reset-perms`?** macOS TCC tracks permissions by bundle ID + CDHash. Every rebuild changes the CDHash, so old permission grants don't apply. `reset-perms` clears stale entries so the next grant applies to the current build.

**Skip setup wizard (for testing):**
```bash
# Mark setup as complete to bypass the setup wizard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benedict2310/ora](https://github.com/benedict2310/ora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
